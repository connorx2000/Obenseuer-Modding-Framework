---
title: Common tasks
parent: Player movement
grand_parent: Player life and skills
nav_order: 1
---

# Common tasks

Short snippets for the usual [player movement](../player-movement.md) jobs: holding the player, turning the camera, stopping the mouse look, crouching and changing the jump. The snippets pass `"example.speedy"` as the mod id, the [owner](../../glossary.md#owner) (the mod id that holds a change) of each lock and multiplier.

This page uses `Transform` from Unity. The [glossary](../../glossary.md#types-that-come-from-unity) says what each one is.
{: .unity-types}

Hold the player still with [`Lock`](holding-the-player-in-place.md) while your mod plays a scene, then let go. While the lock is held, the player cannot move or look around.

```csharp
if (PlayerMovement.Lock("example.speedy")) Log.Info("The player is held.");
PlayerMovement.Unlock("example.speedy");
```

Hold the player and turn the camera toward something with `LockLookAt`. The turn only happens in first person. `target` is the thing to look at, and `true` snaps the camera at once.

```csharp
void LookAtIt(Transform target)
{
    PlayerMovement.LockLookAt("example.speedy", target, true);
}
```

Stop only the mouse look with [`LockLook`](locking-only-the-mouse-look.md), and leave walking free.

```csharp
PlayerMovement.LockLook("example.speedy");
PlayerMovement.UnlockLook("example.speedy");
```

Make the player [crouch](crouching.md), then stand up again. `StandUp` returns `false` when there is a ceiling over the player, and they stay crouched.

```csharp
PlayerMovement.Crouch();
if (!PlayerMovement.StandUp()) Log.Info("No room to stand up here.");
```

Give the player a higher jump with [`SetJumpMultiplier`](speed-and-jump-multipliers.md), then take back every change your mod made.

```csharp
PlayerMovement.SetJumpMultiplier("example.speedy", 2f);
PlayerMovement.ClearMultipliers("example.speedy");
```
