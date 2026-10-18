---
title: Common tasks
parent: Shop stock
grand_parent: Money and shops
nav_order: 1
---

# Common tasks

Short snippets for the usual [`ShopStock`](shop-keys.md) jobs: place a starting stack, sell in a vending machine, take stock back, and check that the feature works.

Give a trader a one-time starting stack. `Place` puts the items in once for each save. It returns `true` when at least one item was placed.

```csharp
if (ShopStock.Place("example.applestall", "some-trader-id", 10620, 3)) Log.Info("placed 3 apples");
```

Sell an item in a vending machine at a fixed price of 15 [OC](../../glossary.md#oc) (the game's everyday currency), with room for 8.

```csharp
ShopStock.AddVendingEntry("example.applestall", "Some Vending Machine", 10620, 8, 15f, MoneyCurrency.OC);
```

Take back what your mod added for one item in one shop.

```csharp
ShopStock.Remove("example.applestall", "Some Trader", 10620);
```

Check that the feature works before you rely on it. If `IsAvailable` is `false`, see [Details](details.md#if-something-fails-to-set-up).

```csharp
if (!ShopStock.IsAvailable) Log.Warn("shop stock is off, the stock will not be added");
```
