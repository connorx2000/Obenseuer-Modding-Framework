---
title: Mod NPCs
parent: People
grand_parent: API reference
has_children: true
nav_order: 4
---

# Mod NPCs

[`ModNpcs`](modnpcs/types-and-members.md) adds new people to the game, called [mod NPCs](../glossary.md#mod-npc), and simple background figures for a crowd. Use it when your mod needs a person that the game's own cast does not have.

This page uses `Vector3` from Unity. The [glossary](../glossary.md#types-that-come-from-unity) says what each one is.
{: .unity-types}

Status: experimental. Every member on this page is experimental and may change.

## What it is

A [mod NPC](../glossary.md#mod-npc) is a new person your mod adds to the game. The game has no way to add a person from nothing, so the framework builds a copy of one that already exists. You choose:

- the source: the person from the game to copy
- the body: the 3D body the copy uses
- a new name and a place to stand

These pages cover adding and removing those people, keeping them across a save and load, and changing how they look. They also cover [extras](modnpcs/extras.md): simple background figures for a crowd. Making a mod NPC walk, wander or follow a timetable is on the [mod NPC routines](modnpc-routines.md) page.

## When to use it

- Put a new shopkeeper, guard or visitor into a place in the game.
- Keep that person in the game after the player saves and loads.
- Fill a room with background figures that stand and watch.

## Quick example

This mod waits until a save is loaded. Then it adds a person named Gus Greeter three metres in front of the player. The copy is built from the first person in [`Characters.All()`](characters.md) who is not the player. The mod writes the new person's number to the [log](../guides/getting-started/logging.md).

```csharp
using ObenseuerCore;
using UnityEngine;

[ModInfo("example.greeter", "Greeter", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class Greeter : FrameworkMod
{
    protected override void OnLoad()
    {
        Events.WhenReady(() =>
        {
            foreach (CharacterInfo character in Characters.All())
            {
                if (character.IsPlayer) continue;
                Vector3 spot = PlayerState.Position + Vector3.forward * 3f;
                int id = ModNpcs.Spawn(Mod.Id, "greeter", character.Id, "Gus", "Greeter", spot, 0f);
                Log.Info(id == 0 ? "Could not spawn the greeter." : "Spawned the greeter as character " + id);
                break;
            }
        });
    }
}
```

[`Events.WhenReady`](core-helpers/events-whenready.md) waits until a save is loaded before it runs your code. See [Using events](../guides/getting-started/using-events.md) for more on it. This person is only there until the [scene](../glossary.md#scene) changes. To add a person who stays after a load, use [`SpawnSaved`](modnpcs/spawnsaved.md).
