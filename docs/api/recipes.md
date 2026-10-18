---
title: Recipes and crafting
parent: Items and crafting
grand_parent: API reference
has_children: true
nav_order: 10
---

# Recipes and crafting

This API reads the game's recipes, crafting stations and growing plants. You need it when your mod checks or gives recipes, or reacts to crafts and harvests.

## What it is

This page covers three connected parts of the game:

- **Recipes** say what the player can make: which items go in, which come out, how long it takes and what skill it needs. See [`Recipes`](recipes/recipes.md).
- **Crafting stations** are the places where a recipe is carried out, such as a stove or an oven. See [`Crafting`](recipes/crafting-stations.md).
- **Growing plants** are the crops that grow in the world and in growing stations. See [`Plants`](recipes/growing-plants.md).

With this API your mod can:

- look recipes up and check whether the player can make one
- give the player a recipe
- list the crafting stations and plants
- be told when something is crafted, a plant is harvested or a recipe is unlocked.

To add a brand new recipe to the game, see the [Registry API](registry.md). There, a new recipe starts as a copy of an existing recipe, its [template](../glossary.md#template).

## When to use it

- Look up the ingredients and results of a recipe, or find every recipe that makes a given item.
- Check whether the player has the ingredients, the skill and the right station for a recipe.
- Give the player a recipe, with or without the "New recipe" notice.
- React when the player starts a craft, finishes one, harvests a plant or learns a recipe.
- List the crafting stations and the plants in the [scene](../glossary.md#scene) you are in.

## Quick example

This mod writes a [log](../guides/getting-started/logging.md) line each time a recipe is unlocked, a craft finishes or a plant is harvested. Once a save is loaded, it looks up the recipe for turnip mash and logs what goes in.

```csharp
using ObenseuerCore;

[ModInfo("example.craftwatch", "Craft Watch", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class CraftWatch : FrameworkMod
{
    protected override void OnLoad()
    {
        Recipes.Unlocked += args => Log.Info("Recipe " + args.RecipeId + " unlocked, learned by crafting: " + args.Learned);
        Crafting.Crafted += args => Log.Info("Recipe " + args.RecipeId + " finished, success: " + args.Success);
        Plants.Harvested += args => Log.Info("Harvested " + args.Items.Count + " kinds of item from plant " + args.PlantId);
        Events.WhenReady(() =>
        {
            RecipeInfo recipe = Recipes.Get(0);
            if (recipe != null) Log.Info(recipe.Name + " needs " + recipe.Ingredients.Count + " ingredients.");
        });
    }
}
```

`Unlocked`, `Crafted` and `Harvested` are events: the framework calls your code each time they happen. [`Events.WhenReady`](core-helpers/events-whenready.md) runs your code once a save is loaded. See [Using events](../guides/getting-started/using-events.md) for more on events and `WhenReady`.
