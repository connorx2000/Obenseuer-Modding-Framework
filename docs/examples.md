---
title: Examples
nav_order: 8
---

# Examples

This page lists the example mods in the repository and walks through one of them, ExampleCounter. Read it after the [Getting started](guides/getting-started.md) pages, when you want a complete mod to build and copy from.

The three mods below are working BepInEx [plugins](glossary.md#plugin). That is the advanced form of a mod. The first-time form is a [framework mod](api/framework-mods.md), as in the ExampleCounter below.

- [HelloCore](https://github.com/NomadsReach/Obenseuer-Modding-Framework/tree/main/Examples/HelloCore): save data, events and logging.
- [SlowHunger](https://github.com/NomadsReach/Obenseuer-Modding-Framework/tree/main/Examples/SlowHunger): survival decay and threshold events.
- [WebUiHello](https://github.com/NomadsReach/Obenseuer-Modding-Framework/tree/main/Examples/WebUiHello): an HTML view with light and dark themes. Needs the [WebUi](glossary.md#webui) module.

## ExampleCounter

This mod counts game hours and key presses, and remembers both in the save. It uses a [config setting](guides/getting-started/config.md), a [keybind](guides/getting-started/keybinds.md), an [event](guides/getting-started/using-events.md) and [save data](guides/getting-started/save-data.md). These are the same pieces as the [Getting started](guides/getting-started.md) pages. It is a [framework mod](api/framework-mods.md). To try it:

1. Make a project as in [Your first mod](guides/getting-started/first-mod.md).
2. Rename `MyFirstMod.csproj` to `ExampleCounter.csproj`, so the DLL is named `ExampleCounter.dll`.
3. Replace the code in `Plugin.cs` with the code below.
4. With the game closed, run `dotnet build -c Release -p:DeployToGame=true`. It copies the DLL to `BepInEx\plugins\ExampleCounter\`. Without `-p:DeployToGame=true`, the DLL is in `bin\Release\net472\ExampleCounter.dll`.

```csharp
using System;
using ObenseuerCore;
using UnityEngine;

[Serializable]
public class CounterState
{
    // Game hours counted in this save.
    public int hoursSeen;
    // Key presses counted in this save.
    public int presses;
}

[ModInfo("example.counter", "Example Counter", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public class ExampleCounter : FrameworkMod
{
    // The saved numbers.
    private CounterState state = new CounterState();
    // Setting: count hours at all.
    private ModSetting<bool> countHours;
    // Setting: write a log line every N-th counted hour.
    private ModSetting<int> logEvery;
    // The F7 key.
    private ModKeybind pressKey;

    protected override void OnLoad()
    {
        // Two settings, shown in the Mods menu. LogEveryHours is a slider from 1 to 24.
        countHours = Config.Bind("General", "CountHours", true, "Count game hours while on.");
        logEvery = Config.Bind("General", "LogEveryHours", 1, "Log every N-th counted hour.", 1, 24);
        // Register the F7 key.
        pressKey = Mod.Key("Press", "Count a press", KeyCode.F7);
        // Save and load the numbers with the save game.
        SaveData.Register<CounterState>("example.counter", () => state, s => state = s ?? new CounterState());
        // Count every game hour, and log what was saved each time a save loads.
        Events.HourPassed += OnHour;
        Events.Loaded += () => Log.Info("loaded, hours seen " + state.hoursSeen + ", presses " + state.presses);
    }

    // Unity calls this once per frame.
    private void Update()
    {
        // Skip while a save runs, a menu or the pause screen is open, or the game has switched its keys off.
        if (pressKey == null || SaveData.Busy || UiState.IsMenuOpen || UiState.IsPaused || UiState.IsInputBlocked) return;
        if (!pressKey.Down()) return;
        state.presses++;
        Log.Info("press " + state.presses);
    }

    // Runs at the start of each game hour.
    private void OnHour(int hour)
    {
        if (!countHours.Value || SaveData.Busy) return;
        state.hoursSeen++;
        if (state.hoursSeen % logEvery.Value == 0) Log.Info("hour " + hour + ", hours seen " + state.hoursSeen);
    }
}
```

After it loads, change `LogEveryHours` in the Mods menu and press F7 in the game. Then check the [log](guides/getting-started/logging.md) at `BepInEx\logs\example.counter.log`. The player can rebind the key on the Mods menu page "Mod Keybinds".

The counter counts each game hour once, including across level changes. A load does not repeat the hour it lands on.

The counter is running. The Cookbook, linked below, shows what to add next.

### Common problems

- `BepInEx\logs\example.counter.log` is missing or empty: check whether the mod loaded. Open `BepInEx\logs\ObenseuerCore.log` and read the line that says "framework mods:". Check that the DLL is directly in `BepInEx\plugins\ExampleCounter\`, not one folder deeper. See [my mod does not load](faq.md#my-mod-does-not-load).
- F7 does nothing: the mod skips presses while a save runs, or while a menu or the pause screen is open. It also skips them while the game has switched its keys off. The player may also have rebound the key on the "Mod Keybinds" page.
- Fewer or no "hour" lines in the log than expected: `CountHours` is off, `LogEveryHours` is above 1 (only every N-th counted hour is logged), or a save was running (hours are skipped while `SaveData.Busy`). Fix: turn `CountHours` on or lower `LogEveryHours` in the Mods menu.

## Where to go next

To add more to a mod, see the [Cookbook](guides/cookbook/index.md). To read about every area of the framework, see the [API reference](API.md).
