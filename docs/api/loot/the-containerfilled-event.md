---
title: The ContainerFilled event
parent: Loot pools
grand_parent: Items and crafting
nav_order: 5
---

# The ContainerFilled event

When [`ContainerFilled`](how-picking-works.md) fires, and which ways of filling a container it does not report.

## Things to know

- `ContainerFilled` fires after a container has been filled with at least one stack. It covers the first fill, the refills when a save loads and on a timer, and any other fill of a container. It fires whether the fill draws from a pool or not. <span class="tech">(It is raised after `Storage.SpawnItems`. With no subscriber the framework does almost nothing extra: one check for `null`.)</span>
- Your handlers run one at a time. If one throws an error, the framework logs it and still runs the others.

## Limits

- It does not fire for some ways items appear. It misses the [janitors'](../janitors.md) fills and items the player moves between containers. It also misses the contents of storage packages, and items that collectible spawners put into the world. <span class="tech">(The janitor fills are `StorageJanitorAction`.)</span>
