---
title: Common tasks
parent: Instruments
grand_parent: Objects and devices
nav_order: 1
---

# Common tasks

Short snippets for the usual jobs with [`Instruments`](guitars.md): list the guitars, play a strum, play a short song, and stop what your mod plays.

List the guitars in the scene and show which ones can play a strum right now.

```csharp
foreach (GuitarInfo guitar in Instruments.All())
    Log.Info(guitar.Id + " held=" + guitar.Equipped + " canPlay=" + guitar.CanPlayNote);
```

Play one strum on every guitar that can play. The first argument is your [mod id](../../glossary.md#mod-id), `Mod.Id`.

```csharp
foreach (GuitarInfo guitar in Instruments.All())
    if (guitar.CanPlayNote) Instruments.PlayNote(Mod.Id, guitar.Id);
```

Play a short song of four strums on a guitar. The numbers are waits in seconds: 0 before the first strum, then half a second, half a second and one second, each counted from the strum before it.

```csharp
string guitarId = "put an id from All() here";
bool started = Instruments.PlaySong(Mod.Id, guitarId, new float[] { 0f, 0.5f, 0.5f, 1f });
Log.Info("Song started: " + started);
```

Stop everything your mod is playing.

```csharp
Instruments.Clear(Mod.Id);
```
