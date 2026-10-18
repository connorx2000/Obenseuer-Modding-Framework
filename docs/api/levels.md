---
title: Levels
parent: Places and travel
grand_parent: API reference
has_children: true
nav_order: 8
---

# Levels

[`Levels`](levels/level-changers.md) lists the doors and areas that take the player to another level, and moves the player between levels. Use it to lock a route until a quest step, to send the player somewhere, or to react to a level change.

## What it is

The game world is split into levels. The player moves from one level to another through [level changers](../glossary.md#level-changer), which come in three kinds:

- doors with a lock,
- other use points,
- invisible areas that take the player to the next level when they walk in.

This API gives your mod these jobs:

- List the level changers in the [scene](../glossary.md#scene) you are in.
- See where each one leads: the level, and the [entrypoint](../glossary.md#entrypoint) (a named spawn point) where the player arrives.
- Lock and unlock doors.
- Use a changer for the player.
- Move the player to any level by name.
- Get told when the game is about to change level.

## When to use it

- Lock a door to another level until a quest step, and unlock it afterwards.
- Take the player to another level from your own code, for example from a menu.
- React when the player is about to leave the level, for example to [save your mod's own data](../guides/getting-started/save-data.md).

## Quick example

This mod writes a [log](../guides/getting-started/logging.md) line with the destination each time the game changes level. Once a save is loaded, it also logs every level changer in the [scene](../glossary.md#scene), with its destination and whether it is locked.

```csharp
using ObenseuerCore;

[ModInfo("example.levelwatch", "Level Watch", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class LevelWatch : FrameworkMod
{
    protected override void OnLoad()
    {
        Levels.Changing += args => Log.Info("Changing level to " + args.OtherLevel + " at " + args.OtherEntrypoint);
        Events.WhenReady(() =>
        {
            foreach (LevelDoorInfo door in Levels.All())
                Log.Info(door.Id + " leads to " + door.OtherLevel + ", locked=" + door.Locked);
        });
    }
}
```

`Changing` is an event: the framework calls your code each time it happens. [`Events.WhenReady`](core-helpers/events-whenready.md) runs your code once the game is ready. See [Using events](../guides/getting-started/using-events.md) for more on both.
