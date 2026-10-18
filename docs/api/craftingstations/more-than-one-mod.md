---
title: More than one mod
parent: Crafting stations
grand_parent: Items and crafting
nav_order: 5
---

# More than one mod

Which mod's speed or quality factor applies when several mods set one. Read this when other mods may change crafting too.

## Things to know

- Each mod keeps one speed for each kind of station and one quality factor, and the newest call wins. If a second mod sets a value another mod already set, the second mod's value applies. One warning in the [log](../../guides/getting-started/logging.md) names both mods.
- A mod that calls again replaces its own value without a warning.
- [`Clear(modId)`](the-calls.md) removes all of that mod's changes. The earlier mod's value then applies again.
