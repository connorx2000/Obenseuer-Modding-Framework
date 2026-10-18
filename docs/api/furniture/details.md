---
title: Details
parent: Furniture
grand_parent: Items and crafting
nav_order: 6
---

# Details

This page lists the game build behind the furniture API, when it is safe to call, and what comes back when something fails. You need it when you call the API around a save load, or when you check `IsAvailable`.

Target: Obenseuer 0.5.08 (buildid 25969064), Unity 6000.0.77f1 Mono, [BepInEx](../../glossary.md#bepinex) (the game's mod loader) 6 Unity.Mono ([be.788](https://builds.bepinex.dev/projects/bepinex_be/788/BepInEx-Unity.Mono-win-x64-6.0.0-be.788%2B5b766a3.zip), the tested build) or BepInEx 5 (5.4.23 or later).
Namespace: `ObenseuerCore`. Logs go to `BepInEx\logs\obenseuer.moddingframework.furniture.log`.

Everything not listed in this reference is internal and may change without notice. Items marked experimental may change too.

The framework changes some of the game's methods when it starts, so it can add furniture, blueprints and flatpacks. This page calls that group of changes a [feature group](../../glossary.md#feature-group) (a named set of the framework's changes for one feature). Its name is `obenseuer.moddingframework.furniture`, the name you will see in the [log](../../guides/getting-started/logging.md).

## When you can call it

- Call everything from your mod's normal code (`OnLoad`, event handlers, `Update`), not from a background thread you started. The game and Unity only run safely on the main thread (see the [glossary](../../glossary.md#main-thread)).
- Calls that change the game wait while a save is loading or being written. `Give` and the shop stock do not work then. Shop stock queued during a load is applied once the furniture state is restored, or at the next in-game hour.

## What to check for and what comes back

- [`FurnitureRegistry.IsAvailable`](defining-furniture.md) tells you whether furniture can be added. It is `false` when the framework could not set up its furniture features, or when the feature group was turned off. `Define`, `DefineBlueprint`, `GiveBlueprint`, `MailBlueprint` and `AddFlatpackLoot` need it to be `true`.
- A bad value throws an `ArgumentException`, and a `null` definition throws an `ArgumentNullException`. A refusal returns `null` or `false`. The rules for each call are on the other pages of this reference.

## If something fails to set up

The framework changes some of the game's methods when it starts. If a game update breaks one of them, the framework turns off the [feature group](../../glossary.md#feature-group) that depends on it. It writes error lines to the group's log file in `BepInEx\logs`. The rest of the framework keeps working.

- If the framework cannot change the game's furniture methods, the whole feature group turns off. `IsAvailable` becomes `false`, `Define` returns `null`, and mod furniture does not appear in the game.
- If the framework cannot change the game's build menu code, the pieces keep their template's group. One warning goes to the log at startup.
