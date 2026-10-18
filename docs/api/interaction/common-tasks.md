---
title: Common tasks
parent: Interaction and locks
grand_parent: Places and travel
nav_order: 1
---

# Common tasks

Short snippets for the usual jobs with [`Interactions`](interactions.md) and [`Locks`](locks-and-doors.md): prompts, what the player looks at, locks, lockpicking and keypads. The snippets use game objects you already have (`treadmill`, `door` and `keypad`) and a `hungry` flag your mod keeps.

Give an object a prompt, give it a red reason that blocks it, and take the prompt off again. The second value is your [mod id](../../glossary.md#mod-id). The prompt reads `Treadmill (Run)`.

```csharp
Interactions.Add(treadmill, Mod.Id, "Treadmill", "Run", () => Log.Info("Running"));
Interactions.SetReason(treadmill, Mod.Id, () => hungry ? "Too hungry" : null);
Interactions.Remove(treadmill, Mod.Id);
```

See what the player is looking at. `Looking()` returns `null` when they are looking at nothing usable.

```csharp
InteractionArgs looking = Interactions.Looking();
if (looking != null) Log.Info("Looking at " + looking.Name);
```

Unlock a door the player has the key for, make it pickable, and make the lock a little easier to pick.

```csharp
if (Locks.IsLocked(door) && Locks.PlayerHasKey(door)) Locks.Unlock(door);
Locks.SetLockpickable(door, true);
Locks.SetDifficulty(door, 1f);
```

Start the lockpicking minigame on a locked door. Your code runs with `true` when the lock opens, and with `false` when the player fails or gives up.

```csharp
Locks.StartLockpick(door, 2f, ok => Log.Info(ok ? "picked" : "failed"));
```

Make a keypad count as solved, then read and change a keypad's code. The id is the keypad's save id. Read it from `InteractionArgs.Guid` while the player looks at the keypad.

```csharp
Locks.TriggerKeypad(keypad, true);
int[] code = Locks.KeypadCode("some-keypad-id");
if (code != null) Locks.SetKeypadCode("some-keypad-id", new[] { 1, 2, 3, 4 });
```
