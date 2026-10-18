---
title: Details
parent: Spawners
grand_parent: Objects and devices
nav_order: 6
---

# Details

The game version, log file and threading rules behind [`Spawners`](reading-spawners.md). It also covers what happens when the framework cannot set it up, and what it does not cover yet. Read it when something does not work as expected.

Target: Obenseuer 0.5.08 (buildid 25969064), Unity 6000.0.77f1 Mono, [BepInEx](../../glossary.md#bepinex) (the game's mod loader) 6 Unity.Mono ([be.788](https://builds.bepinex.dev/projects/bepinex_be/788/BepInEx-Unity.Mono-win-x64-6.0.0-be.788%2B5b766a3.zip), the tested build) or BepInEx 5 (5.4.23 or later).
Namespace: `ObenseuerCore`. Logs go to `BepInEx\logs\obenseuer.moddingframework.spawners.log`.

Everything not listed in this reference is internal and may change without notice.

- The framework changes one of the game's methods when it starts, so it can tell your mod when a spawner spawns.
- This page's [feature group](../../glossary.md#feature-group) (a named set of framework changes) is `obenseuer.moddingframework.spawners`, the name you will see in the [log](../../guides/getting-started/logging.md).

## When you can call it

- Call everything from your mod's normal code, not from a background thread you started. The game and Unity only run safely on the main thread (see the [glossary](../../glossary.md#main-thread)). This goes for every member on this page and for the code that handles `Spawned`.
- It is safe to call any of it at any time. With no save loaded, reads return an empty list or `null`, and the writing calls return `false`. Nothing throws, and a null or unknown id finds nothing.
- The writing calls return `false` while a save is loading or being written. <span class="tech">(the check is `SaveData.Busy`.)</span>

## If something fails to set up

The framework changes some of the game's methods when it starts. If a game update breaks one of them, the framework turns off the [feature group](../../glossary.md#feature-group) that depends on it. It writes error lines to the group's log file in `BepInEx\logs`. The rest of the framework keeps working.

- If the change behind `Spawned` cannot be made, `Spawners.IsAvailable` becomes `false` and `Spawned` never fires. The feature group is turned off. The reads, `SetRespawn`, `SetRespawnTiming` and `TriggerNow` keep working. <span class="tech">(the change is a postfix on `Spawner.TrySpawn(bool)`.)</span>

**Common problems**

- `Spawned` never fires and `IsAvailable` is `false`: the change behind `Spawned` could not be made, and the feature group `obenseuer.moddingframework.spawners` is turned off. Look for the error lines in `BepInEx\logs\obenseuer.moddingframework.spawners.log`. The reads, `SetRespawn`, `SetRespawnTiming` and `TriggerNow` still work.

## What you can't do (yet)

- You can't change the game's difficulty adjustment of the respawn wait. You can set the timing fields, but the countdown itself is only readable.
- You can't read or change what a spawner spawns. Which items, furniture or characters appear is not covered here. For item lists, see [Loot](../loot.md) and `Spawn` on the [world objects page](../worldobjects/spawn.md#spawn).
- You can't make a spawn that skips the chance roll. The game calls its plain spawn from its own code in many places, so there is no single place for the framework to attach to. <span class="tech">(`Spawner.Spawn()` with no chance roll is not wrapped.)</span>

## While a save is loading

- The framework does not hold back `Spawned` while a save is loading.
- If your handler should ignore events raised during a load, check [`SaveData.Busy`](../../api-core/savedata.md#savedatabusy) first. It is `true` while a save is loading or being written.
