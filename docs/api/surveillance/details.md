---
title: Details
parent: Surveillance
grand_parent: Law and crime
nav_order: 5
---

# Details

The build this API targets and where its log goes. Also when it is safe to call it, and what happens when the framework cannot change a game method.

Target: Obenseuer 0.5.08 (buildid 25969064), Unity 6000.0.77f1 Mono, [BepInEx](../../glossary.md#bepinex) 6 Unity.Mono ([be.788](https://builds.bepinex.dev/projects/bepinex_be/788/BepInEx-Unity.Mono-win-x64-6.0.0-be.788%2B5b766a3.zip), the tested build) or BepInEx 5 (5.4.23 or later).
Namespace: `ObenseuerCore`. Logs go to `BepInEx\logs\obenseuer.moddingframework.surveillance.log`.

Everything not listed in this reference is internal and may change without notice. Items marked experimental may change too.

## When you can call it

- Call everything from your mod's normal code, not from a background thread you started. The game only runs safely on the main thread (see the [glossary](../../glossary.md#main-thread)). This goes for the code that handles the [events](events.md) too.
- It is safe to call any of it at any time, even on the main menu, while loading, or with no save loaded. Reads give a plain default: `false` or an empty list. [`SetCameraActive`](cameras-and-the-checks.md) gives `false` when no camera has the id, and while a save is loading or saving.
- Your event handlers run inside the game's own call. If one throws an error, the framework logs it and still runs the other handlers.

## If something fails to set up

The framework changes some of the game's methods when it starts, so it can tell your mod about sightings and trespassing. If a game update breaks one of them, only that part turns off. The framework writes error lines and a warning to the [log](../../guides/getting-started/logging.md). The rest of the framework keeps working. This page has two [feature groups](../../glossary.md#feature-group) (named sets of framework changes).

- `PlayerSpotted` uses the feature group `obenseuer.moddingframework.surveillance.witness`. `TrespassEntered` and `TrespassLeft` use `obenseuer.moddingframework.surveillance.trespass`.
- If a change is missing, only that group is turned off and only its events stop. The other group and all the reads keep working. `IsAvailable` is `false` while any group is missing. <span class="tech">(The changed methods are `Witness.Witnessed` (witness group) and `TrespassingArea.OnTriggerStay` and `OnTriggerExit` (trespass group).)</span>
