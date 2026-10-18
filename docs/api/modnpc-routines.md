---
title: Mod NPC routines
parent: People
grand_parent: API reference
has_children: true
nav_order: 3
---

# Mod NPC routines

[`ModNpcRoutines`](modnpc-routines/orders.md) gives a [mod NPC](../glossary.md#mod-npc) (a new person your mod adds to the game) something to do: one-off orders, or a weekly timetable. Use it when an NPC your mod spawned should walk, wander, patrol, sit, sleep or keep set hours.

Status: experimental.

## What it is

A mod NPC is a character that your mod spawns into the game (see [Mod NPCs](modnpcs.md)). On its own it only stands where you put it. This API gives a mod NPC something to do. There are two parts:

- Orders are one-off commands: walk to a point, wander around a spot, patrol along some points, sit on a chair, sleep in a bed, or stop.
- Routines are a weekly timetable. Each step says which days and hours it covers, which [scene](../glossary.md#scene) the NPC is in, and what it does there. While a step is in effect, the framework puts the NPC in that scene. When the clock moves on, the NPC leaves.

These calls work only on NPCs that your mod spawned, not on the game's own NPCs. To read what a game NPC is doing, see [NPC schedules](npc-schedules.md).

## When to use it

- Make a shopkeeper who stands at the counter from 9 to 17 and goes home at night.
- Send an NPC to a point when something happens, and know when it has arrived.
- Have an NPC wander around a yard, patrol a corridor, sit on a bench or sleep in a bed.
- React when an NPC's timetable moves from one activity to the next.

## Quick example

This mod gives a new NPC a timetable: wander near the player's position every day from 8:00 to 18:00. It writes a [log](../guides/getting-started/logging.md) line whenever the NPC changes from one step to the next. The routine spawns the NPC for you, so there is no separate spawn call.

```csharp
using System;
using ObenseuerCore;
using UnityEngine;
using UnityEngine.SceneManagement;

[ModInfo("example.keeper", "Keeper", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class Keeper : FrameworkMod
{
    protected override void OnLoad()
    {
        ModNpcRoutines.StepChanged += args => Log.Info(args.NpcId + " went from " + args.From + " to " + args.To);
        Events.WhenReady(() =>
        {
            var source = default(ObenseuerCore.CharacterInfo);
            foreach (var character in Characters.All())
            {
                if (character.IsPlayer) continue;
                source = character;
                break;
            }
            if (source == null) return;

            string scene = SceneManager.GetActiveScene().name;
            NpcRoutine routine = new NpcRoutine()
                .Wander(NpcDays.Every, TimeSpan.FromHours(8), TimeSpan.FromHours(18), scene, PlayerState.Position, 5f);
            bool set = ModNpcRoutines.Set(Mod.Id, "keeper", source.Id, "Sam", "Keeper", routine);
            Log.Info("Routine set: " + set);
        });
    }
}
```

`Mod.Id` is your [mod id](../glossary.md#mod-id) (the id your mod uses for its files and data). `"keeper"` is the id you choose for this NPC, and every later call uses the same pair of ids. The example uses the first non-player character from [`Characters.All()`](characters.md) as the source character. [`StepChanged`](modnpc-routines/routines.md) is an event: the framework calls your code each time it happens. [`WhenReady`](core-helpers/events-whenready.md) runs your code once a save is loaded (see [Using events](../guides/getting-started/using-events.md)).
