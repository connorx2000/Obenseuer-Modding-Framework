---
title: Try it out
parent: Inventory
grand_parent: Items and crafting
nav_order: 6
---

# Try it out

A short test of the inventory calls. It counts, gives and takes an item and [logs](../../guides/getting-started/logging.md) each result.

```csharp
int id = 10001;
Log.Info("count " + PlayerInventory.Count(id) + " has1 " + PlayerInventory.Has(id, 1) + " free " + PlayerInventory.FreeSlots + "/" + PlayerInventory.Capacity);
Log.Info("give " + PlayerInventory.Give(id, 2) + " count now " + PlayerInventory.Count(id));
Log.Info("take " + PlayerInventory.Take(id, 2) + " count now " + PlayerInventory.Count(id));
Log.Info("stacks " + PlayerInventory.Items().Count + " held right " + PlayerInventory.GetHeldItemId(false));
Log.Info("containers " + Containers.All().Count);
```

1. Put the snippet in a test mod, where it runs once a save is loaded.
2. Replace `id` with a real item id. `0` and `1` are money, not items.
3. Load a save and read the log lines.

Expected: `give` is `true` and the count goes up by 2, then `take` is `true` and the count goes back. The `ItemAdded` and `ItemRemoved` handlers log each event.

With `give` and `take` both `true`, the test has passed. [Details](details.md) covers when to call the API and what to check.
