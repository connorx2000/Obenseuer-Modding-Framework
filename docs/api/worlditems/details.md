---
title: Details
parent: World item events
grand_parent: Items and crafting
nav_order: 5
---

# Details

This page gives the target game version, the feature groups and the rules for calling `WorldItems`. Check it when an event does not fire or `IsAvailable` is `false`.

Target: Obenseuer 0.5.08 (buildid 25969064), Unity 6000.0.77f1 Mono, [BepInEx](../../glossary.md#bepinex) (the game's mod loader) 6 Unity.Mono ([be.788](https://builds.bepinex.dev/projects/bepinex_be/788/BepInEx-Unity.Mono-win-x64-6.0.0-be.788%2B5b766a3.zip), the tested build) or BepInEx 5 (5.4.23 or later).
Namespace: `ObenseuerCore`. Source: `src/Api/WorldItems.cs` and `src/Api/WorldItemsPatches.cs`.

Everything not listed in this reference is internal and may change without notice.

The framework changes some of the game's methods when it starts, so it can tell your mod about pickups, drops and uses. This page calls each group of these changes a [feature group](../../glossary.md#feature-group) (a named set of framework changes). The feature group is `obenseuer.moddingframework.worlditems`, with four groups under it: `.pickup`, `.drop`, `.used` and `.saved`. A separate one, `obenseuer.moddingframework.worlditems.expiry`, covers the expiry timer. These are the names you will see in the [log](../../guides/getting-started/logging.md).

## When you can call it

- Call everything from your mod's normal code (`OnLoad`, event handlers, `Update`), not from a background thread you started. The game only runs safely on the main thread (see the [glossary](../../glossary.md#main-thread)).
- With no listeners, the changes do almost nothing. They return before they allocate anything.
- The framework does not hold back these events while a save is loading. A handler that should ignore events raised then can check [`SaveData.Busy`](../../api-core/savedata.md). It is `true` while a save is loading or being written.

## What to check for and what comes back

- [`WorldItems.IsAvailable`](events.md) tells you whether all four core groups were set up. It is `false` when any of `.pickup`, `.drop`, `.used` or `.saved` could not be set up, or when the feature group was turned off.
- Each event needs only its own group: `PickedUp` needs `.pickup`, `Dropped` needs `.drop`, and `Used` needs `.used`. `IdOf` gives `null` for a restored dropped item only when `.saved` is missing. The other groups keep working. Other parts of the framework are not affected.

## If something fails to set up

The framework changes some of the game's methods when it starts. If a game update breaks one of them, the framework turns off the [feature group](../../glossary.md#feature-group) that depends on it. It writes error lines and a warning to the group's log file in `BepInEx\logs`. The rest of the framework keeps working.

- If a change fails, only its group is turned off, and the framework writes error lines and a warning to the log. `WorldItems.IsAvailable` then becomes `false`. <span class="tech">(The six changes are a postfix on `CollectibleItem.PerformPickup` (`PickedUp`); a prefix and finalizer on `ItemData.DropItem(int, bool)`, which marks a player drop in progress; a postfix on `ItemOperations.SpawnItemToPosition`, which raises `Dropped` only while a drop is in progress; a prefix and postfix on `Item.InvokeAction(ItemData, Item.ItemAction, bool)` and a prefix, finalizer and postfix on `Item.InvokeActions(ItemData, bool)` (`Used`); and a postfix on `DestructibleList.OnLoadingGameDestructibleList()`, which records the dropped items that the save restored so `IdOf` can choose the id form.)</span>
- The expiry timer has its own feature group. If its change (a prefix on `Collectible.StartExpirationTimer(int?)`) fails, the events are not affected and [`SetExpiryOverride`](expiry-default.md) returns `false`.

**Common problems**

- `WorldItems.IsAvailable` is `false`: one of `.pickup`, `.drop`, `.used` or `.saved` could not be set up, or the feature group was turned off. Look for the error lines and the warning in the group's log file in `BepInEx\logs`. The events whose groups were set up keep working.
