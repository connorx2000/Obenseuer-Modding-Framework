---
title: Common tasks
parent: Trains
grand_parent: Places and travel
nav_order: 1
---

# Common tasks

Short examples for the train jobs mods do most: list the trains, flip switches, turn on loaders and find taken track.

This page uses `GameObject` from Unity. The [glossary](../../glossary.md#types-that-come-from-unity) says what each one is.
{: .unity-types}

List every [train on the track](trains-on-the-track.md).

```csharp
foreach (TrackTrainInfo train in Trains.All())
    Log.Info(train.Id + ": " + train.WagonType + " at speed " + train.Speed);
```

Send every [track switch](switches-and-loaders.md) that points at branch 0 down branch 1 instead. The call is refused when a train is on the switch's track.

```csharp
foreach (UnityEngine.GameObject sw in Trains.Switches)
{
    if (Trains.GetBranch(sw) == 0) Trains.SetBranch(sw, 1, force: false);
}
```

Turn every wagon loader on.

```csharp
foreach (UnityEngine.GameObject loader in Trains.Loaders)
    Trains.SetLoaderOn(loader, true);
```

List the [pieces of track](pieces-of-track.md) that a train is on.

```csharp
foreach (TrackSegmentInfo segment in Trains.Segments())
    if (segment.Occupied) Log.Info("Track " + segment.Id + " is taken by train " + segment.OccupiedBy);
```
