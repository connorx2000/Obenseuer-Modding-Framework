---
title: Common tasks
parent: Renovations
grand_parent: Places and travel
nav_order: 1
---

# Common tasks

Short examples for reading, ordering and checking renovations with [`Renovations`](reading-renovations.md). Where an example uses `renovationId`, it is the `Id` of a renovation from `Renovations.All()`.

Read how far a renovation has got. The value runs from 0 to 1.

```csharp
float done = Renovations.Progress(renovationId);
Log.Info("Renovation is " + (int)(done * 100f) + " percent done.");
```

Order a renovation. This only works while the player has the contractor menu open. It takes the money and items the way the game does.

```csharp
if (!Renovations.Start(renovationId)) Log.Info("The renovation could not be ordered.");
```

Find the renovations that wait for the player's final payment.

```csharp
foreach (RenovationInfo info in Renovations.All())
{
    if (info.State == RenovationState.AwaitingFinalPayment) Log.Info(info.Name + " waits for " + info.FinalCost + " RM.");
}
```
