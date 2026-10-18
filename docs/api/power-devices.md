---
title: Power devices
parent: Objects and devices
grand_parent: API reference
has_children: true
nav_order: 11
---

# Power devices

[`PowerDevices`](power-devices/listing-and-switching-devices.md) gives your mod one list of the things in the [scene](../glossary.md#scene) that make or use power, with a way to switch them. Use it when your mod reads fuel or power, or switches a device on or off.

## What it is

Many things in the game need power: lamps, fridges and other furniture, crafting stations such as generators and stoves, and batteries. For each device this API tells you:

- whether it is switched on,
- whether it really has power,
- how much fuel or charge it has.

It also switches a device on or off. Events tell you when a device gains or loses power, and when a fuel burner runs out of fuel.

It covers three kinds of power object:

- Furniture power devices, the lamps and machines the player places in a room.
- Crafting stations that have a power source, including generators and stoves that burn fuel.
- Batteries, which use up stored fuel items over time.

Blackouts and generator switches are on the [World](world.md) page ([`WorldPower`](world/worldpower.md)) and are not repeated here. The gas pumps are on the [Fuel](fuel.md) page.

## When to use it

- Switch a device on or off as part of a quest.
- Show how much fuel a generator or a stove has left.
- React when a device loses power, or when a generator runs out of fuel.
- Find every power source in a room.

## Quick example

This mod writes a [log](../guides/getting-started/logging.md) line when a furniture device or a battery gains or loses power. It also logs when a fuel burner runs out of fuel. Once a save is loaded, it also logs how many power sources the scene has.

```csharp
using ObenseuerCore;

[ModInfo("example.powerwatch", "Power Watch", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class PowerWatch : FrameworkMod
{
    protected override void OnLoad()
    {
        PowerDevices.PoweredChanged += args => Log.Info(args.Id + (args.IsPowered ? " now has power." : " lost power."));
        PowerDevices.FuelRanOut += args => Log.Info(args.Id + " ran out of fuel.");
        Events.WhenReady(() =>
        {
            int sources = 0;
            foreach (PowerDeviceInfo device in PowerDevices.All())
                if (device.Kind == PowerDeviceKind.Source) sources++;
            Log.Info("This scene has " + sources + " power sources.");
        });
    }
}
```

`PoweredChanged` and `FuelRanOut` are events: the framework calls your code each time they happen. [`Events.WhenReady`](core-helpers/events-whenready.md) waits until a save is loaded before it runs your code. See [Using events](../guides/getting-started/using-events.md).
