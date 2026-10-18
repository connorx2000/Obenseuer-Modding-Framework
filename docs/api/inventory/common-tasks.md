---
title: Common tasks
parent: Inventory
grand_parent: Items and crafting
nav_order: 1
---

# Common tasks

Short snippets for the usual inventory jobs: pay with items, list stacks, read a hand, look into a container and react when one opens.

Each snippet runs after a save has loaded, for example inside [`Events.WhenReady(() => { ... })`](../core-helpers/events-whenready.md). The item id 10001 is the Empty Glass Bottle.

Take items only if the player has all of them. [`PlayerInventory.Take`](player-inventory.md) already refuses when the whole amount is not held. The `Has` check is only there so you can show your own message first.

```csharp
int id = 10001;
if (PlayerInventory.Has(id, 3) && PlayerInventory.Take(id, 3)) Log.Info("The player paid 3 bottles.");
```

List every stack the player carries, with its quality. Each stack is an [`ItemStackInfo`](item-stacks.md).

```csharp
foreach (ItemStackInfo stack in PlayerInventory.Items())
    Log.Info(stack.ItemId + " x" + stack.Count + " quality " + stack.Quality);
```

See what is in the right hand. `-1` means the hand is empty.

```csharp
int right = PlayerInventory.GetHeldItemId(false);
```

Find a container by its save id and list what is in it.

```csharp
ContainerInfo chest = Containers.FindByGuid("some-guid");
if (chest != null)
    foreach (ItemStackInfo stack in Containers.GetItems(chest))
        Log.Info(stack.ItemId + " x" + stack.Count);
```

Move one item from a container to the player. [`Containers.Take`](containers.md) destroys the items, so give them to the player first.

```csharp
if (chest != null && PlayerInventory.Give(10001, 1)) Containers.Take(chest, 10001, 1);
```

React when the player opens or closes any container.

```csharp
Containers.Opened += c => Log.Info("opened " + c.Name);
Containers.Closed += c => Log.Info("closed " + c.Name);
```
