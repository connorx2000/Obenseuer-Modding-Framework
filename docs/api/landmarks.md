---
title: Landmarks
parent: Places and travel
grand_parent: API reference
has_children: true
nav_order: 7
---

# Landmarks

[`Landmarks`](landmarks/game-landmarks.md) lets your mod read and unlock the landmarks on the game's map, and add landmarks of its own. Use it to reward the player with a place, or to mark a place your mod adds.

This page uses `Vector3` from Unity. The [glossary](../glossary.md#types-that-come-from-unity) says what each one is.
{: .unity-types}

## What it is

The map in the game shows landmarks: places the player has discovered, each with an icon and a name. This API gives your mod these jobs:

- Read the game's landmarks, and see which ones the player has.
- Unlock a landmark.
- Place a game landmark at a spot you choose.
- Create new landmarks of your own, and simple map markers.

Your own landmarks and markers stay on the map after the player saves and loads.

## When to use it

- Check whether the player has discovered a landmark.
- Unlock a landmark as a reward, with the game's own notification.
- Put a marker on the map for a place your mod adds, such as a camp or a quest spot.
- React when any landmark is added to the map.

## Quick example

This mod puts a red marker called Camp on the map once a save is loaded. It also writes a [log](../guides/getting-started/logging.md) line each time any landmark is added to the map.

```csharp
using ObenseuerCore;
using UnityEngine;

[ModInfo("example.camp", "Camp Marker", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class CampMarker : FrameworkMod
{
    protected override void OnLoad()
    {
        Landmarks.Added += args => Log.Info("Landmark added: " + args.Title);
        Events.WhenReady(() =>
        {
            if (!Landmarks.Mark(Mod.Id, "camp", "Camp", new Vector3(10f, 0f, 10f)))
                Log.Info("The marker was not placed.");
        });
    }
}
```

`Mark` places a simple red dot on the map. It gives back `true` when the dot is there afterwards, and `false` when the map is not ready. The example logs a line only in that case. `Added` is an event: the framework calls your code each time it happens. [`Events.WhenReady`](core-helpers/events-whenready.md) waits until a save is loaded before it runs your code. See [Using events](../guides/getting-started/using-events.md).
