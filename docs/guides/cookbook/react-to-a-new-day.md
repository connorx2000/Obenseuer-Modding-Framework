---
title: React to a new day
parent: Cookbook
nav_order: 3
---

# React to a new day

Write a log line each new game day, and set the clock to 07:00 on Mondays. Use this when your mod should do something at the start of each day.

## The example

[`Events.DayPassed`](../../api-core/events.md) is the signal for a new day. [`GameTime.DayOfWeek`](../../api-core/gametime.md) says which weekday it is, and [`WorldClock.SetTimeOfDay`](../../api/world/worldclock.md) moves the clock.

Start from the project you made in [Your first mod](../getting-started/first-mod.md), and replace the code in `Plugin.cs` with the code below.

```csharp
using System;
using ObenseuerCore;

[ModInfo("example.newday", "New Day", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public class NewDay : FrameworkMod
{
    protected override void OnLoad()
    {
        // Run OnDayPassed every time the game day changes.
        Events.DayPassed += OnDayPassed;
    }

    private void OnDayPassed(int day)
    {
        // Write the new day number and its weekday to the log.
        Log.Info("day " + day + " is " + GameTime.DayOfWeek);
        // Do nothing while a save or load is running.
        if (SaveData.Busy) return;
        // On Mondays, move the clock to 07:00.
        if (GameTime.DayOfWeek == DayOfWeek.Monday) WorldClock.SetTimeOfDay(7, 0);
    }
}
```

Build it and watch a day pass:

1. Close the game and run `dotnet build -c Release -p:DeployToGame=true` in your mod folder.
2. Start the game and load a save.
3. Sleep or wait until the next day.
4. Open `BepInEx\logs\example.newday.log` in the game folder to see the day lines.

The mod now reacts to each new day. Things to know, below, covers the save check and hourly events.

### Common problems

- The log has no day lines: a line is written only when the game day changes, so sleep or wait until the next day first. If it still has none, see [my mod does not load](../../faq.md#my-mod-does-not-load).
- The clock does not move to 07:00 on a Monday: the code does nothing while a save or load is running. `WorldClock.SetTimeOfDay` also returns `false` when the clock is not ready, during a save, or when the time is earlier than the current game time.

## Things to know

- [`Events.DayPassed`](../../api-core/events.md) gives the new day counter. [`GameTime.DayOfWeek`](../../api-core/gametime.md) gives the weekday.
- Check [`SaveData.Busy`](../../api-core/savedata.md) before you change game state.
- Use `Events.HourPassed` if you need a check on every hour. See [Using events](../getting-started/using-events.md).

## Where to go next

Next: [Add a Mods menu page](mods-menu-page.md).
