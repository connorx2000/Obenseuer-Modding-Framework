---
title: Common tasks
parent: Card games
grand_parent: Money and shops
nav_order: 1
---

# Common tasks

Short snippets for the usual jobs with [`CardGames`](tables-and-games.md): list the card tables and find the running game, then set and clear payout factors.

List the card tables in the [scene](../../glossary.md#scene) you are in and say which have a game running. `Active()` gives the first table with a running game, or null when none.

```csharp
foreach (CardTableInfo table in CardGames.Tables())
    Log.Info(table.Id + ": " + table.Kind + (table.Active ? ", a game is running" : ", idle"));

CardTableInfo running = CardGames.Active();
if (running != null) Log.Info("A game is running at " + running.Id);
```

Make Shittypants pay half as much again and Crosseven pay half as much. A factor of 1 is the game's normal payout (see [Payout factors](payout-factors.md)). `Mod.Id` is your [mod id](../../glossary.md#mod-id). Set the factors again after every save load, and take them back with `Clear`.

```csharp
if (CardGames.PayoutFactorsAvailable)
{
    CardGames.SetPayoutFactor(Mod.Id, CardGameKind.ChinaHand, 1.5f);
    CardGames.SetPayoutFactor(Mod.Id, CardGameKind.Ristiseiska, 0.5f);
}

CardGames.Clear(Mod.Id);
```
