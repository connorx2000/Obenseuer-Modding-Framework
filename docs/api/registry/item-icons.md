---
title: Item icons
parent: Registry
grand_parent: Items and crafting
nav_order: 4
---

# Item icons

`ItemIcons` swaps the icon of any item for your own PNG while the game runs. Use it when an existing item should show a different picture.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>ItemIcons</code> <span>replaces item pictures</span></p>

Call `OverrideVanilla` with an item number and the path to a PNG. That item's icon is replaced while the game runs.

<div class="api-pair" markdown="1">

```csharp
public static class ItemIcons
{
    public static bool OverrideVanilla(int itemId, string pngPath);
    public static bool IsAvailable { get; }
}
```

| Method | Returns | What it does |
|---|---|---|
| `OverrideVanilla(itemId, pngPath)` | `bool` | Replaces the icon of any item. `true` when the change was accepted. |

| Property | Type | What it means |
|---|---|---|
| `IsAvailable` | `bool` | `false` when the icon changes could not be set up. |

</div>
</div>

## Things to know

- `OverrideVanilla` works on any item, from the game or from a mod. Nothing is saved, so removing the mod brings back the original icon.
- The icon is put back after the game reloads its item list. A second call for the same item replaces the earlier path, and the replacement is written to the [log](../../guides/getting-started/logging.md).
- `pngPath` is the full path to a square PNG of 128 or 256 pixels with a see-through background. The same padding, size limits and fallback as for [`ItemDefinition.IconPath`](items.md) apply.
- Call it from your mod's normal code, not from a background thread you started. Normal code runs on the [main thread](../../glossary.md#main-thread) (the one thread the game uses).

## Limits

- It returns `false` when it can't do the job:
  - `IsAvailable` is `false`;
  - the image can't be loaded;
  - the game's item list is built and no item has that number;
  - that item has no picture.
- If the item list is not built yet, the change is remembered and applied when the items load. The call returns `true`.
- A path that is not a full path throws an `ArgumentException`.
