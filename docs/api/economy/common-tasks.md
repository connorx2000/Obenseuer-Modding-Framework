---
title: Common tasks
parent: Economy
grand_parent: Money and shops
nav_order: 1
---

# Common tasks

Short code for the money jobs mods do most often: paying, charging, shop purchases, prices, bank accounts, mail, legal services and player shops. Copy a snippet into your mod and change the values.

Pay the player with [`Cash.Give`](money-cash.md). It returns `false` when the money could not be added, for example while a save loads. This pays 100 [OC](../../glossary.md#oc) (the everyday currency):

```csharp
if (Cash.Give(MoneyCurrency.OC, 100)) Log.Info("paid 100 OC");
```

Charge the player. `TrySpend` takes the money only if the wallet covers it:

```csharp
if (Cash.TrySpend(MoneyCurrency.OC, 20)) Log.Info("charged 20 OC");
else Log.Info("the player cannot afford 20 OC");
```

Run code after every shop purchase:

```csharp
Shops.Bought += trade => Log.Info("bought " + trade.Count + " of item " + trade.ItemId + " for " + trade.Price + " " + trade.Currency + " at " + trade.Shop.Name);
```

Give the player a 10 percent discount at traders. The limits of `PriceQuery` are under [Shops](shops.md#shops).

```csharp
Shops.PriceQuery += args => { if (args.PlayerBuys) args.Price *= 0.9f; };
```

List the player's [bank accounts](banking.md):

```csharp
foreach (BankAccount account in Banking.All())
    if (account.Exists) Log.Info(account.BankName + ": " + account.Oc + " OC");
```

Send the player a parcel with an item, found with [`Items.Find`](../items/lookup.md). The game's [mail](mail.md) delay applies, so it arrives later:

```csharp
ItemDef item = Items.Find("Empty Glass Bottle");
if (item != null) Mail.SendItem("A small thank you", item.Id, 1, MailKind.Package);
```

React when mail arrives:

```csharp
Mail.Received += args => Log.Info("mail arrived: " + args.Name);
```

List the [legal services](legal-services.md) and what they cost:

```csharp
foreach (LegalActionInfo action in LegalServices.Actions())
    Log.Info(action.Name + " costs " + action.PriceOC + " OC, can trigger: " + action.CanTrigger);
```

List the [player's own shops](player-shops.md) and their tills:

```csharp
foreach (PlayerShopInfo shop in PlayerShops.All())
    Log.Info(shop.Id + " open: " + shop.Open + ", till: " + shop.UncollectedOc + " OC");
```
