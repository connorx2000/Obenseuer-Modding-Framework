---
title: Common tasks
parent: Traders
grand_parent: People
nav_order: 1
---

# Common tasks

Short snippets for the usual jobs with [`Traders`](reading-a-trader.md): top up a trader's money, buy and sell for the player, and remove your price factors. Each snippet needs a shop id, which comes from [`Shops.All()`](../economy/shops.md).

Give a trader at least 500 money to pay the player with.

```csharp
string shopId = "put a shop id from Shops.All() here";
TraderInfo trader = Traders.Get(shopId);
if (trader != null && trader.Money < 500f) Traders.SetMoney(Mod.Id, shopId, 500f);
```

Buy 2 apples (item id 10620) from a trader for the player. When `Buy` refuses, `refusal` says why.

```csharp
string shopId = "put a shop id from Shops.All() here";
TraderTradeRefusal refusal;
if (!Traders.Buy(Mod.Id, shopId, 10620, 2, out refusal))
    Log.Warn("The buy was refused: " + refusal);
```

Sell 1 apple from the player to a trader.

```csharp
string shopId = "put a shop id from Shops.All() here";
TraderTradeRefusal refusal;
if (!Traders.Sell(Mod.Id, shopId, 10620, 1, out refusal))
    Log.Warn("The sale was refused: " + refusal);
```

Take your price factors off a trader.

```csharp
string shopId = "put a shop id from Shops.All() here";
Traders.ClearPriceFactors(Mod.Id, shopId);
```
