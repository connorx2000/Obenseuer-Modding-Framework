---
title: Housing
parent: Places and travel
grand_parent: API reference
has_children: true
nav_order: 4
---

# Housing

`Housing` and `Building` cover the tenement's apartments, tenants and rent, and the furniture the player places. Use them to react to tenants, run evictions, or place and track furniture from your mod.

## What it is

The tenement is the building where the player lives. It has apartments, and tenants who move in, pay rent and can be evicted. The player can also place and take furniture in a room using build mode. This API has two halves:

- [`Housing`](housing/apartments-and-tenants.md) covers the apartments and the people in them. It tells you who lives where and what rent they pay. It reports when someone moves in or out, and it starts or stops an eviction.
- [`Building`](housing/building.md) covers the furniture the player places. It lists furniture, places a piece, moves one and takes one out. It also unlocks furniture blueprints and finds the [building area](../glossary.md#building-area) (a part of a building that holds placed furniture) the player is standing in.

Some related jobs have their own pages:

- Cleaning an apartment: [Cleaning](cleaning.md).
- Renovations (contractor upgrades): [Renovations](renovations.md).
- Tenant behaviour, such as starting a tenement event: [Tenants](tenants.md).

## When to use it

- Log or react when a tenant moves in or out, or when rent comes in.
- Start an eviction, or call it off, as part of a story.
- Place a piece of furniture, or tidy a room, from your mod.
- React when the player puts furniture down or takes it out.
- Re-attach your mod's behaviour to its own furniture after a save loads.

## Quick example

This mod writes a [log](../guides/getting-started/logging.md) line when a tenant moves in or out and when rent comes in. Once a save is loaded, it also logs how many apartments are vacant.

<div class="api-example" markdown="1">

```csharp
using ObenseuerCore;

[ModInfo("example.tenantwatch", "Tenant Watch", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class TenantWatch : FrameworkMod
{
    protected override void OnLoad()
    {
        Housing.TenantMovedIn += args => Log.Info(args.Tenant.Name + " moved into apartment " + args.ApartmentId);
        Housing.TenantMovedOut += args => Log.Info(args.Tenant.Name + " moved out of apartment " + args.ApartmentId);
        Housing.RentPaid += rent => Log.Info("Rent came in: " + rent.Oc + " OC and " + rent.Rm + " RM.");
        Events.WhenReady(() =>
        {
            int vacant = 0;
            foreach (ApartmentInfo apartment in Housing.AllApartments())
                if (apartment.Tenant == null) vacant++;
            Log.Info(vacant + " apartments are vacant.");
        });
    }
}
```

| Step | What it does |
|---|---|
| `Housing.TenantMovedIn` | Logs each move-in. |
| `Housing.TenantMovedOut` | Logs each move-out. |
| `Housing.RentPaid` | Logs each rent payment. |
| `Events.WhenReady` | Waits until a save is loaded, then counts the vacant apartments. |

</div>

`TenantMovedIn`, `TenantMovedOut` and `RentPaid` are events: the framework calls your code each time they happen. [`Events.WhenReady`](core-helpers/events-whenready.md) waits until a save is loaded before it runs your code. See [Using events](../guides/getting-started/using-events.md). OC is Open Sewer Coins and RM is Real Money (see the glossary entries [OC](../glossary.md#oc) and [RM](../glossary.md#rm)).
