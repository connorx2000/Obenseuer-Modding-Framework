---
title: Rules for all three APIs
parent: Quests, dialogue and law
grand_parent: Story
nav_order: 2
---

# Rules for all three APIs

These rules are the same for [`Quests`](quests-experimental.md), [`Dialogue`](dialogue.md) and [`Law`](law-experimental-for-arrest.md). They say what a call gives back, when it throws, and when your event handlers run. Read them once before you use any of the three.

## Things to know

- `IsAvailable` tells you whether the API works. It is `false` when the changes it needs to the game were not made. It is also `false` when the game build is not one the framework supports and the player turned on `[Safety] RefuseUnknownBuild`. The third case is that the API's [feature group](../glossary.md#feature-group) was turned off after a critical change failed. <span class="tech">(The second case is `Core.PatchingAllowed` being false. The third is a failed critical change in the API's feature group.)</span>
- When `IsAvailable` is `false`, reads give you a plain default: `""`, `0`, `false`, `null` or an empty array. Anything that changes the game returns `false`. See [If something fails to set up](details.md#if-something-fails-to-set-up).
- A wrong [mod id](../glossary.md#mod-id) or a missing callback throws an error. Arguments that register your code throw `ArgumentException` or `ArgumentNullException`. They throw before the framework checks whether the API is available. The bad arguments are a null or empty `modId`, and a null handler or method. A name that is not valid for [Lua](../glossary.md#lua) (the dialogue scripting language) throws too. So does a `target` that is not an object of the method's class.
- A wrong game value does not throw. Arguments that name something in the game (ids, names, values, amounts) never throw. If one is wrong, the call returns `false`, `null` or the plain default. One line is logged for each API, member and reason.
- If the game fails inside a call, you get the plain default. The first failure of each member is logged. Later failures of that member are not logged again.
- Events arrive a moment after the change, outside the game's own call. The framework notes the change while the game works, and tells your handlers a little later, from the next frame. So your handlers may call anything that changes the game. <span class="tech">(Events are queued during the game's call and raised from `LateUpdate` of a host object the framework creates, marked `DontDestroyOnLoad`.)</span>
- Events are delivered in the order they happened. An event that happens while others are being delivered waits for the next frame. A handler that throws is logged, and the other handlers still run.
- An `Intercept` handler runs inside the game's call, before it happens, and may change or cancel it. "Intercept" means your code gets a first look at something the game is about to do (see [interceptor](../glossary.md#interceptor)). Handlers run in the order they were added. If your handler calls the same API from inside, the framework skips the intercept handlers for that inner call, so it goes through unchanged. A handler that throws is logged and skipped.
- After a load, set your Lua variables again. Loading a save replaces the whole table of conversation variables. If your mod needs a variable, set it again on [`Events.Loaded`](../api-core/events.md), which fires when a save finishes loading. Lua functions you add with `RegisterFunction` live only in memory, so register them again after each start.
- `RemoveAll(modId)` takes away everything your mod added to that API. That is its intercepts and, for `Dialogue`, the responses added with `AddResponse` and the Lua functions from `RegisterFunction`.

## Limits

- Changes are refused while the game is loading or saving. Writes in the money, inventory, container, quest, dialogue, law and world-object APIs return `false` during a load or save.
- A story event noted before a load starts is dropped in three cases: the load is for a different character, the main menu opens, or a new game starts. A level change for the same character keeps it, because the game saves and reloads the same character's save. Events noted while a save or a load runs are delivered once it ends.
