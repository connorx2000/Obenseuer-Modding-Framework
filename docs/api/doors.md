---
title: Doors
parent: Places and travel
grand_parent: API reference
has_children: true
nav_order: 1
---

# Doors

[`Doors`](doors/finding-doors.md) lets your mod find and open or close the doors in the current [scene](../glossary.md#scene) (one loaded area of the game). It also changes opening hours and tells you when a door finishes moving. Use it when a quest or a feature needs control over a door.

## What it is

The doors in the game swing or slide open and shut. Some of them follow opening hours, like a shop door that is locked at night. This API does these jobs:

- List the doors in the [scene](../glossary.md#scene) you are in.
- [Open or close](doors/opening-and-closing.md) a door.
- Read or change a door's [opening hours](doors/opening-hours.md).
- Tell you when a door has [finished opening or closing](doors/door-events.md).

## When to use it

- Open a door for the player, or close one behind them.
- Check whether a door is open before your mod acts.
- Change the hours a shop door is open.
- Run code when a door has finished opening or closing.
- Find where an NPC stands while it waits for a door.

## Quick example

This mod writes a [log](../guides/getting-started/logging.md) line each time a door has fully opened or fully closed. Once a save is loaded, it also logs how many doors the scene has.

```csharp
using ObenseuerCore;

[ModInfo("example.doorwatch", "Door Watch", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class DoorWatch : FrameworkMod
{
    protected override void OnLoad()
    {
        Doors.Opened += args => Log.Info("Door opened: " + Doors.Get(args.Id)?.Name);
        Doors.Closed += args => Log.Info("Door closed: " + Doors.Get(args.Id)?.Name);
        Events.WhenReady(() => Log.Info("This scene has " + Doors.All().Count + " doors."));
    }
}
```

`Opened` and `Closed` are events: the framework calls your code each time the thing happens. `Doors.Get(id)` gives the door's details, or `null` when the id is not known. [`Events.WhenReady`](core-helpers/events-whenready.md) runs your code once a save is loaded. See [Using events](../guides/getting-started/using-events.md) for more on events and `WhenReady`.
