---
title: Economy
parent: Money and shops
grand_parent: API reference
has_children: true
nav_order: 4
---

# Economy

The Economy API covers money in the game: the wallet, shops, prices, banks, mail, player shops and legal services. Use it when your mod pays, charges or reacts to the player's spending.

## What it is

The player has two kinds of money: [OC](../glossary.md#oc) (Open Sewer Coins, the everyday coins) and [RM](../glossary.md#rm) (Real Money). A mod can do these jobs:

- Read and change the wallet.
- Watch shops and vending machines.
- Set prices.
- Work with bank accounts and loans.
- Send and read mail.
- Run the player's own shops.
- Queue legal services.

## When to use it

- Pay the player a reward in OC when a quest step is done.
- Charge the player a fee, and only continue if they can pay.
- Show a message or keep a log when the player buys something in a shop.
- Change what a trader charges, for example a discount.
- Send the player a parcel or a letter with an item or money in it.

## Quick example

This mod waits until a save is loaded, then writes the wallet to the [log](../guides/getting-started/logging.md). After that it logs every change to either balance, whatever caused it (a shop, the bank, a reward from another mod).

```csharp
using ObenseuerCore;

[ModInfo("example.walletwatch", "Wallet Watch", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class WalletWatch : FrameworkMod
{
    protected override void OnLoad()
    {
        Cash.MoneyChanged += OnMoneyChanged;
        Events.WhenReady(() => Log.Info("wallet: " + Cash.Oc + " OC, " + Cash.Rm + " RM"));
    }

    private void OnMoneyChanged(MoneyChangedArgs args)
    {
        Log.Info(args.Currency + " went from " + args.Before + " to " + args.After);
    }
}
```

[`Cash.Oc`](economy/money-cash.md) is the OC balance as a whole number. `Cash.Rm` is the RM balance. `MoneyChanged` is an event: the framework calls your code each time it happens, once for each currency whose balance changed. [`Events.WhenReady`](core-helpers/events-whenready.md) waits until a save is loaded before it runs your code. See [Using events](../guides/getting-started/using-events.md) for more on events and `WhenReady`.
