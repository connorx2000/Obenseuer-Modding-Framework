---
title: Lights
parent: Objects and devices
grand_parent: API reference
has_children: true
nav_order: 10
---

# Lights

[`Lights`](lights/simple-lights.md) lists the light fixtures a player can switch, switches them, and tells you when one changes. Use it to control lights from a quest or a scripted event.

## What it is

The game has two kinds of light fixture that a player can switch:

- A simple light is either on or off.
- A multi-state light has several settings. Exactly one of them is lit at a time, or none when it is all off.

This API gives your mod these jobs, for both kinds, in the [scene](../glossary.md#scene) you are in:

- List the lights.
- Read whether a light is on, or which setting is lit.
- Switch a light.
- Get told when a light changes.

When you switch a light through this API, the game runs its normal switching code. The light objects, the materials and the signals wired to the light all behave as if the player had done it.

One more light script, `LightsON`, is not covered here. See [What you can't do (yet)](lights/what-you-can-t-do-yet.md).

## When to use it

- Turn a light on or off for a quest or a scripted event.
- React when a light is switched, by the player or by the game, for example to log it or play a sound.
- Step a multi-state light through its settings.
- List every light fixture in a scene.

## Quick example

This mod writes a [log](../guides/getting-started/logging.md) line each time a light is switched on or off. Once a save is loaded, it logs how many simple lights and multi-state lights the [scene](../glossary.md#scene) has.

```csharp
using ObenseuerCore;

[ModInfo("example.lightwatch", "Light Watch", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class LightWatch : FrameworkMod
{
    protected override void OnLoad()
    {
        Lights.Toggled += args => Log.Info("light " + args.Id + " is now " + (args.On ? "on" : "off"));
        Events.WhenReady(() =>
        {
            Log.Info(Lights.All().Count + " lights and " + Lights.AllMultiState().Count + " multi-state lights here.");
        });
    }
}
```

`Toggled` is an event: the framework calls your code each time it happens. [`Events.WhenReady`](core-helpers/events-whenready.md) waits until a save is loaded before it runs your code. See [Using events](../guides/getting-started/using-events.md) for more on events and `WhenReady`.
