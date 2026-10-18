---
title: Common tasks
parent: Registry
grand_parent: Items and crafting
nav_order: 1
---

# Common tasks

Short snippets for the usual registry jobs: your own icon, a replaced icon, an item's number in advance, and the clean-up after a removed mod.

These snippets also go in your plugin's `Awake`, or in a method that `Awake` calls, unless the text says otherwise. `ModGuid` is the constant from the [quick example](../registry.md#quick-example).

**Give the item your own icon.** `IconPath` must be an absolute path to a square PNG: 128 or 256 pixels, with a see-through background. The folder of your DLL is a good place for it.

```csharp
string folder = System.IO.Path.GetDirectoryName(GetType().Assembly.Location);
ItemRegistry.Define(ModGuid, "spicy-sewer-jerky", new ItemDefinition
{
    TemplateId = 19990,
    IconPath = System.IO.Path.Combine(folder, "jerky.png")
});
```

**Replace the icon of an existing item** for as long as the game runs, with [`ItemIcons`](item-icons.md). Nothing is saved, so removing the mod brings back the original icon.

```csharp
void ReplaceIcon(int itemId, string folder)
{
    ItemIcons.OverrideVanilla(itemId, System.IO.Path.Combine(folder, "other-icon.png"));
}
```

**Get the number a key will have**, for example to put your item in a recipe of another mod. The same [mod id](../../glossary.md#mod-id) and key always give the same number.

```csharp
int id = ItemRegistry.IdFor("example.spicy-jerky", "spicy-sewer-jerky");
```

**Clear the stacks of a removed mod.** A removed mod can leave placeholder stacks (see [Missing items](missing-items.md)). This snippet lists them in the player's inventory and in the containers that are loaded. It then clears the ones that belong to your own mod. Do this after a save has loaded, for example from a handler of [`Events.Loaded`](../../api-core/events.md).

```csharp
void Cleanup()
{
    foreach (MissingItemInfo missing in ItemRegistry.MissingItems())
        Logger.LogInfo(missing.Amount + " x " + missing.Title + " in " + missing.Where);
    ItemRegistry.RemoveMissingItems("example.spicy-jerky");
}
```
