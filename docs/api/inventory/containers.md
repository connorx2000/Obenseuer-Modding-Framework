---
title: Containers
parent: Inventory
grand_parent: Items and crafting
nav_order: 5
---

# Containers

A container is a chest, locker, shelf or any other storage the player can open. `Containers` finds them, looks inside, moves items in and out, and tells you when the player opens or closes one. Each container is a `ContainerInfo`, a snapshot taken when you asked.

This page uses `Vector3` from Unity. The [glossary](../../glossary.md#types-that-come-from-unity) says what each one is.
{: .unity-types}

<div class="api-type" markdown="1">
<p class="api-type-head"><code>Containers</code> <span>finding, reading and changing containers</span></p>

The class you start from. `All()` and `FindByGuid` find containers, and `GetItems` lists what is inside. `Give` and `Take` move items, and `Opened` and `Closed` tell you when the player opens or closes one.

<div class="api-pair" markdown="1">

```csharp
public static class Containers
{
    public static List<ContainerInfo> All();
    public static ContainerInfo FindByGuid(string guid);
    public static string GuidOf(ContainerInfo container);
    public static List<ItemStackInfo> GetItems(ContainerInfo container);
    public static bool Give(ContainerInfo container, int itemId, int count);
    public static bool Take(ContainerInfo container, int itemId, int count);
    public static ItemStackInfo TakeStack(ContainerInfo container, int slot, int count);

    public static event Action<ContainerInfo> Opened;
    public static event Action<ContainerInfo> Closed;
}
```

| Method | Returns | What it does |
|---|---|---|
| `All()` | `List<ContainerInfo>` | Every container in the [scene](../../glossary.md#scene) you are in. |
| `FindByGuid(guid)` | `ContainerInfo` | The container with this save id. `null` when no container has it. |
| `GuidOf(container)` | `string` | The container's save id. `null` for a `null` container. |
| `GetItems(container)` | `List<ItemStackInfo>` | The stacks in the container, as [`ItemStackInfo`](item-stacks.md), without the empty slots. |
| `Give(container, itemId, count)` | `bool` | Puts items into the container. `true` when all of them went in. |
| `Take(container, itemId, count)` | `bool` | Takes items out and destroys them. `true` when all of them were taken. |
| `TakeStack(container, slot, count)` | `ItemStackInfo` | Takes items from one slot and destroys them. `null` when it could not. |

| Event | Gives you | When it fires |
|---|---|---|
| `Opened` | `ContainerInfo` | The player opened a container. |
| `Closed` | `ContainerInfo` | The player closed a container. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>ContainerInfo</code> <span>a snapshot of one container</span></p>

`All()`, `FindByGuid` and the `Opened` and `Closed` events give you this. It is a snapshot taken when you asked. `Give` and `Take` check the lock again each time. <span class="tech">(It is built from the game's `Storage` object.)</span>

<div class="api-pair" markdown="1">

```csharp
public sealed class ContainerInfo
{
    public string Guid { get; }
    public int InstanceId { get; }
    public string Name { get; }
    public Vector3 Position { get; }
    public int Apartment { get; }
    public int OwnerId { get; }
    public bool NpcOwned { get; }
    public bool Locked { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `Guid` | `string` | The container's save id. `""` when it has none. |
| `InstanceId` | `int` | The game's number for this object in this session. |
| `Name` | `string` | The container's title, with " (Empty)" added when it is empty. |
| `Position` | `Vector3` | Where it is, in world coordinates. |
| `Apartment` | `int` | The number of the apartment it is in. `-1` when it is not in an apartment. |
| `OwnerId` | `int` | The id of the NPC who owns it. `-1` when nobody does. |
| `NpcOwned` | `bool` | `true` when an NPC owns it, so taking from it counts as stealing. |
| `Locked` | `bool` | `true` when it is locked. |

</div>
</div>

## Things to know

- `All()` lists every container in the [scene](../../glossary.md#scene) you are in. It is slow, because it searches the whole scene each time. Call it once and keep the result; do not call it every frame. <span class="tech">(It finds every active `Storage` in the active scene with `FindObjectsByType`.)</span>
- `All()` includes shops, vending machines, prisons and the storage of the equipped backpack, so check `Name`.
- A `ContainerInfo` does not update by itself: `Locked` and `Name` stay as they were. `Give` and `Take` check the lock again each time.
- `GetItems` lists the stacks in a container, without the empty slots, and sets each stack's `Slot`. Pass that `Slot` to `TakeStack` or [`PlayerShops.Sell`](../economy/player-shops.md) to take that exact stack. <span class="tech">(It reads the game's `storageSlots`, and `Slot` is the index there.)</span>
- `Take` removes items from a container and destroys them. Nothing is added to the player's inventory.
  - A mod that wants a transfer must call [`PlayerInventory.Give`](player-inventory.md) with the same amount itself, and should do that first.
  - The game would call taking from an NPC's container stealing. The framework refuses instead of getting around that.
- `TakeStack(container, slot, count)` takes `count` items from the stack in one slot. It gives you that stack's details as an `ItemStackInfo`, with who owns it and its extra data. As with `Take`, the items are destroyed, not moved to the player.
- `FindByGuid(guid)` finds a container by its save id, or gives `null`. It gives `null` for an empty or `null` id, for an unknown id, and for a container that was destroyed.
- `GuidOf(container)` gives the container's save id, or `null` for `null`. An empty text means the container has no save id.
- A mod that tracks containers should ignore the ones with an empty `Guid`. They have no save identity, so `FindByGuid` cannot return them, and another player's game cannot match them.
- `Opened` fires once each time a container is opened, and `Closed` once each time it is closed. Each gives you a `ContainerInfo` built the same way as in `All()`.
- `Closed` also fires when the container is switched off or destroyed while open, for example when the scene changes. Closing something twice raises nothing.
- Opening a container while another is open raises `Closed` for the old one first. Use `container.Guid` to point at a container on another player's game. <span class="tech">(`Opened` is raised after `Storage.EnterExternally`, and `Closed` after `Storage.ExitExternally` or `Storage.OnDisable`. The framework tracks the one container it has open.)</span>
- The two events cover more than chests. They also cover these, because the game opens them all the same way:
  - backpack, cooler bag and rucksack storage
  - crafting input and output storage
  - trade and vending storage
- Bottles and tanks (liquid storage) have their own way in and out, and raise nothing.
- Every way a container closes raises `Closed`. <span class="tech">(The menu close is the game's `HideGameMenu`.)</span>
  - The menu closing: the pause key while a menu is open, or the menu's own close buttons.
  - The container closing itself: crafting, vending machines, trades and backpacks close their own storage.
  - The player cannot walk away from an open container menu, because the game stops movement while it is open.
  - A container opened without a menu is closed by whatever opened it, and that is covered too.
- Your handlers run one at a time, right when the game opens or closes the container. If a handler throws an error, the framework logs it and still runs the other handlers. The framework does not hold these events back while a save is loading either.

## Limits

- `Give` puts items into a container. It refuses when the container is locked, or when no slot can hold the whole amount. Giving to a container that an NPC owns is allowed, as it is in the game. Money ids `0` and `1` are refused. <span class="tech">(It checks the fit against every slot, then calls the game's `Storage.AddItem`, which places into slot 0 and its siblings. In an unusual slot layout that call could still drop the overflow in the world.)</span>
- `Take` refuses when the container is locked, or does not hold the whole amount. It also refuses when an NPC owns it and the player may not take from it. <span class="tech">(It calls the game's `Storage.RemoveItem` and refreshes the container's slots. The refusal is `PermissionToTake` being false. The game's theft path is not a public method the framework can call.)</span>
- `TakeStack` returns `null` in these cases:
  - the container is locked
  - an NPC owns it and the player may not take from it
  - the slot is out of range or empty
  - the stack holds fewer than `count`
  - a save is loading or being written
  - `slot` is `-1`
  - the container's slots were rebuilt since you listed them, so list again before taking
- When the game's quick lookup has no such container, `FindByGuid` searches the whole scene once. So every miss costs one full search: do not call it every frame. <span class="tech">(It first uses the game's GUID registry, the same lookup `SaveData.CaptureByGuid` and `SaveData.ApplyByGuid` use. The game fills that registry only for objects it matched to an entry in the loaded save. The fallback is `SaveController.GetObjectByGUID(guid, typeof(Storage))`, which calls `FindObjectsOfType` once.)</span>
- A container can count as open while no menu shows. The events mean that the storage was entered or left, not that a container menu is visible. <span class="tech">(`Storage.EnterExternally` runs its side effects, including setting `Storage.active` and raising `Opened`, even when no menu shows. The menu is shown only when the `isInteractable | forceOpen` check in `Storage` passes.)</span>
- Opening the backpack while a container is open closes that container and reopens it later.
  - With container A open, opening the backpack raises `Closed(A)` and then `Opened(backpack)`.
  - Closing the backpack raises `Closed(backpack)` and then `Opened(A)`, because the game opens the last used container again.
  - A mod that locks a container while it is open sees A close and reopen on every backpack toggle. <span class="tech">(`BackpackStorage.CloseBackpack` re-interacts with the last used container.)</span>

**Common problems**

- `Take` returns `true`, but the player gets nothing: `Take` destroys the items. Call `PlayerInventory.Give` with the same amount first.
- `TakeStack` returns `null` for a stack that `GetItems` listed: one of the cases in the `TakeStack` list under Limits applies. When the container's slots were rebuilt, list again before taking.
- `FindByGuid` returns `null`: the id is empty, `null` or unknown, or the container was destroyed. A container with an empty `Guid` has no save identity, so a mod that tracks containers should ignore those.
