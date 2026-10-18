---
title: Player inventory
parent: Inventory
grand_parent: Items and crafting
nav_order: 3
---

# Player inventory

`PlayerInventory` counts, gives and takes the player's items, reports what is in each hand, and tells you when items are added or removed. Items are named by their item id (see [Items](../items.md)). Money is not an item here: ids `0` and `1` are money, and money goes through `Wallet` (see the [core API page](../../api-core/wallet.md#wallet)).

<div class="api-type" markdown="1">
<p class="api-type-head"><code>PlayerInventory</code> <span>the player's items</span></p>

The class you start from. It counts, gives and takes items, shows what the player holds in each hand, and raises `ItemAdded` and `ItemRemoved`.

<div class="api-pair" markdown="1">

```csharp
public static class PlayerInventory
{
    public static int FreeSlots { get; }
    public static int Capacity { get; }
    public static int Count(int itemId);
    public static bool Has(int itemId, int count);
    public static bool Give(int itemId, int count);
    public static bool Take(int itemId, int count);
    public static List<ItemStackInfo> Items();
    public static int GetHeldItemId(bool leftHand);
    public static bool EquipHeld(string modId, int itemId);
    public static bool UnequipHeld(string modId, bool leftHand);
    public static bool IsAvailable { get; }
    public static event Action<ItemChangedArgs> ItemAdded;
    public static event Action<ItemChangedArgs> ItemRemoved;
}
```

| Method | Returns | What it does |
|---|---|---|
| `Count(itemId)` | `int` | How many of this item the player has. |
| `Has(itemId, count)` | `bool` | `true` when the player has at least this many. |
| `Give(itemId, count)` | `bool` | Gives the player items. `true` when all of them were added. |
| `Take(itemId, count)` | `bool` | Takes items from the player. `true` when all of them were taken. |
| `Items()` | `List<ItemStackInfo>` | Every stack the player carries, as [`ItemStackInfo`](item-stacks.md). |
| `GetHeldItemId(leftHand)` | `int` | The id of the item in a hand. `-1` when the hand is empty. |
| `EquipHeld(modId, itemId)` | `bool` | Puts one item from the main grid into a hand. |
| `UnequipHeld(modId, leftHand)` | `bool` | Moves the item in a hand back into the main grid. |

| Property | Type | What it means |
|---|---|---|
| `FreeSlots` | `int` | The number of empty slots. |
| `Capacity` | `int` | The number of slots in all. |
| `IsAvailable` | `bool` | `false` when the added and removed events could not be set up. |

| Event | Gives you | When it fires |
|---|---|---|
| `ItemAdded` | `ItemChangedArgs` | Items went into the player's slots. |
| `ItemRemoved` | `ItemChangedArgs` | Items came out of the player's slots. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>ItemChangedArgs</code> <span>what you get when items are added or removed</span></p>

`ItemAdded` and `ItemRemoved` give you this. `Count` is how many went in or out for that one event, so add the counts up for a total.

<div class="api-pair" markdown="1">

```csharp
public sealed class ItemChangedArgs
{
    public int ItemId { get; }
    public int Count { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `ItemId` | `int` | The id of the item. |
| `Count` | `int` | How many were added or removed. |

</div>
</div>

## Things to know

- `Count(id)` counts that item in every slot the player has, the backpack and the equipment slots included.
- `Has(id, count)` is `true` when the player has at least that many. It ignores who owns the items, and it counts ordinary item stacks only. <span class="tech">(It uses the game's `CheckForItem`.)</span>
- `Give` adds the whole amount or nothing. It returns `false` and adds nothing unless the whole amount fits in the main grid. It also shows the game's "Items added!" notice. <span class="tech">(It uses the game's dialogue add (`Inventory.AddItemDialogueAmount`) after a fit check of the main grid. That game call drops what does not fit on the ground and still returns `true`, which is why the fit check comes first.)</span>
- `Take` removes the whole amount or nothing. It returns `false` unless the player holds the whole amount.
  - It takes one unit at a time from the first matching stack in slot order: the main grid, then the backpack, then the equipment slots.
  - It does not prefer spoiled, low-quality or the player's own items. <span class="tech">(It uses the game's dialogue remove (`Inventory.RemoveItemDialogueAmount`). A stack with `ownerId > 0` is an owned stack.)</span>
- `Items()` lists the stacks that are not empty, in every slot, the backpack and the character slots included.
- `FreeSlots` and `Capacity` count the same slots as `Items()`. That includes the backpack and the equipment slots (hat, watch, hands and the backpack slot). `FreeSlots` is not the room that `Give` can use, because `Give` uses the main grid only.
- `GetHeldItemId` gives the id of the item in the right or left hand, or `-1`. The game keeps no count for a held item.
- `EquipHeld(modId, itemId)` puts one item from the main grid into a hand. The item's category picks the hand: `Wieldable Primary` is the right hand and `Wieldable Secondary` is the left. The hand's drop and take reactions run as they do in the game. It returns `false` and changes nothing in these cases: <span class="tech">(It calls the game's `Inventory.EquipItem`, the call that `ItemEquipable.Invoke` makes.)</span>
  - The item has no hand category.
  - The main grid has no single unit of it that nobody owns.
  - The hand's own rules refuse the item.
  - The hand holds an item that belongs to someone.
  - The item, or the item in the other hand, needs both hands and the other hand is full.
  - The [mod id](../../glossary.md#mod-id) is not valid.
  - A save is loading or being written.
  - A handler for `ItemAdded` or `ItemRemoved` calls it more than 8 levels deep.
- `UnequipHeld(modId, leftHand)` moves the item in that hand into the main grid.
  - It refuses an empty hand.
  - It refuses an item that belongs to someone, because the game does not let the player take off an owned worn item.
  - It refuses an item that cannot be stored, because the game shows no Unequip button for it. <span class="tech">(It calls the game's `Inventory.Unequip`. An item that cannot be stored has the `Unstorable` category.)</span>
  - It returns `true` only when the hand is empty afterwards. So a full main grid gives `false`, and the item stays in the hand.
- Hats, masks, watches and backpacks are not held items. Use `Wardrobe.Equip` and `Wardrobe.Unequip` for those (see [Wardrobe](../wardrobe.md)). They use the same game call.
- `ItemAdded` and `ItemRemoved` fire when items go into or come out of the player's slots: the main grid, the backpack and the equipment slots. <span class="tech">(They are raised after the game's slot add and take methods (`SlotController.AddItemAndReturnRemaining` and `SlotController.TakeItem`), filtered to player slots.)</span>
  - `Give` and `Take` raise one event for each item id for the whole call. `Take(id, 5)` raises one `ItemRemoved` with a count of 5.
  - When the game itself adds or removes items (pickups, crafting, using things up), you get one event for each game call.
  - Crafting and selling remove one unit per call, so add the counts up.
- The events do not hold back while a save is loading. An add or take made while a save loads still fires. A handler that should ignore those can check [`SaveData.Busy`](../../api-core/savedata.md#savedatabusy), which is `true` while a save is loading or being written.
- `Give` and `Take` return `false` while the player is in a trade with a trader. See [Traders](../traders.md).

## Limits

- Pass a count above zero. A count of `0` or less throws an `ArgumentException`. Nothing else throws.
- `Give` only fills the main grid, never the equipped backpack or the equipment slots. So a full main grid with free backpack space makes `Give` return `false`. Money ids `0` and `1` are refused: use `Wallet`.
- Some stacks belong to someone else. Taking one while a [witness](../../glossary.md#witness) (a person or camera that can see the player) sees it is theft, the same as in the game. Check `Items()` first when that matters.
- Money is not kept as item stacks, so `Take` returns `false` for ids `0` and `1`: use `Wallet`.
- `Give` and `Take` return `false` when an `ItemAdded` or `ItemRemoved` handler calls them more than 8 levels deep. The first refusal goes to the [log](../../guides/getting-started/logging.md).
- `EquipHeld` does not do what the game does when the player uses the item. It plays no use sound, raises no consumed event, skips the achievement check, and plays no item sound. Items that belong to someone are refused, so the theft check that goes with ownership does not apply. <span class="tech">(`ItemEquipable.Invoke` first calls the base `Item.ItemAction.Invoke`, which plays the use sound and raises `ItemConsumed` and the achievement check, then plays the item sound. `EquipHeld` skips all of these.)</span>
- Equipping and unequipping raise `ItemAdded` but not `ItemRemoved`. This holds for `EquipHeld`, `UnequipHeld`, `Wardrobe.Equip` and `Wardrobe.Unequip`. The game clears the source stack without calling its normal take method.
  - Swapping into a hand that is already full raises two `ItemAdded`.
  - `Count` does not change, so count the stacks rather than adding up the events.
- Some moves raise no event at all:
  - dragging items around the inventory
  - item transfers started from the container screen
  - shift-click moves into a container (shift-click moves out of a container raise `ItemAdded`)
  - dropping an item on the ground from the drag area
  - removals that skip the slot's take method: quality or liquid removals, and `Storage.RemoveItem` on a backpack
- Treat the events as a feed of script and gameplay changes, and use `Count` for the truth.
- Items that the game drops on the ground because the inventory was full are not counted. `ItemAdded` reports only what went into the inventory.

**Common problems**

- `Give` or `Take` returns `false` for ids `0` and `1`: those ids are money. Use `Wallet`.
- The added-up `ItemAdded` and `ItemRemoved` counts do not match `Count`: some moves raise no event, and equips and unequips raise `ItemAdded` only (see Limits). Use `Count` for the truth.
- `Give` returns `false` while `FreeSlots` is above 0: `FreeSlots` also counts the backpack and the equipment slots, but `Give` fills the main grid only.
- Your handlers fire while a save loads: the events do not hold back during a load. Check `SaveData.Busy` in the handler.
