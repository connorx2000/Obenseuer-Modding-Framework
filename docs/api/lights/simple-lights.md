---
title: Simple lights
parent: Lights
grand_parent: Objects and devices
nav_order: 2
---

# Simple lights

A simple light is a fixture that is either on or off. This page lists the `Lights` calls, property and event for simple lights, with the rules behind them.

`Lights.All()` lists simple lights. Each one is a `LightInfo`: a snapshot of the light at the moment you asked. The snapshot does not update by itself, so ask again for fresh values.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>Lights</code> <span>simple light calls and event</span></p>

Call `All()` or `Get(id)` to read simple lights, and `SetOn(id, on)` to switch one. `Toggled` fires after a light is switched.

<div class="api-pair" markdown="1">

```csharp
public static class Lights
{
    public static List<LightInfo> All();
    public static LightInfo Get(string id);
    public static bool SetOn(string id, bool on);
    public static event Action<LightToggledArgs> Toggled;
    public static bool IsAvailable { get; }
}
```

| Method | Returns | What it does |
|---|---|---|
| `All()` | `List<LightInfo>` | Every simple light in the active scene. |
| `Get(id)` | `LightInfo` | One light. `null` when no light has that id. |
| `SetOn(id, on)` | `bool` | Switches the light. `true` when it now matches what you asked for. |

| Property | Type | What it means |
|---|---|---|
| `IsAvailable` | `bool` | `false` when a light event could not be set up. |

| Event | Gives you | When it fires |
|---|---|---|
| `Toggled` | `LightToggledArgs`: the light's id and whether it is on after the change. | Right after a light is switched on or off. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>LightInfo</code> <span>a snapshot of one simple light</span></p>

`All()` and `Get(id)` give you one of these. `On` is as it was when you asked, so ask again for a fresh value. <span class="tech">(It is built from the game's `LightTurnOffOn` component and its `on` flag.)</span>

<div class="api-pair" markdown="1">

```csharp
public sealed class LightInfo
{
    public string Id { get; }
    public bool On { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `Id` | `string` | Its save id. When it has none, a text path to where it sits in the scene. |
| `On` | `bool` | `true` when the light is on. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>LightToggledArgs</code> <span>what the Toggled event tells you</span></p>

`Toggled` gives you one of these when a light is switched. `On` is the state after the change.

<div class="api-pair" markdown="1">

```csharp
public sealed class LightToggledArgs
{
    public string Id { get; }
    public bool On { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `Id` | `string` | The light, the same as `LightInfo.Id`. |
| `On` | `bool` | `true` when the light is on after the change. |

</div>
</div>

## Things to know

- `Id` is the light's save id, or a text path when it has none. A path changes if the object is moved or renamed, so do not keep it for long. <span class="tech">(The id is the `GUID` field, or the scene path when `GUID` is empty.)</span>
- `SetOn(id, on)` switches the light the way the game does. The materials, the light objects and the signals wired to the light all run as they do in the game. If the light is already in the state you asked for, nothing happens. You get `true`, and the signals do not fire twice. <span class="tech">(It calls the game's `turnLightOn()` or `turnLightOff()`. The return value is `true` when `LightTurnOffOn.on` now equals the requested value.)</span>
- Switching on a light that is fading out brings it back properly. The game fades some lights out over a few frames. If you ask for such a light to turn on, the framework uses the game's fade-in instead of the plain switch. The plain switch would leave it dark. <span class="tech">(It calls `TurnLightOnFade(false)` when the private `fadeOut` flag is set.)</span>
- `Toggled` fires whenever a light really changes between on and off, whoever changed it: your `SetOn` calls, the player, or the game's own scripts. A call that changes nothing does not fire it. <span class="tech">(It is raised after `turnLightOn`, `turnLightOff` and `TurnLightOffFade` when `on` is different from before.)</span>
- For a light that fades out, `Toggled` fires at once, not when the fade is over. The light is already counted as off, and its brightness then fades out over the next few frames. <span class="tech">(`TurnLightOffFade` sets `on = false` immediately.)</span>

## Limits

- `All()` and `Get(id)` only see the [scene](../../glossary.md#scene) you are in. Lights that are switched off at the object level, or in another scene, are not listed. Whether a fixture whose script sits on a switched-off object should be listed is still undecided. <span class="tech">(They use `FindObjectsByType`, which skips inactive objects.)</span>
- `SetOn` returns `false` for an id nobody has, and while a save is loading or being written. It also returns `false` for a [multi-state light](multi-state-lights.md), because `All`, `Get` and `SetOn` don't see those.
- Loading a save does not fire `Toggled`, even though lights change on screen. The game restores each light's saved state before it switches the light, so the framework sees no change. <span class="tech">(The game's `OnLoadingGame` writes `on` from the save and then calls `turnLightOn` or `turnLightOff`. By then `on` already holds the restored value.)</span>

**Common problems**

- `SetOn` returns `false`: no light has that id, a save is loading or being written, or the id belongs to a multi-state light. For a multi-state light, use `SetState` to change the lit setting.
- `Toggled` does not fire: the light did not really change, or the change came from loading a save. It also stops when the simple-light feature group is turned off, which makes `IsAvailable` `false`. See [Details](details.md#if-something-fails-to-set-up).
