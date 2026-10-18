---
title: Interaction and locks
parent: Places and travel
grand_parent: API reference
has_children: true
nav_order: 5
---

# Interaction and locks

[`Interactions`](interaction/interactions.md) and [`Locks`](interaction/locks-and-doors.md) let your mod work with the objects the player uses, and with the locks on doors, chests and keypads. Use them to make your own props usable, to react to the player using things, or to control locks and lockpicking.

## What it is

When the player looks at a door, a switch or a chest, the game shows a prompt such as "Open (Use)". That prompt is an interaction. This API lets your mod:

- Give any object its own prompt.
- Get told when the player uses something.
- See what the player is looking at.

It also covers locks. Your mod can:

- Check whether a door, chest or keypad is locked.
- Lock or unlock it.
- Change how hard the lock is to pick.
- Start the lockpicking minigame.
- Get told when a lock changes, or when a pick succeeds or fails.

## When to use it

- Make a prop in your area usable, for example a button that runs your code. Block it with a red reason such as "Too hungry".
- React when the player uses anything in the game, or picks a lock.
- Lock or unlock doors and chests from your mod, or make them easier or harder to pick.
- Open a keypad, or change its code.

## Quick example

This mod writes a [log](../guides/getting-started/logging.md) line each time the player uses something. It writes another each time they pick a lock or fail to.

```csharp
using ObenseuerCore;

[ModInfo("example.usewatch", "Use Watch", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class UseWatch : FrameworkMod
{
    protected override void OnLoad()
    {
        Interactions.Used += use => Log.Info("The player used " + use.Name + " (" + use.Kind + ") from " + use.Distance + " metres");
        Locks.LockpickSucceeded += pick => Log.Info("The player picked the lock on " + pick.Name);
        Locks.LockpickFailed += pick => Log.Info("The player failed to pick the lock on " + pick.Name);
    }
}
```

`Used`, `LockpickSucceeded` and `LockpickFailed` are events: the framework calls your code each time they happen. See [Using events](../guides/getting-started/using-events.md) for more on events.
