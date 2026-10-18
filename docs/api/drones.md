---
title: Drones
parent: Objects and devices
grand_parent: API reference
has_children: true
nav_order: 5
---

# Drones

[`Drones`](drones/reading-and-controlling-drones.md) lets your mod read and steer the drones in the current [scene](../glossary.md#scene) (one loaded area of the game). Use it to start, stop or redirect a drone, or to react when one changes state or passes a waypoint.

## What it is

Drones are machines in the game that follow a route. The route is a chain of [air nodes](drones/air-nodes.md), which are waypoints the drone moves toward one after another. This API does these jobs:

- List the drones and air nodes in the [scene](../glossary.md#scene) you are in.
- Switch a drone on or off, send it to a node, and change its top speed.
- Tell you when a drone changes state or [passes a node](drones/node-passes.md).

## When to use it

- Start or stop a drone from your own code, for example when the player finishes a quest step.
- Send a drone to a different air node, then start it.
- Slow a drone down or speed it up.
- Log or react when a drone switches on, switches off, or passes a node.

## Quick example

This mod writes a [log](../guides/getting-started/logging.md) line whenever a drone is switched on or off, and another whenever a drone passes a node. When the game is ready, it also logs how many drones are in the scene.

```csharp
using ObenseuerCore;

[ModInfo("example.dronewatch", "Drone Watch", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class DroneWatch : FrameworkMod
{
    protected override void OnLoad()
    {
        Drones.Changed += args => Log.Info("drone " + args.Id + " is now " + (args.Active ? "on" : "off"));
        Drones.NodePassed += args => Log.Info("drone " + args.DroneId + " passed node " + args.NodeId);
        Events.WhenReady(() => Log.Info(Drones.All().Count + " drones in this scene"));
    }
}
```

`Changed` and `NodePassed` are events: the framework calls your code each time they happen. `DroneId` is null when the framework cannot tell which drone passed the node. A real mod should check it before using it. [`Events.WhenReady`](core-helpers/events-whenready.md) waits until the game is ready before it runs your code. See [Using events](../guides/getting-started/using-events.md) for more on events and `WhenReady`.
