---
title: Give an item
parent: Cookbook
nav_order: 1
---

# Give an item

Press F7 to give the player one item. This recipe finds the item by its title, the name the player sees in the game. Use it when your mod needs to hand the player something.

## The example

The mod registers a key with [`Mod.Key`](../getting-started/keybinds.md), then checks it every frame in `Update`, a method Unity (the game engine) calls on every frame. [`Items.Find`](../../api/items/lookup.md) looks the item up by title. [`PlayerInventory.Give`](../../api/inventory/player-inventory.md) puts it in the player's inventory.

Set `ItemTitle` to the item's title. Capital letters do not matter, but the spelling must match the game's item list. Start from the project you made in [Your first mod](../getting-started/first-mod.md), and replace the code in `Plugin.cs` with the code below.

```csharp
using ObenseuerCore;
using UnityEngine;

[ModInfo("example.giveitem", "Give Item", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public class GiveItem : FrameworkMod
{
    // The title of the item to give.
    private const string ItemTitle = "Empty Glass Bottle";
    // The F7 key. It stays null if it could not be created.
    private ModKeybind giveKey;

    protected override void OnLoad()
    {
        // Register the key: short id, name shown in the Mods menu, default key.
        giveKey = Mod.Key("Give", "Give the item", KeyCode.F7);
    }

    // Unity calls this once per frame.
    private void Update()
    {
        // Skip when there is no key, a menu is open, the game is paused, or the game has switched its keys off.
        if (giveKey == null || UiState.IsMenuOpen || UiState.IsPaused || UiState.IsInputBlocked) return;
        // True only on the frame the key goes down.
        if (!giveKey.Down()) return;
        // Look the item up by title, then add one to the player's inventory.
        ItemDef item = Items.Find(ItemTitle);
        if (item == null || !PlayerInventory.Give(item.Id, 1)) Log.Warn("could not give " + ItemTitle);
    }
}
```

Then build and try it:

1. Close the game and run `dotnet build -c Release -p:DeployToGame=true` in your mod folder.
2. Start the game and load a save.
3. Press F7. The item appears in your inventory.

The F7 key now gives the item. Things to know, below, lists other ways to find an item.

### Common problems

- The item does not appear: open `BepInEx\logs\example.giveitem.log` in the game folder.
  - A warning that starts with `could not give`: `Items.Find` returns `null` for an unknown title, so check the spelling of `ItemTitle`. `PlayerInventory.Give` returns `false` when the main grid is full, or while a save runs.
  - No warning: the code ignores F7 while a menu is open, the game is paused, or the game has switched its keys off. It also ignores F7 when the key could not be created (`giveKey` is `null`). If the mod did not load at all, see [my mod does not load](../../faq.md#my-mod-does-not-load).

## Things to know

- [`Items.Search`](../../api/items/lookup.md) finds titles that contain some text.
- The item id is also in `Obenseuer_Data\StreamingAssets\Items.json`. See [Finding item ids](../content.md#finding-item-ids).

## Where to go next

Next: [Pay the player](pay-the-player.md).
