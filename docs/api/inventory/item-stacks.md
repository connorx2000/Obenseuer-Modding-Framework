---
title: Item stacks
parent: Inventory
grand_parent: Items and crafting
nav_order: 4
---

# Item stacks

An `ItemStackInfo` is one stack of an item, with everything the game knows about it. It is a snapshot taken when you asked.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>ItemStackInfo</code> <span>one stack of an item</span></p>

[`Containers.GetItems`](containers.md) and `Containers.TakeStack` give you this, and so does [`PlayerInventory.Items()`](player-inventory.md). `Slot` is `-1` when the stack is not in a container slot.

<div class="api-pair" markdown="1">

```csharp
public sealed class ItemStackInfo
{
    public int ItemId { get; }
    public int Count { get; }
    public int Slot { get; }
    public int OwnerId { get; }
    public float Quality { get; }
    public int IntegrityPercent { get; }
    public int LiquidItemId { get; }
    public float LiquidAmount { get; }
    public bool IsStolen { get; }
    public bool HasOwnerPermission { get; }
    public float Charge { get; }
    public float MaxCharge { get; }
    public float WalletCoins { get; }
    public int WalletOc { get; }
    public bool IsWritable { get; }
    public string WrittenTitle { get; }
    public string WrittenText { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `ItemId` | `int` | The id of the item. |
| `Count` | `int` | How many items the stack holds. |
| `Slot` | `int` | The slot to pass to `Containers.TakeStack`. `-1` when the stack is not in a container slot. |
| `OwnerId` | `int` | Who owns the stack. The game gives `-1` to a stack nobody owns, and treats any value below 1 as no owner. |
| `Quality` | `float` | The quality of the stack. `-1` when the item has no quality. |
| `IntegrityPercent` | `int` | How much condition is left, in percent. `-1` when the item does not spoil. |
| `LiquidItemId` | `int` | The id of the liquid inside. `-1` when there is none. |
| `LiquidAmount` | `float` | How much liquid is inside. |
| `IsStolen` | `bool` | `true` when the game marked the stack as stolen by the player. |
| `HasOwnerPermission` | `bool` | `true` when the item's owner has given permission to take the stack. |
| `Charge` | `float` | The charge or ammo now. `-1` when the item has no charge. |
| `MaxCharge` | `float` | The most charge or ammo the item holds. `-1` when the item has no charge. |
| `WalletCoins` | `float` | The coins in a coin wallet. `-1` when the item is not one. |
| `WalletOc` | `int` | The wallet's coins as [OC](../../glossary.md#oc), rounded. `-1` when the item is not a wallet. |
| `IsWritable` | `bool` | `true` when the item can hold writing, such as a note. |
| `WrittenTitle` | `string` | The written title. `null` when there is none. |
| `WrittenText` | `string` | The written text. `null` when there is none. |

</div>
</div>

## Things to know

- `Slot` is the position of the stack in a container. It is the number to pass to `Containers.TakeStack` or [`PlayerShops.Sell`](../economy/player-shops.md), and those two calls refuse `-1`. <span class="tech">(`Slot` is the stack's index in the container's `storageSlots`. No framework code reads `Slot`; the two calls take the index as their own argument.)</span>
- `Slot` is `-1` for stacks that are not in a container slot: those from `PlayerInventory.Items`, [`StackActions.StolenStacks`](../stackactions.md) and the trash piles from [`Cleaning.TrashItems`](../cleaning.md).
- Dishwasher contents from [`Dishwashers.Contents`](../dishwashers.md) come from `GetItems`, so they carry the position too.
- `IntegrityPercent` is how much of the item's condition is left. It falls by the item's loss per hour. The game keeps no spoil timer. <span class="tech">(It is `ItemPerishabledata.GetIntegrityPercent`.)</span>
- `LiquidAmount` is the amount of liquid for the whole stack. <span class="tech">(It is `ItemLiquidData.GetLiquidAmount` for the stack size.)</span>
- `IsStolen` is the game's own flag on the stack, not a crime record. The game sets it when the player takes someone else's stack (an owner id of 2 or more) without permission. It does so whether or not anyone saw it.
- A [witness](../../glossary.md#witness) (a person or camera that can see the player) only decides whether the theft crime is also committed. So the flag does not show a crime, a witness or a claim by the owner. The game's mental-health expectation reads it for a used item. <span class="tech">(It is `ItemStolenData.playerStolen`.)</span>
- `HasOwnerPermission` is `true` when the stack carries a permission from its owner that says yes. The game sets that to yes by default, so having one usually means permitted. It does not tell you whether the owner allows use. <span class="tech">(Every meta entry is checked for an `ItemOwnerPermission` with `ownerPermission` set. `allowUse` is not available to mods.)</span>
- `Charge` and `MaxCharge` are the ammo or battery values of the stack. <span class="tech">(They are `ItemAmmodata.ammo` and `maxAmmo`.)</span>
- `WalletCoins` and `WalletOc` describe a coin wallet item. <span class="tech">(`WalletCoins` is `ItemOSCoinWalletData.coins` and `WalletOc` is its `ToOC()`, the game's rounded OC value.)</span>
- `IsWritable` lets you tell a note with no title from a stack that cannot hold writing at all. <span class="tech">(It is true when the stack has `ItemWritableData`. `WrittenTitle` and `WrittenText` are its `title` and `content`.)</span>
