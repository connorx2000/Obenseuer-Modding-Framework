---
title: Common tasks
parent: Loot pools
grand_parent: Items and crafting
nav_order: 1
---

# Common tasks

Short snippets for the usual jobs with [`LootPools`](how-picking-works.md): list the categories, look inside one, take your item out again, and add to a rare list. `Mod.Id` is your [mod id](../../glossary.md#mod-id), such as `example.hello`.

List the categories and how long their lists are.

```csharp
foreach (LootCategoryInfo category in LootPools.Categories())
    Log.Info(category.Name + ": " + category.CommonWeight + " common, " + category.RareWeight + " rare");
```

See what one category contains.

```csharp
foreach (LootEntryInfo entry in LootPools.PoolFor("Cupboard"))
    Log.Info(entry.Tier + " item " + entry.ItemId + " x" + entry.Weight);
```

Take your item out of a category again. `Remove` returns `true` when your mod had an entry for that item.

```csharp
bool removed = LootPools.Remove(Mod.Id, "Cupboard", 14780);
```

Put an item in the rare list. Do this only for a category that already has rare items. The reason is in [How picking works](how-picking-works.md#things-to-know).

```csharp
LootPools.Add(Mod.Id, "Toolbox", 14780, 1, LootTier.Rare);
```
