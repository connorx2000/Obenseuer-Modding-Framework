---
title: Common tasks
parent: Framework core classes
grand_parent: Framework basics
nav_order: 1
---

# Common tasks

Short recipes for the most common jobs with the [framework](../glossary.md#framework) core classes. Each one is a few lines you can copy. The page for each class has the full rules.

Write to your own [log](../guides/getting-started/logging.md) file with [`CoreLog`](corelog.md). The file is `BepInEx\logs\example.mymod.log`, where `example.mymod` is your [mod id](../glossary.md#mod-id).

```csharp
CoreLog log = CoreLog.For("example.mymod");
log.Info("started");
log.Warn("something looks odd");
```

Do something at the start of every game hour with [`Events`](events.md).

```csharp
Events.HourPassed += hour => CoreLog.For("example.mymod").Info("hour " + hour);
```

Keep a number in the player's [save](../guides/getting-started/save-data.md) with [`SaveData`](savedata.md). `s ?? new Counter()` handles a save made before your mod existed.

```csharp
Counter state = new Counter();
SaveData.Register<Counter>("example.mymod", () => state, s => state = s ?? new Counter());
```

`Counter` is a small class of your own:

```csharp
[System.Serializable]
public class Counter
{
    public int visits;
}
```

Read a need and set it with [`Survival`](survival.md). This heals the player to full when the health value drops below 20.

```csharp
if (Survival.Get(Stat.Health) < 20f) Survival.Set(Stat.Health, 100f);
```

Switch the camera to third person with [`CameraView`](cameraview.md), if the game allows it.

```csharp
if (CameraView.IsAvailable) CameraView.SetThirdPerson();
```
