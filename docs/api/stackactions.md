---
title: Stack actions
parent: Items and crafting
grand_parent: API reference
nav_order: 14
---

# Stack actions

A stack is a pile of one kind of item in the player's inventory. `StackActions` gives the player a stack that is more than a plain item: one with a set quality, one that belongs to someone else (so it counts as stolen), or a container filled with a liquid. Use it when you need more than `PlayerInventory.Give` (see [Inventory](inventory.md)), which only adds plain items.

It can also drop such a stack on the floor and pour a liquid into the containers the player is carrying. It lists the stacks the game counts as stolen, too.

## When to use it

- Give the player a reward with a better quality than normal.
- Hand over an item that counts as stolen, for a story or a crime mod.
- Give a container that is already full of water or another liquid.
- Drop a stack at the player's feet when their inventory is full.
- Find out which stacks the player is carrying that count as stolen.

## Quick example

This mod gives the player two turnips of quality 1.5 once a save is loaded. Turnips that do not fit in the inventory are dropped at the player's feet. It [logs](../guides/getting-started/logging.md) whether it worked.

```csharp
using ObenseuerCore;

[ModInfo("example.turnipgift", "Turnip Gift", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class TurnipGift : FrameworkMod
{
    protected override void OnLoad()
    {
        Events.WhenReady(() =>
        {
            StackSpec spec = new StackSpec { Quality = 1.5f };
            bool given = StackActions.GiveOrDrop(Info.Id, 12, 2, spec);
            Log.Info(given ? "Gave two good turnips." : "Could not give the turnips.");
        });
    }
}
```

`12` is the item number of the turnip. `Info.Id` is your [mod id](../glossary.md#mod-id), which the calls use for logging. [`Events.WhenReady`](core-helpers/events-whenready.md) waits until a save is loaded before it runs your code. See [Using events](../guides/getting-started/using-events.md).

## Common tasks

Give a stack that counts as stolen. Any `OwnerId` above 1 means the stack belongs to that character, so the game treats it as stolen. `Give` adds nothing at all if the whole stack does not fit.

```csharp
StackSpec stolen = new StackSpec { OwnerId = 5 };
bool fitted = StackActions.Give(Info.Id, 12, 3, stolen);
Log.Info(fitted ? "The stolen turnips fit." : "No room, so nothing was added.");
```

Give as many as fit and drop the rest on the floor.

```csharp
StackSpec fine = new StackSpec { Quality = 1.2f };
StackActions.GiveOrDrop(Info.Id, 12, 20, fine);
```

Drop a stack at the player's feet without trying the inventory first.

```csharp
StackActions.Drop(Info.Id, 12, 1, new StackSpec());
```

Give an empty bucket that is already filled with 2 units of clean water.

```csharp
StackSpec water = new StackSpec { LiquidItemId = 10840, LiquidAmount = 2f };
StackActions.Give(Info.Id, 90010, 1, water);
```

Pour clean water into the containers the player already carries, and see how much went in.

```csharp
float poured = StackActions.AddLiquid(Info.Id, 10840, 5f);
Log.Info("Poured " + poured + " of water.");
```

List the stolen stacks the player is carrying.

```csharp
List<ItemStackInfo> stolenStacks = StackActions.StolenStacks();
Log.Info("You are carrying " + stolenStacks.Count + " stolen stacks.");
```

## Reference

This is the full list of what the API gives you. `StackSpec` describes the extra things a stack should have. `StackActions` takes a `StackSpec` with every give and drop call.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>StackSpec</code> <span>the extras for one stack</span></p>

Pass one of these to the give and drop calls to set a stack's `Quality`, `OwnerId` or liquid. Leave a field alone to keep the normal value.

<div class="api-pair" markdown="1">

```csharp
public sealed class StackSpec
{
    public float? Quality { get; set; }
    public int OwnerId { get; set; }
    public int LiquidItemId { get; set; }
    public float LiquidAmount { get; set; }
}
```

| Property | Type | What it means |
|---|---|---|
| `Quality` | `float?` | The stack's quality, above 0 and at most 2. `null` keeps the item's normal quality. |
| `OwnerId` | `int` | Who owns the stack. `-1` (the default) means nobody. Above 1 means it counts as stolen. |
| `LiquidItemId` | `int` | The item number of a liquid to fill a container with. `0` means no liquid. |
| `LiquidAmount` | `float` | How much of that liquid to put in. Must be above 0 when `LiquidItemId` is set. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>StackActions</code> <span>give, drop and pour stacks</span></p>

This is the class to start from. `Give`, `GiveOrDrop` and `Drop` add or drop a stack with a `StackSpec`. `AddLiquid` pours liquid into the containers the player carries. `StolenStacks()` lists the stolen stacks they carry.

<div class="api-pair" markdown="1">

```csharp
public static class StackActions
{
    public static bool Give(string modId, int itemId, int count, StackSpec spec);
    public static bool GiveOrDrop(string modId, int itemId, int count, StackSpec spec);
    public static bool Drop(string modId, int itemId, int count, StackSpec spec);
    public static float AddLiquid(string modId, int liquidItemId, float amount);
    public static List<ItemStackInfo> StolenStacks();
}
```

These calls change what the player carries. Each takes your mod's id first.

| Method | Returns | What it does |
|---|---|---|
| `Give(modId, itemId, count, spec)` | `bool` | Adds the whole stack, or nothing. `true` when added. |
| `GiveOrDrop(modId, itemId, count, spec)` | `bool` | Adds what fits and drops the rest on the floor. `true` when done. |
| `Drop(modId, itemId, count, spec)` | `bool` | Drops the stack at the player's feet. `true` when dropped. |
| `AddLiquid(modId, liquidItemId, amount)` | `float` | Pours liquid into the containers the player carries. Returns how much went in. |
| `StolenStacks()` | `List<ItemStackInfo>` | The stacks the player carries that count as stolen. Each entry is an `ItemStackInfo` (see [Inventory](inventory.md)). |

</div>
</div>

### Things to know

- A quality only works on an item that has a quality. If the item has none, or the number is not above 0 and at most 2, the call fails and gives nothing. <span class="tech">(The item's template must carry quality data (`ItemQualityData`). A `NaN` quality is refused too.)</span>
- A liquid only works on a container, and only with a real liquid. `LiquidItemId` must be an item in the game's liquid category. The item you give must be able to hold a liquid, and `LiquidAmount` must be above 0. Otherwise the call fails and gives nothing. <span class="tech">(The liquid item's categories must contain `Liquid`, and the container's data must include liquid data (`ItemLiquidData`).)</span>
- `Give` adds the stack only when all of it fits. If the inventory has no room for the full `count`, `Give` returns `false` and adds nothing.
- `GiveOrDrop` adds what fits and drops the rest on the floor at the player's feet. Use it when you do not want the reward to get lost.
- `Drop` puts the stack on the floor at the player's feet, the way the game drops an item. It does not try the inventory first.
- `AddLiquid` pours into containers the player already carries. It returns how much liquid went in. That can be less than you asked for if the containers are nearly full. It never creates a container, so with no container it returns `0`. It also returns `0` when `amount` is not above 0. <span class="tech">(It calls the game's `ItemOperations.AddLiquid` and returns the amount minus what the game says is left over.)</span>
- A stack is stolen when its owner number is above 1. `StolenStacks()` lists the stolen stacks the player carries. Each entry is an `ItemStackInfo` snapshot.
- There is no separate "stolen" setting, and giving a stack with an owner above 1 can count as theft. A [witness](../glossary.md#witness) (a person or camera that can see the player) may see the player receive a stack owned by another character. Then the game can record a theft, the same as when the player picks it up. <span class="tech">(`Give` with such an `OwnerId` can commit theft when a witness sees it, because the game's own add method runs.)</span>
- The same events still fire. [`PlayerInventory.ItemAdded`](inventory/player-inventory.md) and the item drop on the floor happen as they do when the game does it itself. <span class="tech">(The game's own slot and drop methods run, so their events fire.)</span>
- Your mod id is checked and used in the log, and it holds no claim. Two mods can call these methods at the same time without getting in each other's way.
- Nothing is changed while a save is loading or being written. The calls return `false` (or `0` for `AddLiquid`) and do nothing. <span class="tech">(They check `SaveData.Busy`.)</span>

### Limits

- You can't give money with these calls. Item numbers 0 and 1, the two currencies, are always refused. Use the [Economy](economy.md) page for money.
- A mod id that is empty, longer than 64 characters or has characters other than letters, digits, `.`, `_` or `-` throws an `ArgumentException`.
- You can't equip or unequip an item. The game's equip methods act on whatever is already in the player's right hand. They cannot be told which item to use. <span class="tech">(`PlayerHandItems.PrimaryEquip` and `PrimaryUnequip` take no item id. The hand is filled by `DropR` and `DropL`, whose names do not match what they do.)</span>
- You can't take liquid out of a container. The game's own method for it is not finished: it only logs an error. <span class="tech">(`ItemOperations.RemoveLiquid` is not implemented. It logs an error and returns `false`.)</span>
- You can't sell a stack. The game's sell method sells a single unit whatever the count. The version that takes a whole stack needs a game stack object that a mod cannot make. <span class="tech">(`Inventory.SellItems(int, ...)` sells one unit, and the stack overload needs a game `ItemStack`.)</span>

**Common problems**

- `Give` returns `false` and adds nothing: the inventory has no room for the full `count`, or a quality or liquid rule under Things to know is not met. It also fails for item numbers 0 and 1, the currencies (use the Economy page for money). The other causes: a save is loading or being written, there is no player or inventory yet, or the game fails inside the call. When room is the problem, use `GiveOrDrop`, which drops what does not fit at the player's feet.
- A call throws an `ArgumentException`: the mod id is empty, longer than 64 characters, or has characters other than letters, digits, `.`, `_` or `-`. Pass a mod id that meets those rules.

## Details

Target: Obenseuer 0.5.08 (buildid 25969064), Unity 6000.0.77f1 Mono, [BepInEx](../glossary.md#bepinex) 6 Unity.Mono ([be.788](https://builds.bepinex.dev/projects/bepinex_be/788/BepInEx-Unity.Mono-win-x64-6.0.0-be.788%2B5b766a3.zip), the tested build) or BepInEx 5 (5.4.23 or later).
Namespace: `ObenseuerCore`. The log name is `obenseuer.moddingframework.stacks`, and the file is `BepInEx\logs\obenseuer.moddingframework.stacks.log`. Source files: `src/Api/StackActions.cs`.

Everything not listed in this reference is internal and may change without notice.

### When you can call it

- Call everything from your mod's normal code, not from a background thread you started. The game only works safely on one thread, the main thread (see the [glossary](../glossary.md#main-thread)).
- It is safe to call at any time, even on the main menu. With no player or inventory yet, the calls return `false`, `0` or an empty list.
- Problems come back as a plain answer, not an error. The one exception is a bad mod id, which throws. If the game fails inside a call, you get `false`, `0` or an empty list. The first failure of each call goes to the log.

### If something fails to set up

The framework does not change any game method for this page. So it has no [feature group](../glossary.md#feature-group) (a named set of framework changes) that can turn off. It only calls game methods when you call it. If a game update changes one of those methods, the call fails with a plain answer and one error goes to the log.
