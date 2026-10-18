---
title: Home
nav_order: 1
permalink: /
---

# Obenseuer Modding Framework

The Obenseuer Modding Framework is a free add-on that makes it easier to mod the game Obenseuer. It is a plugin for [BepInEx](glossary.md#bepinex) (the tool that loads mods) and works on both BepInEx 5 and BepInEx 6.

It gives every mod the same building blocks, so each mod author does not have to build them again:

- a shared Mods menu with settings
- data kept in each save
- game events, such as a day passing or the player's money changing
- helpers for the game's items, NPCs, shops, doors and more
- safe changes to the game's code

[Install](install.md){: .btn .btn-primary }
[Make your first mod](guides/getting-started/first-mod.md){: .btn }
[Everything the framework covers](features.md){: .btn }

## Who it is for

- **Players** who want to run mods that need the framework. Follow [Install](install.md). It takes a few minutes and needs no coding.
- **People who want to make a mod**, even if you have never made one. Follow Install, then [Your first mod](guides/getting-started/first-mod.md). It builds a small mod that writes one line to the [log](guides/getting-started/logging.md).
- **Experienced modders** who want to look something up. The [API reference](API.md) lists every page with one line on what it does. [Events](events.md) lists everything a mod can react to. The [glossary](glossary.md) explains the terms.

## Where to start

1. [Install](install.md) the framework. Start the game once, then check that `BepInEx\logs\ObenseuerCore.log` exists.
2. Make [your first mod](guides/getting-started/first-mod.md).
3. Browse the [guides](guides/index.md) and the [cookbook](guides/cookbook/index.md) for step-by-step tasks, such as giving an item or showing an HTML panel.
4. If you are stuck, read the [FAQ](faq.md).

After the first two steps you have a working mod. The guides show what to add next.

{: .warning }
This is a beta and has not been tested extensively. The framework is built for Obenseuer 0.5.08. [Status](status.md) lists what is tested.

## What is new in 0.11.1-beta

Version 0.11.1-beta supports game build 0.5.08 and fixes bugs. Version 0.11.0-beta added `FrameworkMod`, the base class for [framework mods](api/framework-mods.md). With it, one DLL (the mod file) runs on both BepInEx 5 and BepInEx 6, and its settings show in the Mods menu. The same version added dialogue branches, HUD bars, screen vignette and blur, NPC control and animation, and more furniture options. The [changelog](https://github.com/NomadsReach/Obenseuer-Modding-Framework/blob/main/CHANGELOG.md) lists every change. The earlier 0.10.0-beta additions, such as [player shops](glossary.md#player-shop), [mod NPC](glossary.md#mod-npc) routines and the BepInEx 5 build, are on the [status page](status.md#what-is-new-in-0100-beta).
