---
title: "Worked example: Spicy Sewer Jerky"
parent: Registry
grand_parent: Items and crafting
nav_order: 9
---

# Worked example: Spicy Sewer Jerky

This is the [quick example](../registry.md#quick-example) again, with the numbers worked out: the template values, the ids, and what to check in the game.

The [mod id](../../glossary.md#mod-id) (the id for your config, log, save data and keybinds) is `example.spicy-jerky`. The item copies Salted Pork, its [template](../../glossary.md#template) (the existing item it copies): item 19990, categories `Food` and `Meat`. The recipe copies Fried rat: recipe 10064, station `Stove`, 60 seconds. `IdFor` gives item `569330863` and recipe `879933748`.

```csharp
string mod = "example.spicy-jerky";
int item = ItemRegistry.Define(mod, "spicy-sewer-jerky", new ItemDefinition
{
    TemplateId = 19990,
    Title = "Spicy Sewer Jerky",
    Description = "Salted pork with bell pepper and tablesalt.",
    BaseValue = 40,
    BaseStackable = 8
});
RecipeRegistry.Define(mod, "spicy-sewer-jerky-recipe", new RecipeDefinition
{
    TemplateRecipeId = 10064,
    Name = "Spicy Sewer Jerky",
    Station = "Stove",
    Inputs = new Dictionary<int, float> { { 26030, 1f }, { 12820, 1f }, { 14780, 1f } },
    Outputs = new Dictionary<int, float> { { item, 1f } },
    ProcessTime = 120,
    RelevantSkill = Skill.Cooking,
    FailChance = 0.2f
});
```

The inputs are Rat Meat (26030), Bell Pepper (12820) and Tablesalt (14780), one each.

1. Call this from the plugin's `Awake`, before the item and recipe lists load. A return value of -1 means the call was refused.
2. Find the item in the game. It has Salted Pork's icon and model.
3. Find the recipe at a Stove.
4. Craft it. It should use the three inputs and make one jerky.

The jerky now exists as an item and a recipe. To give it its own icon, see [Common tasks](common-tasks.md).
