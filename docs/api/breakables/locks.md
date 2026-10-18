---
title: Locks
parent: Breakables
grand_parent: Objects and devices
nav_order: 3
---

# Locks

A lock can break when it is hit hard enough. `Breakables` lists the locks in the [scene](../../glossary.md#scene), breaks one by its id, and reports when a hit breaks one. This page lists those calls and the rules behind them.

A lock is not a breakable. It is a different kind of object, so [`Broken` and `Repaired`](listing-and-breaking-objects.md) never fire for it. Use `Locks()`, `BreakLock` and `LockBroken` for locks. There is no call to repair a lock.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>Breakables</code> <span>break locks</span></p>

`Locks` lists the locks in the scene. `BreakLock` breaks one by its id. `LockBroken` tells you when a hit breaks one.

<div class="api-pair" markdown="1">

```csharp
public static class Breakables
{
    public static List<BreakableLockInfo> Locks();
    public static bool BreakLock(string id);

    public static event Action<BreakableArgs> LockBroken;
}
```

| Method | Returns | What it does |
|---|---|---|
| `Locks()` | `List<BreakableLockInfo>` | Every lock in the scene the player is in. |
| `BreakLock(id)` | `bool` | Breaks a lock. `true` when it is broken afterwards. |

| Event | Gives you | When it fires |
|---|---|---|
| `LockBroken` | `BreakableArgs` | A hit took a lock's strength to 0 or less. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>BreakableLockInfo</code> <span>a snapshot of one lock</span></p>

`Locks` gives you one of these for each lock. Read `Strength` to see whether it is broken, because `Locked` does not change when a lock breaks.

<div class="api-pair" markdown="1">

```csharp
public sealed class BreakableLockInfo
{
    public string Id { get; }
    public bool Locked { get; }
    public int Strength { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `Id` | `string` | The id to pass to `BreakLock`. |
| `Locked` | `bool` | The game's lock state. Breaking a lock does not change it. |
| `Strength` | `int` | How strong the lock is. `0` or less once it is broken. |

</div>
</div>

## Things to know

- `BreakLock(id)` breaks the lock the way a hit would. <span class="tech">(It calls `BreakableLock.Break` with the lock's remaining strength as damage and `DamageReceiver.DamageType.Any`.)</span>
- Read `Strength` to see whether a lock is broken, not `Locked`. Breaking a lock does not turn `Locked` off, so a broken lock can still say `Locked` is `true`. The game disables a broken lock's collision and interaction, so the player cannot use it. <span class="tech">(`BreakableLock.Break` never sets `_Lock.Locked`. `UpdateModel` disables the lock's collider and interaction.)</span>
- `LockBroken` fires once, when a hit takes a lock's strength from above 0 to 0 or less. Its [`BreakableArgs.Id`](listing-and-breaking-objects.md) is the lock's id.

## Limits

- `Locks()` only lists locks that are switched on in the [scene](../../glossary.md#scene) the player is in. A lock that is switched off is skipped. <span class="tech">(It searches the active scene with `FindObjectsByType` and skips inactive objects. A lock's id is made the same way as a breakable's id.)</span>
- `BreakLock(id)` returns `false` when no lock has that id, when the lock is already broken, or when a save is loading or being written. It returns `true` when the lock is broken afterwards.
- `LockBroken` does not fire in three cases: a save is loading, a key or a lockpick opens the lock, or the lock does not accept the hit. <span class="tech">(A key or lockpick unlock goes through the game's `Unlocked`, which sets the strength to 0 without a break. A hit that the lock's allowed damage types refuse does not change its strength.)</span>
