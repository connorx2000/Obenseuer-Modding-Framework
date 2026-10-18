---
title: World item events
parent: Items and crafting
grand_parent: API reference
has_children: true
nav_order: 15
---

# World item events

[`WorldItems`](worlditems/events.md) covers the items that lie in the world. Use it when your mod reacts to pickups, drops and uses, or needs to find, remove or time out such items.

## What it is

A world item is something that lies in the world and can be picked up: a dropped apple, a tool on a shelf, a coin on the floor. This API gives your mod these jobs:

- Get told when the player picks up a world item, drops one, or uses an item they hold.
- Give each world item an [id](../glossary.md#object-id) (a piece of text that names it), so you can find it again.
- Remove a world item from the world.
- Change how long new items last before they spoil or vanish.

## When to use it

- React when the player picks up or drops a certain item.
- Count how often the player uses held items.
- Find a particular item lying in the [scene](../glossary.md#scene), or remove it.
- Make items disappear sooner or later than the game normally does.

## Quick example

This mod writes a [log](../guides/getting-started/logging.md) line each time the player picks up or drops an item.

```csharp
using ObenseuerCore;

[ModInfo("example.itemwatch", "Item Watch", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class ItemWatch : FrameworkMod
{
    protected override void OnLoad()
    {
        WorldItems.PickedUp += args => Log.Info("Picked up " + args.Amount + " of item " + args.ItemId + " at " + args.Position);
        WorldItems.Dropped += args => Log.Info("Dropped " + args.Amount + " of item " + args.ItemId + " at " + args.Position);
    }
}
```

`PickedUp` and `Dropped` are events: the framework calls your code each time they happen. [Using events](../guides/getting-started/using-events.md) explains events in more detail.
