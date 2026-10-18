---
title: Spawners
parent: Objects and devices
grand_parent: API reference
has_children: true
nav_order: 16
---

# Spawners

[`Spawners`](spawners/reading-spawners.md) lets your mod read and change the spawners in the [scene](../glossary.md#scene) you are in, and tells you when one spawns something. Use it to change how things respawn or to react to a spawn.

## What it is

A spawner is a script in a scene that places things in the world and puts them back after a while. It can place items you can pick up, furniture, blueprints, animals or characters. This API gives your mod these jobs:

- List the spawners in the scene you are in.
- Read and change how they respawn.
- Make one spawn right now.
- Get told each time one spawns something.

The spawning itself is still done by the game.

Spawner is a family of scripts. The base one is `Spawner`. `All()` also lists the others in the family: `CollectibleItemSpawner`, `AnimalSpawner`, `PrefabSpawner`, `FurnitureBlueprintSpawner` and `CharacterSpawner`. Each spawner's kind is in its `TypeName`.

## When to use it

- Log or react when something spawns in a scene.
- Turn respawning on or off for one spawner, or change its chance.
- Change how long a spawner waits before it respawns.
- Make a spawner spawn right now.

## Quick example

This mod writes a [log](../guides/getting-started/logging.md) line each time a spawner spawns something. When the game is ready, it also logs every spawner in the scene with its kind and how many things it has spawned.

```csharp
using ObenseuerCore;

[ModInfo("example.spawnwatch", "Spawn Watch", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class SpawnWatch : FrameworkMod
{
    protected override void OnLoad()
    {
        Spawners.Spawned += args => Log.Info(args.TypeName + " spawned something, it now has " + args.SpawnedCount);
        Events.WhenReady(() =>
        {
            foreach (SpawnerInfo spawner in Spawners.All())
                Log.Info(spawner.TypeName + " " + spawner.Id + ": " + spawner.SpawnedCount + " spawned");
        });
    }
}
```

`Spawned` is an event: the framework calls your code each time it happens. [`Events.WhenReady`](core-helpers/events-whenready.md) runs your code when the game is ready. See [Using events](../guides/getting-started/using-events.md) for more on events and `WhenReady`.
