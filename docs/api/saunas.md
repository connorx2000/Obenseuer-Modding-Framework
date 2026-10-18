---
title: Saunas
parent: Objects and devices
grand_parent: API reference
nav_order: 12
---

# Saunas

`Saunas` reads the saunas in the [scene](../glossary.md#scene) you are in, switches them on or off, and tells you when one is switched. Use it when a quest step or a screen of yours depends on a sauna.

## What it is

A sauna is a heated room in the world that the player can switch on and use. This API gives your mod these jobs:

- List the saunas in the scene you are in.
- Read whether each one is on, how hot it is and how much steam (called löyly) it has.
- Switch a sauna on or off.
- Get told when a sauna is switched on or off.

It does not cover the player's own sauna stat (see `SaunaHeat` on the [core API page](../api-core.md)). It does not cover the sauna contest (see the [contests page](contests.md)) or the sauna's cooling object either.

## When to use it

- Switch a sauna on when the player starts a quest step, or off when it ends.
- Show the temperature of a sauna on your own screen.
- Run your own code when a sauna is switched on or off, by anyone.

## Quick example

This mod writes a [log](../guides/getting-started/logging.md) line each time a sauna is switched on or off. Once a save is loaded, it also logs every sauna in the scene.

```csharp
using ObenseuerCore;

[ModInfo("example.saunawatch", "Sauna Watch", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class SaunaWatch : FrameworkMod
{
    protected override void OnLoad()
    {
        Saunas.TurnedOn += args => Log.Info("Sauna " + args.Id + " is on at " + args.Temperature + " degrees.");
        Saunas.TurnedOff += args => Log.Info("Sauna " + args.Id + " is off.");
        Events.WhenReady(() =>
        {
            foreach (SaunaInfo sauna in Saunas.All())
                Log.Info("Sauna " + sauna.Id + ": on = " + sauna.On + ", " + sauna.Temperature + " degrees.");
        });
    }
}
```

`TurnedOn` and `TurnedOff` are events: the framework calls your code each time they happen. [`WhenReady`](core-helpers/events-whenready.md) waits until a save is loaded before it runs your code. See [Using events](../guides/getting-started/using-events.md) for more on events and `WhenReady`.

## Common tasks

Switch a sauna on.

```csharp
if (Saunas.SetOn("some-id", true)) Log.Info("The sauna is on.");
```

Read how much steam a sauna has.

```csharp
SaunaInfo sauna = Saunas.Get("some-id");
if (sauna != null) Log.Info("Steam: " + sauna.Loyly);
```

## Reference

Each sauna is a `SaunaInfo`, a snapshot of its state at the moment you asked. The snapshot does not update by itself, so ask again for fresh values. The two events give you a `SaunaChangedArgs`, which holds the state right after the change.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>Saunas</code> <span>list, switch and watch saunas</span></p>

Start here. `All()` and `Get(id)` read saunas, and `SetOn` switches one on or off. `TurnedOn` and `TurnedOff` tell you when a switch happens.

<div class="api-pair" markdown="1">

```csharp
public static class Saunas
{
    public static List<SaunaInfo> All();
    public static SaunaInfo Get(string id);
    public static bool SetOn(string id, bool on);
    public static bool IsAvailable { get; }
    public static event Action<SaunaChangedArgs> TurnedOn;
    public static event Action<SaunaChangedArgs> TurnedOff;
}
```

| Method | Returns | What it does |
|---|---|---|
| `All()` | `List<SaunaInfo>` | Every sauna that is switched on in the scene you are in. |
| `Get(id)` | `SaunaInfo` | The sauna with this id. `null` when none has it. |
| `SetOn(id, on)` | `bool` | Switches a sauna on or off. `true` when it ends up as you asked. |

| Property | Type | What it means |
|---|---|---|
| `IsAvailable` | `bool` | `false` when the on and off events could not be set up. |

| Event | Gives you | When it fires |
|---|---|---|
| `TurnedOn` | `SaunaChangedArgs` | A sauna has just been switched on. |
| `TurnedOff` | `SaunaChangedArgs` | A sauna has just been switched off. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>SaunaInfo</code> <span>a snapshot of one sauna</span></p>

`All()` and `Get(id)` give you one of these. It is a copy of the sauna at the moment you asked, so ask again for fresh values. <span class="tech">(It is built from the game's `Sauna` object, and `Loyly` is its `löyly` field.)</span>

<div class="api-pair" markdown="1">

```csharp
public sealed class SaunaInfo
{
    public string Id { get; }
    public bool On { get; }
    public float Temperature { get; }
    public float Loyly { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `Id` | `string` | The sauna's id: its save id, or its place in the scene when it has none. |
| `On` | `bool` | `true` when the sauna is on. |
| `Temperature` | `float` | How hot it is, in degrees Celsius. |
| `Loyly` | `float` | The steam from water thrown on the stones. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>SaunaChangedArgs</code> <span>what you get when a sauna is switched on or off</span></p>

The `TurnedOn` and `TurnedOff` events give you one of these. It holds the sauna's state after the switch, so `On` is the state it has now.

<div class="api-pair" markdown="1">

```csharp
public sealed class SaunaChangedArgs
{
    public string Id { get; }
    public bool On { get; }
    public float Temperature { get; }
    public float Loyly { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `Id` | `string` | The sauna's id, the same as `SaunaInfo.Id`. |
| `On` | `bool` | Whether the sauna is on now, after the change. |
| `Temperature` | `float` | How hot it is, in degrees Celsius. |
| `Loyly` | `float` | The steam. |

</div>
</div>

### Things to know

- `All()` lists every sauna that is switched on in the scene you are in. `Get(id)` gives one sauna. An unknown or empty id also gives `null`.
- `SetOn(id, on)` does what the game does when the player or a script switches the sauna. Switching a sauna on starts its timer. For a passive sauna it also sets the steam to 1. <span class="tech">(It calls the game's own `Sauna.TurnOn` or `Sauna.TurnOff`, and the save check is `SaveData.Busy`. `Loyly` is the game's `Sauna.löyly` field.)</span>
- `TurnedOn` fires after a sauna goes from off to on. `TurnedOff` fires after it goes from on to off. Switching on a sauna that is already on fires nothing.
- The events fire no matter who switched the sauna, not only on your calls to `SetOn`. The `SaunaChangedArgs` holds the sauna's state after the change. <span class="tech">(They are raised from a Harmony postfix (a piece of code that runs right after the game method) on `Sauna.TurnOn` and `Sauna.TurnOff`.)</span>
- The events also fire while a save is loading. If your handler should ignore those, check [`SaveData.Busy`](../api-core/savedata.md#savedatabusy). It is `true` while a save is loading or being written.

### Limits

- `SetOn` returns `false` when no sauna has that id, and while a save is loading or being written.
- You can't change how strongly a sauna affects the player each moment, for example how fast it sobers them up or improves their hygiene. The game keeps these rates in fields it reads every tick, so a setter is possible. The framework does not offer one yet: eight setters would be a long-term commitment for a feature nobody has asked for. <span class="tech">(The rates are `ReduceDrunk`, `ReduceSMV`, `ImproveHygiene` and the others, public fields read in `PlayerStatsSauna.ApplySauna`.)</span>
- You can't tell when the player is inside a sauna. The game learns this only from a private trigger handler and gives no public signal. A session event would need a new change to the game, and that change is left out. <span class="tech">(The sauna's player-inside trigger is a private Unity message handler on `Sauna`, so it has no public method to patch.)</span>
- You can't throw water on the stones, or control the stove sounds and particles. <span class="tech">(`Sauna.ThrowWater` is not covered.)</span>
- The sauna cooling object is not covered. It is a separate object with its own switch. <span class="tech">(`SaunaCooling` has its own `Toggle(bool)`.)</span>

**Common problems**

- `SetOn` returns `false`: no sauna has that id, a save is loading or being written, no save is loaded, or the game is on the main menu.
- Your `TurnedOn` or `TurnedOff` handler runs while a save is loading: the events also fire then. Check `SaveData.Busy` in the handler and ignore the event when it is `true`.

## Details

Target: Obenseuer 0.5.08 (buildid 25969064), Unity 6000.0.77f1 Mono, BepInEx 6 Unity.Mono ([be.788](https://builds.bepinex.dev/projects/bepinex_be/788/BepInEx-Unity.Mono-win-x64-6.0.0-be.788%2B5b766a3.zip), the tested build) or BepInEx 5 (5.4.23 or later).
Namespace: `ObenseuerCore`. Source: `src/Api/Saunas.cs` and `src/Api/SaunasPatches.cs`. Logs go to `BepInEx\logs\obenseuer.moddingframework.saunas.log`.

Everything not listed in this reference is internal and may change without notice.

The framework changes two of the game's methods when it starts, so it can tell your mod when a sauna is switched on or off. This page calls that change a [feature group](../glossary.md#feature-group). Its feature group is `obenseuer.moddingframework.saunas`, the name you will see in the log.

### When you can call it

- Call everything from your mod's normal code (`OnLoad`, event handlers, `Update`), not from a background thread you started. The game only works safely on its [main thread](../glossary.md#main-thread).
- It is safe to call any of it at any time. With no save loaded, or on the main menu, the reads give an empty list or `null`, and `SetOn` gives `false`. Nothing throws.
- `Saunas.IsAvailable` tells you whether the events work. It is `false` when the framework could not set up the on and off events, or when the feature group was turned off.
- Your event handlers run right after the game has changed the sauna. If your handler throws an error, the framework logs it and still runs the other handlers.

### If something fails to set up

The framework changes some of the game's methods when it starts. If a game update breaks one of them, the framework removes all the changes of the feature group that depends on it. It turns the group off and writes error lines and a warning to the group's log file in `BepInEx\logs`. The rest of the framework keeps working.

- If either of the two changes fails, the feature group turns off and `TurnedOn` and `TurnedOff` stop. One warning goes to the log file named above. `All()`, `Get` and `SetOn` keep working. <span class="tech">(The methods are `Sauna.TurnOn` and `Sauna.TurnOff`. The log line reads "sauna hooks incomplete, sauna events are disabled".)</span>

**Common problems**

- `TurnedOn` and `TurnedOff` never fire and `IsAvailable` is `false`: the framework could not set up the on and off events, or the feature group was turned off. Look for the warning in `BepInEx\logs\obenseuer.moddingframework.saunas.log`. `All()`, `Get` and `SetOn` still work.
