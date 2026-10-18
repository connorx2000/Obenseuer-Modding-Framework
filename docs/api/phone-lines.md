---
title: Phone lines
parent: Phones, mail and media
grand_parent: API reference
has_children: true
nav_order: 8
---

# Phone lines

[`PhoneLines`](phone-lines/phones-and-calls.md) lets your mod use the phones in the [scene](../glossary.md#scene) you are in: ring them, answer them, start calls and react to calls. Use it for quest or story moments that involve a phone.

## What it is

A phone line is one phone in the game world, the kind the player can walk up to and use. This API gives your mod these jobs:

- List the phones in the scene.
- Make a phone ring, as if someone were calling.
- Answer or decline a ring.
- Start a call from a phone.
- Check whether a phone is busy or ringing.
- Get told when a phone rings, or when a call starts or ends.

The numbers a phone can dial are the phone book, `Phones`, on the [Phones page](phones.md). They are not repeated here. This page does not cover the dial pad on screen. It also does not let you hang up a call that is going on.

## When to use it

- Make a phone ring as part of a quest, with a number the player knows.
- Start a call from a phone for a story moment.
- React when the player starts or ends a phone call.
- Check whether a phone is free before you ring it.

## Quick example

This mod writes a [log](../guides/getting-started/logging.md) line when a phone rings, and when a call starts or ends. Once a save is loaded, it also logs how many phones the scene has.

```csharp
using ObenseuerCore;

[ModInfo("example.phonewatch", "Phone Watch", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class PhoneWatch : FrameworkMod
{
    protected override void OnLoad()
    {
        PhoneLines.Ringing += args => Log.Info("Phone " + args.Id + " is ringing, caller " + args.Number);
        PhoneLines.CallStarted += args => Log.Info("A call started on " + args.Id + " with " + args.Number);
        PhoneLines.CallEnded += args => Log.Info("A call ended on " + args.Id);
        Events.WhenReady(() =>
        {
            Log.Info("This scene has " + PhoneLines.All().Count + " phones.");
        });
    }
}
```

`Ringing`, `CallStarted` and `CallEnded` are events: the framework calls your code each time they happen. [`Events.WhenReady`](core-helpers/events-whenready.md) waits until a save is loaded before it runs your code. See [Using events](../guides/getting-started/using-events.md).
