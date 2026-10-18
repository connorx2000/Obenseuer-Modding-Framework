---
title: Fuel
parent: Money and shops
grand_parent: API reference
has_children: true
nav_order: 5
---

# Fuel

[`FuelStations`](fuel/gas-stations.md) and [`FuelTypes`](fuel/fuel-items.md) cover the gas stations in the scene and the items that burn as fuel. Use them to read fuel prices, react when the player buys fuel, or work out burn times.

## What it is

This API covers two parts of fuel:

- Gas stations: the pumps where the player fills up and pays. The API lists the stations in the [scene](../glossary.md#scene) you are in and gives each station's current price. It also tells your mod when the player pays for fuel.
- Fuel items: the items that burn as fuel in a generator or similar machine. For each one it gives how long it burns and what it leaves behind.

The minitrain's fuel tank is not on this page. `Trains.Minitrain` already reports its `Fuel` and `Capacity` (see [Trains](trains.md)).

## When to use it

- Show or log the fuel price at each gas station.
- React when the player buys fuel, for example to give a reward or keep a record of what was spent.
- Work out how long an item burns in a generator, or which items leave ash or another leftover.

## Quick example

This mod [logs](../guides/getting-started/logging.md) a line when the player pays for fuel. Once a save is loaded, it logs the price at each gas station and counts the items that can be used as fuel.

```csharp
using ObenseuerCore;

[ModInfo("example.fuelwatch", "Fuel Watch", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class FuelWatch : FrameworkMod
{
    protected override void OnLoad()
    {
        FuelStations.FuelBought += args => Log.Info("Paid " + args.Cost + " " + args.Currency + " for " + args.Amount + " fuel at " + args.StationId);
        Events.WhenReady(() =>
        {
            foreach (FuelStationInfo station in FuelStations.All())
                Log.Info("Station " + station.Id + " charges " + station.Price + " " + station.Currency + " a unit.");
            Log.Info(FuelTypes.All().Count + " items can be burned as fuel.");
        });
    }
}
```

`FuelBought` is an event: the framework calls your code each time it happens. [`Events.WhenReady`](core-helpers/events-whenready.md) runs your code once a save is loaded. See [Using events](../guides/getting-started/using-events.md) for more on events and `WhenReady`.
