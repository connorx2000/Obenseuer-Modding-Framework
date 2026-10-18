---
title: Listing and breaking objects
parent: Breakables
grand_parent: Objects and devices
nav_order: 2
---

# Listing and breaking objects

A breakable is an object with a strength. `Breakables` lists them, looks one up by its id, breaks or repairs it, and reports when the game breaks or repairs one. This page lists every call, property and event, with the rules behind them.

This page uses `Transform` from Unity. The [glossary](../../glossary.md#types-that-come-from-unity) says what it is.
{: .unity-types}

Each breakable you get back is a `BreakableInfo`, a snapshot taken when you asked. The snapshot does not update by itself, so ask again for fresh values.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>Breakables</code> <span>list, break and repair breakables</span></p>

`All` and `Get` find breakables. `Break` and `Repair` change them. `Broken` and `Repaired` tell you when the game does the same.

<div class="api-pair" markdown="1">

```csharp
public static class Breakables
{
    public static List<BreakableInfo> All();
    public static BreakableInfo Get(string id);
    public static bool IsBroken(string id);
    public static bool Break(string id);
    public static bool Repair(string id);

    public static event Action<BreakableArgs> Broken;
    public static event Action<BreakableArgs> Repaired;

    public static bool IsAvailable { get; }
}
```

| Method | Returns | What it does |
|---|---|---|
| `All()` | `List<BreakableInfo>` | Every breakable in the scene the player is in. |
| `Get(id)` | `BreakableInfo` | The breakable with this id. `null` when none matches. |
| `IsBroken(id)` | `bool` | `true` when the breakable exists and is broken. |
| `Break(id)` | `bool` | Breaks it. `true` when it is broken afterwards. |
| `Repair(id)` | `bool` | Repairs it. `true` when it is whole afterwards. |

| Event | Gives you | When it fires |
|---|---|---|
| `Broken` | `BreakableArgs` | A breakable went from whole to broken. |
| `Repaired` | `BreakableArgs` | A broken breakable became whole again. |

| Property | Type | What it means |
|---|---|---|
| `IsAvailable` | `bool` | `false` when any part of the break, repair, lock or smash events could not be set up. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>BreakableInfo</code> <span>a snapshot of one breakable</span></p>

`All` and `Get` give you one of these for each breakable. Pass its `Id` to the other calls.

<div class="api-pair" markdown="1">

```csharp
public sealed class BreakableInfo
{
    public string Id { get; }
    public string Name { get; }
    public bool IsBroken { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `Id` | `string` | The id to pass to the other calls. The game's saved number is stable between sessions. |
| `Name` | `string` | The object's name in the scene. |
| `IsBroken` | `bool` | `true` when its strength was 0 or less when you asked. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>BreakableArgs</code> <span>what an event tells you</span></p>

`Broken`, `Repaired` and [`LockBroken`](locks.md) hand you this. `Id` is the breakable's id, or the lock's id for `LockBroken`.

<div class="api-pair" markdown="1">

```csharp
public sealed class BreakableArgs
{
    public string Id { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `Id` | `string` | The id of the breakable, or of the lock for `LockBroken`. |

</div>
</div>

## Things to know

- A breakable's id is the number the game saves it under, or a path through the [scene](../../glossary.md#scene) when it has none. Use the id with `Get`, `IsBroken`, `Break` and `Repair`. An id for an object with no saved number is a path such as `SceneName:Root[0]/Child[2]`. It changes if objects in the scene are reordered. <span class="tech">(`Id` is the object's `GUID` field when it has one, otherwise the GUID of the `SavableScript` on its object, otherwise a path counted from the scene root with sibling positions.)</span>
- `All()` and the id lookups only see objects that are switched on in the scene the player is in. A breakable that is switched off, or in another scene, is not listed. An id for it gives `null` or `false`. <span class="tech">(They search the active scene with `FindObjectsByType` and skip inactive objects.)</span>
- `Break(id)` breaks the object the same way a hit would, so its normal break effects play. The sound, the pieces that fall off and anything the game attaches to the break all happen. The hit has no push direction, so nothing is knocked sideways. <span class="tech">(It calls `Breakable.Break` with the object's remaining strength as damage, zero vectors for the direction and `DamageReceiver.DamageType.Any`. The `onBreak` relay runs as it does for a hit.)</span>
- `Broken` fires once, when a breakable goes from whole to broken. It does not fire for an object that was already broken. It does not fire while a save is loading. <span class="tech">(It runs after `Breakable.Break`, and only when `strength` is 0 or less and the object was not already broken. Calls made with `onLoadingGame` set are ignored.)</span>
- `Repaired` fires once, when a broken breakable becomes whole again. It does not fire while a save is loading. That includes an object whose repair timer ran out while the game was closed. <span class="tech">(It runs after `Breakable.Repair`. A repair that `Breakable.OnLoadingGame` restored from an expired timer is remembered and skipped.)</span>
- `IsAvailable` is `false` when any part of the break, repair, lock or smash events could not be set up. The calls that read or change things keep working. Only the event whose setup failed stops. See [If something fails to set up](details.md#if-something-fails-to-set-up).

## Limits

- `Break(id)` returns `false` when nothing was broken. Four cases cause that: the id matches nothing, the object is already broken, a save is loading or being written, or the object has no parent in the scene. <span class="tech">(The checks are an unknown id, `strength <= 0`, `SaveGate.Busy` and `transform.parent == null`.)</span>
- `Repair(id)` makes the object whole again but does not rebuild what the break removed. The game does the same when its own repair timer runs out. A repaired object can therefore look different from a new one. <span class="tech">(It calls `Breakable.Repair`, which restores `strength` only. It does not rebuild the model.)</span>
- `Repair(id)` returns `false` when nothing was repaired. Four cases cause that: the id matches nothing, the object is not broken, a save is loading or being written, or the object has no repair timer. An object has no repair timer when its repair time is below 0. <span class="tech">(The check is `repairAfterTime < 0`.)</span>
