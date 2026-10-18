---
title: Getting started
has_children: true
nav_order: 3
---

# Getting started

This section takes you from nothing to a mod you can share. Each page covers one job. You do not need to have made a mod before.

## Before you start

Install [BepInEx](../glossary.md#bepinex) (the game's mod loader) and the [framework](../glossary.md#framework) first, using the [install page](../install.md). Then start with [Your first mod](getting-started/first-mod.md). It builds a small mod that shows a hint box and writes a log line, in about five minutes.

Every later page uses the same project, so keep it. To try a page's code, replace the code in `Plugin.cs` with the code on that page and build again. The build writes over the same DLL (the mod file), so you do not get duplicate mods.

## Reading order

Read the pages in this order the first time:

1. [Your first mod](getting-started/first-mod.md): make, build and run a mod.
2. [Logging](getting-started/logging.md): write notes to your mod's log file.
3. [Config](getting-started/config.md): add settings players can change.
4. [Keybinds](getting-started/keybinds.md): react to a key.
5. [Using events](getting-started/using-events.md): react to the game clock, saves and levels.
6. [Save data](getting-started/save-data.md): remember things per save game.
7. [Patching](getting-started/patching.md): change a game method, as a last resort.
8. [Packaging](getting-started/packaging.md): make a zip to share.

After these eight pages you can make, test and share a mod. The [Cookbook](cookbook/index.md) then has short recipes for common jobs, such as giving an item or paying the player. Before you share your mod, read [Using the framework alongside other mods](multiple-mods.md).
