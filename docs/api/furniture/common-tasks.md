---
title: Common tasks
parent: Furniture
grand_parent: Items and crafting
nav_order: 1
---

# Common tasks

Short examples for the usual jobs with [`FurnitureRegistry`](defining-furniture.md): wall placement, blueprints, flatpacks, mail and loot.

Make the furniture mount on the wall instead of standing on the floor.

```csharp
FurnitureDefinition shelf = new FurnitureDefinition();
shelf.TemplatePath = "Assets/Content/Prefabs/Building System/Furniture/Tables/Wooden Table.asset";
shelf.Title = "Wall Shelf";
shelf.Placement = FurniturePlacement.Wall;
string shelfId = FurnitureRegistry.Define(Mod.Id, "shelf", shelf);
```

Teach the player the [blueprint](../../glossary.md#blueprint) (a crafting plan for mod furniture) for your furniture. This blueprint needs 2 planks (item 35010) and 4 nails (item 35050). It must be defined while your mod loads.

```csharp
string furnitureId = FurnitureRegistry.IdFor(Mod.Id, "table");
BlueprintDefinition blueprint = new BlueprintDefinition();
blueprint.Inputs[35010] = 2;
blueprint.Inputs[35050] = 4;
FurnitureRegistry.DefineBlueprint(furnitureId, blueprint);
FurnitureRegistry.GiveBlueprint(furnitureId);
```

Give the player two [flatpacks](../../glossary.md#flatpack) (the packed form of mod furniture) of your furniture.

```csharp
string furnitureId = FurnitureRegistry.IdFor(Mod.Id, "table");
bool given = FurnitureRegistry.GiveFlatpack(furnitureId, 2);
Log.Info("Flatpacks given: " + given);
```

Send the player the blueprint as a letter by [mail](../economy/mail.md).

```csharp
string furnitureId = FurnitureRegistry.IdFor(Mod.Id, "table");
FurnitureRegistry.MailBlueprint(furnitureId, "Oak Table plans", MailKind.Letter);
```

Make flatpacks of your furniture turn up where the game rolls [loot](../loot/how-picking-works.md) from the "Toolbox" category. The rules for this call are on [World spawning](world-spawning.md).

```csharp
string furnitureId = FurnitureRegistry.IdFor(Mod.Id, "table");
FurnitureRegistry.AddFlatpackLoot(furnitureId, "Toolbox", 20);
```
