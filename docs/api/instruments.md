---
title: Instruments
parent: Objects and devices
grand_parent: API reference
has_children: true
nav_order: 8
---

# Instruments

[`Instruments`](instruments/guitars.md) lets your mod find the guitars in the [scene](../glossary.md#scene), play them, and react when someone strums one. Use it to reward guitar playing or to play a short tune from a quest.

## What it is

The guitar is the only musical instrument in the game. The player holds it in a hand and strums it with the Primary Use key. People nearby may comment on the playing.

This API gives your mod these jobs:

- List the guitars in the scene you are in.
- Play a single strum, or a timed run of strums.
- Hold a long note.
- Get told when a strum is played and when a [witness](../glossary.md#witness) (a person nearby) comments on it.

## When to use it

- Do something when the player strums a guitar, for example count strums or give a reward.
- React when a person nearby comments on the playing.
- Play a short tune on the guitar the player is holding, for example as part of a quest or a gag.

## Quick example

This mod [logs](../guides/getting-started/logging.md) each strum and each comment a nearby person makes on the playing.

```csharp
using ObenseuerCore;

[ModInfo("example.guitarwatch", "Guitar Watch", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class GuitarWatch : FrameworkMod
{
    protected override void OnLoad()
    {
        Instruments.GuitarPlayed += args => Log.Info("Strum on guitar " + args.GuitarId);
        Instruments.GuitarCommented += args => Log.Info("NPC " + args.NpcId + " commented, positive=" + args.Positive);
    }
}
```

`GuitarPlayed` and `GuitarCommented` are events: the framework calls your code each time they happen. See [Using events](../guides/getting-started/using-events.md) for more on events.

To see it work, hold a guitar in the game and strum it. The lines appear in the log.
