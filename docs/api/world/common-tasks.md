---
title: Common tasks
parent: World
grand_parent: Places and travel
nav_order: 1
---

# Common tasks

Short snippets for the usual jobs with the world API: list and change the weather, move the clock, read radiation and set up the player's wake-up.

List the weather the game has, so you know which index means what. The index is what [`WorldWeather.Set`](worldweather.md) takes:

```csharp
for (int i = 0; i < WorldWeather.Count; i++)
    Log.Info(i + ": " + WorldWeather.NameOf(i));
```

Change the weather to the first one in the list, blending over 30 seconds. `Set` throws an `ArgumentException` for an index that does not exist, so stay between `0` and `Count - 1`:

```csharp
if (WorldWeather.Count > 0 && !WorldWeather.Set(0, 30f)) Log.Warn("The weather could not be changed.");
```

Move the clock to noon with [`WorldClock.SetTimeOfDay`](worldclock.md). It returns `false` when noon has already passed today, because the game does not move the day back:

```csharp
if (!WorldClock.SetTimeOfDay(12, 0)) Log.Info("It is already past noon, or the clock is not ready.");
```

Read how much radiation the player has taken in, and how strong it is where they stand, with [`WorldRadiation`](worldradiation.md):

```csharp
Log.Info("Dose " + WorldRadiation.Dose + ", rate " + WorldRadiation.DoseRate + " mSv/h");
```

Set up the player's wake-up for the next level load with [`WorldPower`](worldpower.md). With these settings, after a blackout they wake on the ground after a short delay:

```csharp
WorldPower.ScheduleWakeUp(1f, 2f, WakeUpAnimation.OnGround, autoWakeUp: true);
```
