---
title: Common tasks
parent: Casino
grand_parent: Money and shops
nav_order: 1
---

# Common tasks

Short snippets for the usual jobs with [`Casino`](events.md): list the machines, set and clear payout factors, react to a double or a scratchcard, and read the last win.

List the machines in the [scene](../../glossary.md#scene) and the stake each one is set to. Each machine is a [`CasinoMachineInfo`](the-machines.md).

```csharp
foreach (CasinoMachineInfo machine in Casino.Machines())
    Log.Info(machine.Kind + " " + machine.Id + ", stake " + machine.Stake);
```

Make every slot machine pay double. `Info.Id` is your [mod id](../../glossary.md#mod-id). [Payout factors](payout-factors.md) are forgotten when a scene loads, so set it again each time a scene loads, from [`Events.SceneLoaded`](../../api-core/events.md).

```csharp
Events.SceneLoaded += scene => Casino.SetPayoutFactor(Info.Id, CasinoMachineKind.Slot, 2f);
```

Take your payout factors back, so the machines pay what the game normally pays.

```csharp
Casino.Clear(Info.Id);
```

React when the player doubles a slot win.

```csharp
Casino.Doubled += args => Log.Info(args.Won ? "The double paid off." : "The double failed.");
```

React when a scratchcard wins.

```csharp
Casino.Scratched += args =>
{
    if (args.Won) Log.Info(args.CardTitle + " won " + args.PrizeOc + " OC worth.");
};
```

Read the last win, and check whether the events and payout factors work. If one is `false`, see [Details](details.md#if-something-fails-to-set-up).

```csharp
Log.Info("The last win was " + Casino.LastPayout + " OC.");

if (!Casino.IsAvailable) Log.Warn("Casino events are off.");
if (!Casino.PayoutFactorsAvailable) Log.Warn("Payout factors are off.");
```
