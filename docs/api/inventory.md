---
title: Inventory
parent: Items and crafting
grand_parent: API reference
has_children: true
nav_order: 6
---

# Inventory

[`PlayerInventory`](inventory/player-inventory.md) and [`Containers`](inventory/containers.md) let your mod work with the items the player carries and the storage in the world. Use them for quest rewards, item costs, pickup reactions and looking into chests.

## What it is

This API covers the player's inventory and the storage in the world around it.

- The player carries items in a main grid, a backpack and a few equipment slots (hat, watch, hands).
- Containers are the chests, lockers, shelves and similar storage the player can open.

With it, a mod can count, give and take items, and see what the player holds in each hand. It can also watch items come and go, and look inside containers.

## When to use it

- Give the player a reward item when a quest step is done.
- Take a required item (a key, a part) from the player before something happens.
- Show a message when the player picks up a certain item.
- Look into a container, for example to show what a shop's storage holds.

## Quick example

This mod waits until a save is loaded. Then it [logs](../guides/getting-started/logging.md) how many bottles the player has and how much room is left, and gives two more. It also logs every item that is added to or removed from the inventory. `Items.Find` finds an item by its name, and the other calls use the item's id (see [Items](items.md)).

```csharp
using ObenseuerCore;

[ModInfo("example.inventorytour", "Inventory Tour", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class InventoryTour : FrameworkMod
{
    protected override void OnLoad()
    {
        PlayerInventory.ItemAdded += args => Log.Info("added " + args.Count + " of item " + args.ItemId);
        PlayerInventory.ItemRemoved += args => Log.Info("removed " + args.Count + " of item " + args.ItemId);
        Events.WhenReady(Tour);
    }

    private void Tour()
    {
        ItemDef bottle = Items.Find("Empty Glass Bottle");
        if (bottle == null) return;

        Log.Info("bottles: " + PlayerInventory.Count(bottle.Id) + ", free slots: " + PlayerInventory.FreeSlots + "/" + PlayerInventory.Capacity);
        if (PlayerInventory.Give(bottle.Id, 2)) Log.Info("gave 2 bottles");
        else Log.Warn("no room for 2 bottles");
    }
}
```

`Give` refuses, and adds nothing, when the whole amount does not fit, so always check what it returns. `Count`, `Give` and `Take` work with item ids, never names. `ItemAdded` and `ItemRemoved` are events: the framework calls your code each time they happen. [`Events.WhenReady`](core-helpers/events-whenready.md) runs `Tour` once a save is loaded. See [Using events](../guides/getting-started/using-events.md) for more on events and `WhenReady`.

How the `PlayerInventory` calls behave:

- `Count` counts the stacks whose item id equals `itemId`. It looks in every inventory slot, the backpack and character slots included.
- `Has` uses the game's `CheckForItem`. That check ignores ownership and counts plain item stacks only.
- `Give` refuses (returns `false`, adds nothing) unless the whole amount fits in the main inventory grid.
  - When the amount fits, `Give` uses the game's dialogue add (`Inventory.AddItemDialogueAmount`). That gives the same stack rules and the game's "Items added!" notification.
  - That add only reaches the main grid, never the equipped backpack or the equipment slots. It drops anything that does not fit on the ground and still returns true.
  - So the fit check covers the main grid only. A full grid with free backpack space makes `Give` return `false`.
  - Money ids `0` and `1` are refused. Use [`Wallet`](../api-core/wallet.md) for money.
- `Take` returns `false` unless the whole amount is held.
  - When it is held, `Take` uses the game's dialogue remove (`Inventory.RemoveItemDialogueAmount`). That takes one unit at a time from the first matching stack in slot order: main grid, then backpack, then equipment slots.
  - It does not prefer spoiled, low-quality or own items.
  - Taking a stack with `ownerId > 0` while a [witness](../glossary.md#witness) (a person or camera that can see the player) sees it commits theft. This follows the game's own crime rule. Check `Items()` first when that matters.
  - Money is not kept as item stacks, so `Take` returns `false` for ids `0` and `1`. Use `Wallet`.
- `Take` counts what the game actually removed.
  - The game's remove call takes one unit per call and ignores a refusal. So a take can stop short of the amount asked for.
  - The framework reads the player's count before and after. When the count did not move by exactly `count`, `Take` returns `false` and adds the removed units back through the same dialogue add.
  - The give back restores the counted units only. They come back unowned, at the default quality and meta, without the spoilage state they had.
  - During the give back the player sees the "Items added!" notification, and `ItemAdded` fires. So a rolled back take is not lossless.
  - When the give back itself fails, the framework logs an error with the item id and the units that could not be returned. Those units are lost from the player's inventory.
  - A rolled back take gives the units back, but does not undo a theft the game recorded while it removed them. The game records the theft inside the removal itself.
  - Set `Debug.InventoryTakeSelfCheck` to log a check of that accounting at start.
- `Take` refuses an id that is a liquid, or that any of the player's slots holds as a liquid (`ItemLiquidData.liquidItem.ID` of a slot's stack). It logs one warning naming the id.
  - The reason: the game's remove also matches a container whose liquid id equals the id, and it can match that container before a plain stack.
  - `Count` compares plain stacks only. So such a take could destroy a whole container while the counted amount does not move, and nothing would be reported.
  - The check is the framework's own. Only an in-game take with a real liquid id can confirm the game's match order.
  - For liquid units, use the container facade on this page or [`StackActions`](stackactions.md).
- `Items()` lists the non-empty stacks in every inventory slot, backpack and character slots included.
- `FreeSlots` and `Capacity` count the same slots as `Items()`. So they include the backpack and the equipment slots (hat, watch, hands, backpack slot). `FreeSlots` is not the room `Give` can use, which is the main grid only.
- `GetHeldItemId` returns the id of the item in the right or left hand (`PlayerHandItems`), or `-1`. The game does not track a stack count for a hand item.
- `EquipHeld(modId, itemId)` puts one unowned unit from the main grid into a hand.
  - The item's category picks the hand: `Wieldable Primary` is the right hand, `Wieldable Secondary` the left.
  - It calls the game's `Inventory.EquipItem`, the call that `ItemEquipable.Invoke` makes. So the hand's `OnDropped` and `OnTaken` callbacks run as in the game.
  - It refuses (returns `false`, changes nothing) in these cases:
    - the item has no hand category
    - the main grid holds no single unowned unit of it
    - the hand's slot rules refuse the item
    - the hand holds an owned item
    - the item or the other hand is two-handed, and the other hand is full
    - the [mod id](../glossary.md#mod-id) is invalid
    - during a save or a load
    - from an `ItemAdded` or `ItemRemoved` handler nested deeper than 8 levels
- `UnequipHeld(modId, leftHand)` moves the item in that hand into the main grid, through the game's `Inventory.Unequip`.
  - It refuses an empty hand.
  - It refuses an owned item, because the game does not let the player take off an owned worn item.
  - It refuses an `Unstorable` item, because the game shows no Unequip button for it.
  - It returns `true` only when the hand is empty afterwards. So a full main grid gives `false`, and the item stays in the hand.
- Hats, faces, watches and backpacks are not held items. Use [`Wardrobe.Equip`](wardrobe.md) and `Wardrobe.Unequip` for them. They use the same game call.
- An equip or unequip made through the framework raises `ItemAdded` for each stack the game adds to a player slot. It raises no `ItemRemoved`, because the game clears the source stack without its take method.
  - A swap into an occupied hand raises two `ItemAdded`.
  - `Count` does not change, so count the stacks rather than summing the events.
- `EquipHeld` does not run the item action's side effects.
  - `ItemEquipable.Invoke` first calls the base `Item.ItemAction.Invoke`. That plays the use sound and raises the `ItemConsumed` event and the achievement check. Then `ItemEquipable.Invoke` plays the item sound.
  - The facade skips all of them.
  - Owned stacks are refused, so the owner theft check in the base call does not apply.
- `ItemAdded` and `ItemRemoved` fire from the game's slot add and take methods, filtered to player slots (main grid, backpack, equipment slots). Their payload is `ItemChangedArgs`, with `ItemId` and `Count`.
  - `Give` and `Take` raise one event per item id for the whole operation. `Take(id, 5)` raises one `ItemRemoved(id, 5)`.
  - Game code that adds or removes items itself (pickups, crafting, consuming) still raises one event per game call.
  - Crafting or selling removes one unit per call, so sum the counts.
- Some changes raise no event:
  - inventory drag and drop
  - item transfers started from the container screen
  - shift-click moves into a container (shift-click moves out of a container raise `ItemAdded`)
  - the ground drop from the drag area
  - removals that bypass the slot take method: quality or liquid removals, and `Storage.RemoveItem` on a backpack
- Equips and unequips made through `EquipHeld`, `UnequipHeld`, `Wardrobe.Equip` and `Wardrobe.Unequip` raise `ItemAdded` only (see the held item bullets above).
- The framework does not gate `ItemAdded` or `ItemRemoved` on a save load (see Save loads, below). Treat the events as a feed of script and gameplay changes, and use `Count` for the truth.
- `Give` and `Take` return `false` when they are called from an `ItemAdded` or `ItemRemoved` handler that is nested deeper than 8 levels. The first refusal is logged once.
- `Give` and `Take` also return `false` while a trader trade runs (see [Traders](traders.md)).
- If the game's own add finds the inventory full, it drops the rest on the ground. Those dropped units are not counted: `ItemAdded` reports only what went into the inventory. `Give` also checks fit before it adds.

**Common problems**

- `Give` or `Take` returns `false` for ids `0` and `1`: those ids are money, not items. Use `Wallet`.
- The summed `ItemAdded` and `ItemRemoved` counts do not match `Count`: some changes raise no event, and equips and unequips raise `ItemAdded` only (see the bullets above). Use `Count` for the truth.

## ItemStackInfo

[`ItemStackInfo`](inventory/item-stacks.md) is one stack of an item. `PlayerInventory.Items()`, `Containers.GetItems` and `Containers.TakeStack` give you these.

```csharp
public sealed class ItemStackInfo
{
    public int ItemId { get; }
    public int Count { get; }
    public int Slot { get; }             // storage slot index for GetItems and TakeStack, -1 when none
    public int OwnerId { get; }          // -1 for the player's own items
    public float Quality { get; }        // -1 when the item has no quality data
    public int IntegrityPercent { get; } // -1 when the item does not spoil
    public int LiquidItemId { get; }     // -1 when there is no liquid
    public float LiquidAmount { get; }
    public bool IsStolen { get; }        // the stack's own stolen flag, see below
    public bool HasOwnerPermission { get; }
    public float Charge { get; }         // -1 when the item has no ammo data
    public float MaxCharge { get; }      // -1 when the item has no ammo data
    public float WalletCoins { get; }    // -1 when the item has no wallet data
    public int WalletOc { get; }         // -1 when the item has no wallet data
    public bool IsWritable { get; }      // true when the item has writable data
    public string WrittenTitle { get; }  // null when none
    public string WrittenText { get; }   // null when none
}
```

- `Slot` is the storage slot index of a stack from `GetItems`, its position in `storageSlots`. It is the value to pass as `slot` to `Containers.TakeStack` or [`PlayerShops.Sell`](economy/player-shops.md).
- `Slot` is -1 for stacks not in a container slot: `PlayerInventory.Items`, [`StackActions.StolenStacks`](stackactions.md) and trash piles from [`Cleaning.TrashItems`](cleaning.md).
- Dishwasher contents from [`Dishwashers.Contents`](dishwashers.md) come from `GetItems`, so they carry the index too.
- No framework code reads `Slot`. `Containers.TakeStack` and `PlayerShops.Sell` take the index as their own argument and refuse -1.
- The game stores no spoil timer. `IntegrityPercent` is the game's own integrity (`ItemPerishabledata.GetIntegrityPercent`), which falls by the item's loss per hour.
- `LiquidAmount` is the game's `ItemLiquidData.GetLiquidAmount` for the stack size.
- `IsStolen` is the stack's own `ItemStolenData.playerStolen` flag.
  - The game sets it when a stack with an owner id of 2 or more enters the player's inventory with no owner permission. It does so whether or not a [witness](../glossary.md#witness) saw it.
  - A witness only decides whether the theft crime is also committed.
  - The flag does not show a crime, a witness or an owner's claim, so it is not the game's crime state.
  - The game's mental-health expectation reads it for a used item.
- `HasOwnerPermission` is true when the stack has an `ItemOwnerPermission` with `ownerPermission` set. Every meta entry is checked. The class defaults `ownerPermission` to true, so presence usually means permitted. `allowUse` is not exposed.
- `Charge` and `MaxCharge` are the game's `ItemAmmodata.ammo` and `maxAmmo` for the stack.
- `WalletCoins` is `ItemOSCoinWalletData.coins`. `WalletOc` is its `ToOC()`, the game's rounded [OC](../glossary.md#oc) (the game's everyday currency) value.
- `IsWritable` is true when the stack has `ItemWritableData`. So a note with no title can be told apart from a stack with no writable data. `WrittenTitle` and `WrittenText` are the game's `title` and `content`.

## Containers

[`Containers`](inventory/containers.md) finds the storage in the scene, reads and changes what is inside, and reports when one opens or closes. Each container is a `ContainerInfo` snapshot.

```csharp
public sealed class ContainerInfo
{
    public string Guid { get; }
    public int InstanceId { get; }
    public string Name { get; }          // the game's GetTitle(), which adds " (Empty)" when empty
    public Vector3 Position { get; }
    public int Apartment { get; }        // -1 when not in an apartment
    public int OwnerId { get; }          // the NPC owner, -1 when none
    public bool NpcOwned { get; }        // owner id > 1, the game's "Steal" condition
    public bool Locked { get; }
}

public static class Containers
{
    public static List<ContainerInfo> All();
    public static ContainerInfo FindByGuid(string guid);
    public static string GuidOf(ContainerInfo container);
    public static List<ItemStackInfo> GetItems(ContainerInfo container);
    public static bool Give(ContainerInfo container, int itemId, int count);
    public static bool Take(ContainerInfo container, int itemId, int count);
    public static ItemStackInfo TakeStack(ContainerInfo container, int slot, int count);
    public static bool IsNotNpcOwned(Storage storage);
    public static bool IsLocked(Storage storage);
    public static int ApartmentOf(Component component);

    public static event Action<ContainerInfo> Opened;
    public static event Action<ContainerInfo> Closed;
}
```

- `All` finds every active `Storage` in the active [scene](../glossary.md#scene) with `FindObjectsByType`, then builds a snapshot per container. That call scans the whole scene. Call `All` once and cache the result; do not call it every frame.
- `All` includes shops, vending machines, prisons and the equipped backpack's storage, so check `Name`.
- `ContainerInfo` is a snapshot: `Locked` and `Name` are not updated. `Give` and `Take` check the lock again.
- `GetItems` reads the game's `storageSlots`, skips empty slots, and sets each stack's `Slot` to its index there. Pass `info.Slot` to `TakeStack` or [`PlayerShops.Sell`](economy/player-shops.md) to take that exact stack.
- `Give` refuses when the container is locked, or when no slot can hold the whole amount (checked against every slot).
  - Otherwise it calls the game's `Storage.AddItem`, which places into slot 0 and its siblings.
  - In an unusual slot layout that call could still drop the overflow in the world.
  - Giving to an NPC-owned container is allowed, as in the game.
- `Take` refuses when the container is locked, when it is NPC-owned and `PermissionToTake` is false, or when it does not hold the whole amount.
  - Otherwise it calls the game's `Storage.RemoveItem` and refreshes the container's slots.
  - The removed items are destroyed, not moved. Nothing is added to the player's inventory.
  - A mod that wants a transfer must `PlayerInventory.Give` the same amount itself, and should do so first.
  - The game shows `Steal` for NPC-owned containers. Its theft path is not a public method a facade can call, so the facade refuses rather than bypass it.
- `TakeStack(container, slot, count)` takes `count` items from the stack in slot `slot`, the index in the game's `Slots` order. It returns that stack as an `ItemStackInfo` with its owner and meta.
  - As with `Take`, the items are destroyed, not moved to the player.
  - Pass `ItemStackInfo.Slot` from `GetItems` as `slot`. -1 is refused.
  - It returns `null` when the container is locked, or is NPC-owned and `PermissionToTake` is false.
  - It returns `null` when the slot is out of range or empty, when the stack holds fewer than `count`, or when a save is busy.
  - It also returns `null` when the slot's stack is no longer the one `GetItems` reads (the container's slots were rebuilt). So list again before taking.
- `IsNotNpcOwned(storage)` answers only that no NPC owns the container.
  - It is not a permission check, and it does not mean the container belongs to the player.
  - It is the negation of `ContainerInfo.NpcOwned`. It is `true` when the container's `StorageOwner` is null or its id is 1 or less.
  - That is the game's own rule for showing `Steal` on the use prompt, minus the private flag that hides that text.
  - A shop shelf, a vending machine, a prison locker and the player's own crates have no owner, and all report `true`. So do not use it on its own to decide whether a mod may take items.
  - An NPC-owned container that grants `PermissionToTake` reports `false`, even though `Containers.Take` and `Containers.TakeStack` allow it. `Containers.Give` does not read ownership at all.
  - A null storage reports `false` and throws nothing.
- `IsLocked(storage)` is the container's own lock flag, and `false` for a null storage. It is the same check `Give` and `Take` make, so a hotkey can test it before it calls them.
- `ApartmentOf(component)` returns the apartment number of the tenement prefab above the component. When the component has no prefab of its own, it returns the apartment number of its `FurnitureManager` instead.
  - It returns `-1` when neither exists or the component is null.
  - `ContainerInfo.Apartment` already carries this value for a container. Use `ApartmentOf` for furniture, doors and other objects.
- `FindByGuid(guid)` returns the container whose save GUID is `guid`, or `null`.
  - It first uses the game's GUID registry, the same lookup `SaveData.CaptureByGuid` and `SaveData.ApplyByGuid` use.
  - The game fills that registry only for objects it matched to an entry in the loaded save.
  - When the registry has no live `Storage` for the GUID, `FindByGuid` scans the live scene with `SaveController.GetObjectByGUID(guid, typeof(Storage))`.
  - That fallback calls `FindObjectsOfType` once, so each miss costs one whole-scene scan. Do not call it every frame.
  - It returns `null` for an empty or null GUID (no scan), for an unknown GUID, and for a destroyed container.
- `GuidOf(container)` returns `container.Guid`, or `null` for `null`. An empty string means the container has no GUID.
- `Opened` and `Closed` fire once per open and once per close of a `Storage` container. They carry a `ContainerInfo` snapshot built as `All` builds it.
  - `Opened` fires when the game opens the container's storage.
  - `Closed` fires when the game closes it, or when the container is disabled or destroyed while open (a scene change).
  - The framework tracks the one container it has open. So a repeated close raises nothing, and opening a container while another is open raises `Closed` for the old one first.
  - Use `container.Guid` (or `GuidOf`) to identify it on another PC.
- Two close paths raise `Closed`:
  - the menu close (the game's `HideGameMenu`). It runs for the "Pause Menu" binding while a menu is open, the menu's own close buttons and the other close calls.
  - the container's own exit. Crafting, vending machines, trades and backpacks close their storage themselves.
- Walking away is not a separate path. While a container menu is open the game disables movement, so the player cannot walk away from it.
- A container opened without a menu is closed by its owner, which the exit hook covers.
- `Opened` and `Closed` also cover the storages behind crafting stations, vending machines, trades and the equipped backpack. The game opens them through the same `Storage` methods. `LiquidStorage` (bottles and tanks) has its own enter and exit, and raises nothing.
- `Opened` and `Closed` mean the storage was entered or exited, not that a container menu is visible.
  - `Storage.EnterExternally` runs its side effects even when no menu shows. These include setting `Storage.active` and the `Opened` raise.
  - The menu is shown only when the `isInteractable | forceOpen` check at `Storage` passes.
- Backpack, cooler bag and rucksack storages, crafting input and output storages, and trade and vending storages fire them too.
- With container A open, opening the backpack raises `Closed(A)`, then `Opened(backpack)`. Closing the backpack raises `Closed(backpack)`, then `Opened(A)`, because `BackpackStorage.CloseBackpack` re-interacts with the last used container. A lock-while-open mod sees A close and reopen on every backpack toggle.
- A mod that tracks world containers should ignore entries with an empty `Guid`. Those have no save identity, so `FindByGuid` cannot return them and another PC cannot match them.
- Handlers run synchronously on the [main thread](../glossary.md#main-thread). A handler that throws is logged, and the other handlers still run. The framework does not gate these events on a save load (see Save loads, below).

**Common problems**

- `Containers.Take` returns `true`, but the player gets nothing: `Take` destroys the items. Call `PlayerInventory.Give` with the same amount first.
- `TakeStack` returns `null` for a stack that `GetItems` listed: the causes are in the `TakeStack` bullet above. When the container's slots were rebuilt, list again before taking.

## Example

Run this after a save is loaded, from a test [plugin](../glossary.md#plugin) on the [main thread](../glossary.md#main-thread). `ITEM_ID` is a placeholder for a real item id.

```csharp
int id = ITEM_ID;   // placeholder: a real item id (0 and 1 are money)
Log.LogInfo("count " + PlayerInventory.Count(id) + " has1 " + PlayerInventory.Has(id, 1) + " free " + PlayerInventory.FreeSlots + "/" + PlayerInventory.Capacity);
Log.LogInfo("give " + PlayerInventory.Give(id, 2) + " count now " + PlayerInventory.Count(id));
Log.LogInfo("take " + PlayerInventory.Take(id, 2) + " count now " + PlayerInventory.Count(id));
Log.LogInfo("stacks " + PlayerInventory.Items().Count + " held right " + PlayerInventory.GetHeldItemId(false));
Log.LogInfo("containers " + Containers.All().Count);
```

Expected: `give` is true and the count goes up by 2, then `take` is true and the count goes back. The `ItemAdded` and `ItemRemoved` handlers log each event.

## Save loads

`ItemAdded` and `ItemRemoved` are not gated on a save load. A slot add or take made while a save loads still fires. A handler that should ignore events raised while a save loads can check [`SaveData.Busy`](../api-core/savedata.md#savedatabusy). It is true while a save is loading or being written.
