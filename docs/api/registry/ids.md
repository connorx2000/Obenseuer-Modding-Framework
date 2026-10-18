---
title: IDs
parent: Registry
grand_parent: Items and crafting
nav_order: 2
---

# IDs

This page explains the number, or id, that the framework gives each item and recipe your mod adds. Read it before you choose the keys for your items.

The reference pages list what the API gives you, in the order a mod meets it: ids, items, icons, models, recipes, then what happens to saves. Text in brackets at the end of a note is extra detail you can skip on a first read.

You never choose the number. The framework works it out from your [mod id](../../glossary.md#mod-id) and the key you give the item. The same pair always gives the same number.

- `IdFor(modGuid, key)` gives the number for a mod id and a key. It is the same in every session and in any load order. Items and recipes have separate lists of numbers. Using the same key for an item and a recipe gives the same number in each list, which does no harm. <span class="tech">(The number is `1000000 + (FNV-1a 32-bit of the UTF-8 bytes of modGuid + "/" + key) % 1999000000`.)</span>
- Capital letters matter in ids. `Define("MyMod", key)` and `Define("mymod", key)` give two different numbers. Use one spelling for your mod id everywhere.
- Saves and log files do not care about capitals. So the same mod written two ways shares its [save data](../../guides/getting-started/save-data.md) and its [log](../../guides/getting-started/logging.md), but not its item numbers. <span class="tech">(`IdFor` hashes the exact text. `SaveData` and `CoreLog` compare names without regard to case. The hash is not changed, because that would break saved ids.)</span>
- `modGuid` and `key` are 1 to 64 characters from `a-z A-Z 0-9 . _ -`. Never rename a key after you have released the mod: a new key is a new item.
