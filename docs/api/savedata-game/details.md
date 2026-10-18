---
title: Details
parent: Game save data
grand_parent: Framework basics
nav_order: 6
---

# Details

This page gives the game build that game save data targets, its log file, and when you can call it.

Target: Obenseuer 0.5.08 (buildid 25969064), Unity 6000.0.77f1 Mono, BepInEx 6 Unity.Mono ([be.788](https://builds.bepinex.dev/projects/bepinex_be/788/BepInEx-Unity.Mono-win-x64-6.0.0-be.788%2B5b766a3.zip), the tested build) or BepInEx 5 (5.4.23 or later).
Namespace: `ObenseuerCore`. Logs go to `BepInEx\logs\ObenseuerCore.log`.

Everything not listed in this reference is internal and may change without notice. Items marked experimental may change too.

## When you can call it

- Call everything from your mod's normal code, not from a background thread you started. The game only runs safely on the [main thread](../../glossary.md#main-thread).
- `Capture` and `Apply` do nothing while a save is loading or saving. See [Copying one saved object](copying-one-saved-object.md).

## If something fails to set up

- This page does not change any game method when the framework starts. So it has no [feature group](../../glossary.md#feature-group) (a named set of framework changes) that can turn off.
- `Capture` and `Apply` use the game's own save and load methods when you call them.
