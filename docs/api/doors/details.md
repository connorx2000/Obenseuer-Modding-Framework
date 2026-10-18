---
title: Details
parent: Doors
grand_parent: Places and travel
nav_order: 6
---

# Details

The versions this API was tested against, what happens when the door events fail to set up, and the thread and save rules. Read this when the door events stay silent.

Target: Obenseuer 0.5.08 (buildid 25969064), Unity 6000.0.77f1 Mono, BepInEx 6 Unity.Mono ([be.788](https://builds.bepinex.dev/projects/bepinex_be/788/BepInEx-Unity.Mono-win-x64-6.0.0-be.788%2B5b766a3.zip), the tested build) or BepInEx 5 (5.4.23 or later).
Namespace: `ObenseuerCore`. Logs go to `BepInEx\logs\obenseuer.moddingframework.doors.log`.

Everything not listed in this reference is internal and may change without notice.

## If something fails to set up

The framework changes two of the game's methods when it starts, so it can tell your mod when a door has opened or closed. If a game update breaks either one, both events turn off. The framework writes error lines and a warning to the [log](../../guides/getting-started/logging.md). The rest of the API keeps working: the reads, `Open` and `Close`. The two changes belong to one [feature group](../../glossary.md#feature-group) (the framework's changes for one feature), `obenseuer.moddingframework.doors`. That is the name you will see in the log.

- [`Doors.IsAvailable`](finding-doors.md) tells you whether `Opened` and `Closed` work. It is `false` when the changes could not be made or the feature group was turned off. <span class="tech">(`CorePatcher.IsDisabled("obenseuer.moddingframework.doors")` is then true. The patched methods are `OnFullyOpen` and `OnFullyClosed` on `Door`, and the game has one version of each.)</span>

## When you can call it

- Call everything from your mod's normal code (OnLoad, event handlers, Update), not from a background thread you started. The game only runs safely on the main thread (see the [glossary](../../glossary.md#main-thread)).
- Any of it is safe to call at any time, even on the main menu. With no scene loaded, the reads give an empty list, `null` or `false`.
- Nothing throws an error. An id that is empty or not known gives `false` (or `null` for `Get` and `OpeningTimes`).
- Your event handlers run inside the game's own door update, one at a time. If a handler throws an error, the framework logs it and still runs the other handlers.

## Limits

- Each read searches every door in the scene. That is fine now and then, but do not call these on every frame. <span class="tech">(Each call runs `FindObjectsByType<Door>`.)</span>

## Save loads

The framework does not hold back the events while a save loads. A handler that should ignore events raised during a save load can check [`SaveData.Busy`](../../api-core/savedata.md). It is `true` while a save is loading or being written.
