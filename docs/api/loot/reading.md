---
title: Reading
parent: Loot pools
grand_parent: Items and crafting
nav_order: 3
---

# Reading

What the reading calls, `Categories` and `PoolFor`, give you, and which calls need `IsAvailable`. The full member list is on [How picking works](how-picking-works.md).

## Things to know

- `IsAvailable` tells you whether adding and the event work. `Add`, `Remove` and `ContainerFilled` need it. `Categories` and `PoolFor` do not. <span class="tech">(It is true when the loot patches are installed and their feature group is not turned off.)</span>
- `Categories()` lists every category the game has. It is empty when the game's loot database is not ready yet, for example on the main menu.
- `PoolFor(category)` lists the items of one category. The common items come first, then the rare ones. Each item is listed once, in the order it first appears, and `Weight` counts its copies. An unknown category gives an empty list.
