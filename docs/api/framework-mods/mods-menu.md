---
title: Mods menu
parent: Framework mods
grand_parent: Framework basics
nav_order: 7
---

# Mods menu

These notes cover how your mod shows up in the game's Mods menu: the "MODS" button on the main menu. The settings you make with [`Config.Bind`](settings.md) appear there.

## Things to know

- Settings from `Config` show in the Mods menu under your mod's name. This is true for the classic menu and for the web menu, which uses [WebUi](../../glossary.md#webui) (the optional HTML page module).
- ConfigurationManager (opened with F1) lists BepInEx [plugins](../../glossary.md#plugin) only. It does not show framework mod settings.

## Limits

- An old framework does not load framework mods. Framework 0.10.0 and earlier skip them without a message. A player on an old framework does not see your mod in the game.

**Common problems**

- Your mod's settings are missing from ConfigurationManager (F1): it lists BepInEx plugins only. Open the Mods menu instead.
- A player does not see your mod at all: their framework is 0.10.0 or earlier. Framework mods need framework 0.11.0 or later; [Updating](../../install.md#updating) on the install page covers getting a newer one.
