---
title: Details
parent: Framework mods
grand_parent: Framework basics
nav_order: 9
---

# Details

The game version, threading rule and failure behaviour behind framework mods. Read it when a framework mod does not load or behaves unexpectedly.

Target: Obenseuer 0.5.08 (buildid 25969064), Unity 6000.0.77f1 Mono, [BepInEx](../../glossary.md#bepinex) (the game's mod loader) 6 Unity.Mono ([be.788](https://builds.bepinex.dev/projects/bepinex_be/788/BepInEx-Unity.Mono-win-x64-6.0.0-be.788%2B5b766a3.zip), the tested build) or BepInEx 5 (5.4.23 or later).
Namespace: `ObenseuerCore`.

Everything not listed in this reference is internal and may change without notice. Items marked experimental may change too.

## When you can call it

- Call everything from your mod's normal code (`OnLoad`, event handlers, `Update`), not from a background thread you started. The game only runs safely on the main thread (see the [glossary](../../glossary.md#main-thread)).

## If something fails to set up

This part of the framework does not change any game method, so there is no [feature group](../../glossary.md#feature-group) to turn off. A mod that fails to load is skipped with a warning in `BepInEx\logs\ObenseuerCore.log`. The warning is on the line that starts with "framework mods:" and in the lines after it. The rest of the mods keep running.

**Common problems**

- Your mod does not load: find the "framework mods:" line in `BepInEx\logs\ObenseuerCore.log` and read the warning there. [My mod does not load](../../faq.md#my-mod-does-not-load) in the FAQ and the Common problems part of [Your first mod](../../guides/getting-started/first-mod.md) list the checks.
