---
title: Common tasks
parent: Recipes and crafting
grand_parent: Items and crafting
nav_order: 1
---

# Common tasks

Short snippets for the usual jobs with [`Recipes`](recipes.md), [`Crafting`](crafting-stations.md) and [`Plants`](growing-plants.md): find and unlock recipes, and list the crafting stations and plants.

Find every recipe that makes an item, and show whether the player can make each one now. The number `90110` is Turnip Mash:

```csharp
foreach (RecipeInfo recipe in Recipes.Find(90110))
    Log.Info(recipe.Name + ": can craft now = " + Recipes.CanCraft(recipe.Id) + ", skill ok = " + Recipes.MeetsSkill(recipe.Id));
```

Give the player a recipe they do not have yet, with the "New recipe" notice. `Unlock` returns `false` when they already know it. The number `10147` is Cabbage Soup:

```csharp
if (!Recipes.Unlock(10147)) Log.Info("The player already knows that recipe, or no save is loaded.");
```

List the crafting stations in the [scene](../../glossary.md#scene) and whether each one is busy:

```csharp
foreach (CraftingStationInfo station in Crafting.All())
    Log.Info(station.Kind + " at " + station.Position + ", busy: " + station.Busy);
```

List the plants in the scene and how many fruits are ready to pick:

```csharp
foreach (PlantInfo plant in Plants.All())
    Log.Info("Plant " + plant.Id + " is " + (plant.FullyGrown ? "fully grown" : "still growing") + ", ready fruit: " + plant.ReadyFruit);
```
