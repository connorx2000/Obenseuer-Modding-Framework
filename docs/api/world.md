---
title: World
parent: Places and travel
grand_parent: API reference
has_children: true
nav_order: 18
---

# World

The world API is about the place the player is in and the conditions around them: weather, power, water, radiation, location and the clock. Use it when your mod needs to read or change these, or react when they change.

## What it is

It has six parts, one class each:

- [`WorldWeather`](world/worldweather.md) reads and changes the weather, including local weather zones.
- [`WorldPower`](world/worldpower.md) tells you about blackouts and generators. It also lets you control the player's wake-up after a blackout.
- [`WorldWater`](world/worldwater.md) reads the water supply and the water the player swims in.
- [`WorldRadiation`](world/worldradiation.md) reads how much radiation the player has taken in.
- [`WorldLocation`](world/worldlocation.md) tells you which [scene](../glossary.md#scene) (one loaded area of the game) the player is in, and which ones they have visited.
- [`WorldClock`](world/worldclock.md) reads the calendar and sets the time of day and the date.

## When to use it

- React when the weather changes, or set the weather for a scene.
- React when a blackout starts or a generator is switched on or off.
- Check whether the player is swimming or underwater, or give them more breath.
- Show the player's radiation dose, or the reading on their dosimeter.
- Find out which scene the player is in, or which ones they have been to.
- Read the date, or move the clock forward.

## Quick example

This mod writes a [log](../guides/getting-started/logging.md) line when the weather changes, a blackout starts or the player goes under water. Once a save is loaded, it logs the current weather, the date and the scene.

```csharp
using ObenseuerCore;

[ModInfo("example.worldwatch", "World Watch", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class WorldWatch : FrameworkMod
{
    protected override void OnLoad()
    {
        WorldWeather.WeatherChanged += args => Log.Info("Weather is now " + WorldWeather.NameOf(args.Index));
        WorldPower.BlackoutStarted += () => Log.Info("A blackout started.");
        WorldWater.PlayerSubmerged += () => Log.Info("The player went under water.");
        Events.WhenReady(() =>
        {
            WorldTime now = WorldClock.Now;
            Log.Info("Weather: " + WorldWeather.CurrentName + ", " + WorldWeather.Temperature + " degrees.");
            Log.Info("Date: " + now.Year + "-" + now.Month + "-" + now.CalendarDay + " " + now.Hour + ":" + now.Minute);
            Log.Info("Scene: " + WorldLocation.CurrentScene);
        });
    }
}
```

`WeatherChanged`, `BlackoutStarted` and `PlayerSubmerged` are events: the framework calls your code each time they happen. [`Events.WhenReady`](core-helpers/events-whenready.md) runs your code once a save is loaded. See [Using events](../guides/getting-started/using-events.md) for more on events and `WhenReady`.
