---
title: Renovations
parent: Places and travel
grand_parent: API reference
has_children: true
nav_order: 9
---

# Renovations

[`Renovations`](renovations/reading-renovations.md) lets your mod work with the upgrades contractors build in the player's building. Use it to show prices and progress, order an upgrade, or react when one is finished.

## What it is

The player's building can be upgraded by contractors: new electricity, water, heating, walls, a kitchen and more. Some upgrades are for the whole building and some are for one apartment. This API calls each upgrade a renovation. It gives your mod these jobs:

- List the renovations.
- See what state each one is in.
- Read the price and the build progress.
- Order one.
- Get told when one is finished.

Renovation cost and build-time tweaks are in `TweaksRenovations`. The rules for the building and its tenants are on the [Housing page](housing.md).

## When to use it

- Show which renovations the player can order and what they cost.
- Show how far an ordered renovation has got.
- Run code when a renovation is finished.
- Order a renovation for the player from your mod, while the contractor menu is open.

## Quick example

This mod writes a [log](../guides/getting-started/logging.md) line when a renovation is finished. Once a save is loaded, it also logs the renovations the player can order now, with their price.

```csharp
using ObenseuerCore;

[ModInfo("example.renowatch", "Renovation Watch", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class RenoWatch : FrameworkMod
{
    protected override void OnLoad()
    {
        Renovations.Finished += info => Log.Info(info.Name + " is finished (apartment " + info.ApartmentId + ").");
        Events.WhenReady(() =>
        {
            foreach (RenovationInfo info in Renovations.All())
            {
                if (info.State == RenovationState.Available) Log.Info(info.Name + " can be ordered for " + info.Cost + " OC.");
            }
        });
    }
}
```

`Finished` is an event: the framework calls your code each time it happens. [`Events.WhenReady`](core-helpers/events-whenready.md) waits until a save is loaded before it runs your code. OC is Open Sewer Coins, the game's everyday money. RM is Real Money, its second currency. See the glossary entries [OC](../glossary.md#oc) and [RM](../glossary.md#rm). See [Using events](../guides/getting-started/using-events.md) for more on events and `WhenReady`.
