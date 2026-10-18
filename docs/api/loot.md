---
title: Loot pools
parent: Items and crafting
grand_parent: API reference
has_children: true
nav_order: 9
---

# Loot pools

[`LootPools`](loot/how-picking-works.md) lets your mod read the lists that game containers fill from, and add its own items to them. Use it to make a mod item turn up in cupboards, fridges and other containers around the world.

Status: experimental.

## What it is

[Containers](inventory/containers.md) in the game, such as cupboards, toolboxes, fridges and dumpsters, fill themselves with items. Each container draws its items from a loot pool: a list of items kept under a category name, such as `Cupboard`. Each category has two lists, a common list and a rare list.

This API gives your mod these jobs:

- Read those lists.
- Add your own items to a category.
- Take your items out again.
- Create a category of your own.
- Get told when a container has been filled.

All changes are in memory only. The framework never writes the game's files. Your entries are gone when the game closes, so your mod adds them again at every start.

## When to use it

- Make a new item turn up in containers around the world, for example your mod's food in kitchen cupboards.
- See what a category can give, and how likely each item is.
- React when a container has been filled, for example to log or count the items.
- Make a new category of your own for containers that your own code fills.

## Quick example

This mod adds table salt to the cupboard loot pool, three times as likely as an item with weight 1. It also writes a [log](../guides/getting-started/logging.md) line with what each container receives when it is filled.

```csharp
using ObenseuerCore;

[ModInfo("example.saltycupboards", "Salty Cupboards", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class SaltyCupboards : FrameworkMod
{
    protected override void OnLoad()
    {
        bool added = LootPools.Add(Mod.Id, "Cupboard", 14780, 3);
        Log.Info("Salt added to cupboards: " + added);
        LootPools.ContainerFilled += args => Log.Info(args.Container.Name + " was filled with " + args.ItemIds.Count + " stacks.");
    }
}
```

`14780` is the item number of table salt in the game's item file, and `3` is the weight. [How picking works](loot/how-picking-works.md) explains weights. `Mod.Id` is your mod's id (see the [glossary](../glossary.md#mod-id)). `ContainerFilled` is an event: the framework calls your code each time it happens. See [Using events](../guides/getting-started/using-events.md).
