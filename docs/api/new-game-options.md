---
title: Obenseuer Modding Framework new game options API
parent: API reference
nav_order: 117
---

# Obenseuer Modding Framework new game options API

`NewGameOptions` lets your mod register its own new game options and read the player's answers for each save. Use it for a setting that belongs to one character, not to the whole installation.

Status: experimental.

## What it is

When the player starts a new game, the game asks a few questions about the character. This API lets your mod add its own questions to that screen. An option is one of these kinds:

- a switch that is on or off
- a list of named choices
- a whole number inside a range

The player's answers are kept with the save. After the game starts, your mod can read them and change how it behaves for that character. An option is named by the pair of your [mod id](../glossary.md#mod-id) and the key you give it. So two mods can use the same key without clashing.

There is no New Game screen user interface yet. This page is the storage and read half of the feature. Your mod registers its options, and the framework keeps their values in the save. The screen the player clicks on is a later task. Until that screen exists, nothing sets a value. A read then gives you the fallback you passed, and everything else keeps working.

## When to use it

- Offer a setting that belongs to one character rather than to the whole installation, such as a starting bonus or a difficulty tweak.
- Let the player choose between named modes at the start of a game, and remember the choice in the save.
- Read the player's choices for this save and apply them, for example by setting difficulty factors again after every load.

## Quick example

This mod registers one of each kind of option and reads the number once the save is ready. It also writes a [log](../guides/getting-started/logging.md) line when a new game starts.

```csharp
using ObenseuerCore;

[ModInfo("example.startbonus", "Start Bonus", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.1")]
public sealed class StartBonus : FrameworkMod
{
    protected override void OnLoad()
    {
        NewGameOptions.RegisterToggle("example.startbonus", "extra", "Starting bonus", "Start with a small cash bonus.", false);
        NewGameOptions.RegisterChoice("example.startbonus", "tier", "Bonus size", "How large the bonus is.", new string[] { "small", "large" }, new string[] { "Small", "Large" }, 0);
        NewGameOptions.RegisterInteger("example.startbonus", "count", "Bonus amount", "The number of coins the bonus pays.", 10, 1000, 50);
        Events.WhenReady(Apply);
        Events.NewGameStarted += OnNewGame;
    }

    private void Apply()
    {
        int count = NewGameOptions.GetInteger("example.startbonus", "count", 50);
        string tier = NewGameOptions.GetChoice("example.startbonus", "tier", "small");
        Log.Info("This save's bonus is " + count + " coins, tier " + tier + ".");
    }

    private void OnNewGame()
    {
        Log.Info("A new game started. The game's difficulty is not applied yet here.");
    }
}
```

Register in `OnLoad`. It is the one moment a [framework mod](framework-mods.md) is guaranteed to run before a save or a new game can fix the option list. [`Events.WhenReady`](core-helpers/events-whenready.md) runs your code once a save is loaded or a new game's first level has loaded. That is when the values are there to read. See [Using events](../guides/getting-started/using-events.md).

## Common tasks

Read a switch once a save is loaded. Pass the same default you registered, so a save made before your mod was installed reads as the default.

```csharp
Events.WhenReady(() =>
{
    if (NewGameOptions.GetBool("example.startbonus", "extra", false))
        Log.Info("The starting bonus is on for this save.");
});
```

Re-apply a difficulty factor after every load.

```csharp
Events.WhenReady(() =>
{
    bool extra = NewGameOptions.GetBool("example.startbonus", "extra", false);
    GameDifficulty.SetFactor("example.startbonus", DifficultyMultiplier.NeedsSpeed, extra ? 1.2f : 1f);
});
```

[`GameDifficulty`](hud/gamedifficulty.md) factors are session only. The game rebuilds them from the save it loads, so a factor your mod set in an earlier session is gone. Read your option again on every load, as the example does. See [HUD, console, achievements and difficulty](hud.md).

React the moment a new game starts, before the first level shows.

```csharp
Events.NewGameStarted += () => Log.Info("The player confirmed a new character.");
```

Keep saved values stable. Give each choice a short id that describes the answer, not its position in the list. Then reordering the `ids` and `titles` arrays later does not change what an old save means.

## Reference

The full list of what the API gives you:

```csharp
public static class NewGameOptions
{
    public static bool RegisterToggle(string modId, string key, string title, string description, bool defaultValue);
    public static bool RegisterChoice(string modId, string key, string title, string description, string[] ids, string[] titles, int defaultIndex);
    public static bool RegisterInteger(string modId, string key, string title, string description, int min, int max, int defaultValue);
    public static bool GetBool(string modId, string key, bool fallback);
    public static string GetChoice(string modId, string key, string fallback);
    public static int GetInteger(string modId, string key, int fallback);
}
```

- **Register from `FrameworkMod.OnLoad`.** A registration returns `true` when the option was added. It returns `false` and writes one line to the log when the same `modId` and `key` are already registered. It does the same when the option list is already fixed, which happens once a save loads or a new game starts. The log line says to register from `OnLoad`.
  - Technical: the option list is fixed at the first `FrameworkMod.OnLoad` cycle's end of the new game or save load. Registration order is the order the screen will show.
- **The pair of `modId` and `key` names one option.** The pair is matched exactly, so `Loud` and `loud` are two options. Two mods may use the same key.
- **`title` and `description` are the text for the screen.** `title` is the short name, and `description` is the sentence under it. Both are shown by the later New Game screen task.
- **A choice is stored as its id string, never as a position.** `ids` holds the stable ids, and `titles` holds the text the player sees. `defaultIndex` says which entry is selected to begin with. The two arrays must have the same length.
- **An integer is stored as its number.** `min` and `max` say what the later screen will allow.
- **Bad arguments throw `ArgumentException`.** `description` may be empty. These are bad arguments:
  - an invalid mod id
  - an empty key or title
  - a null or empty `ids` or `titles`
  - an empty string inside either array
  - arrays of different lengths
  - a `defaultIndex` outside the list
  - a `min` larger than `max`
- **`GetBool`, `GetChoice` and `GetInteger` give back the value for the save**, or the `fallback` you pass when there is none. Pass the same value you registered as the default.
  - Technical: reads look in the values loaded from the save first, then in the values chosen for a new game that has not saved yet. If neither has a value, they give the fallback.
- **A saved value that your code cannot read gives the fallback and writes one line to the log.** An example is a number edited into the file by hand.
- **A read for an option that was never registered, or that has no value yet, gives the fallback.** It also writes one line to the log. A value of `0` or `false` is a value, not a missing one.
- **Values are written when the save is written**, the first time and every time after that. They are read when a save loads. The first save of a new game is the moment the chosen values leave memory and become part of the save.
- **A save made before your mod existed has no entry for your options.** Nothing is written for it, and every read gives your fallback. The file gains your values the next time that save is written.

**Common problems**

- A registration returns `false` and writes one line to the log: the same `modId` and `key` are already registered, or the option list is already fixed. Register from `FrameworkMod.OnLoad`.
- A read gives the fallback: there is no New Game screen yet, so nothing sets a value. Reads also give the fallback before a save loads and before a new game saves. So do reads of an option that was never registered or has no value yet. The same goes for a saved value your code cannot read, and for a save made before your mod existed. Pass the same value you registered as the default, so a save made before your mod was installed reads as the default.
- A difficulty factor your mod set is gone after a load: `GameDifficulty` factors are session only. Read your option again and set the factor from `Events.WhenReady` on every load.

### Events

- **`Events.NewGameStarted`** is an [`Action`](../glossary.md#action-and-actionlttgt) with no argument. It fires when a player confirms a new character. By then the framework has taken the chosen values into memory and reset its own per-save state.
- **The game's difficulty is not applied yet when `Events.NewGameStarted` fires.** A handler must not read the difficulty or the character's applied start there. Read it later, for example from the first level's `Events.WhenReady` or the first `Events.SceneLoaded`.
  - Technical: the event is raised in the postfix of `NewGame.SaveCharacter`, and the game applies the chosen difficulty one frame after the new scene starts, in `PlayerIdentity.StartDelay`.
- **`Events.Loaded`** is an `Action` with no argument, raised once a save has loaded and the framework's saved values are in memory. `Events.WhenReady` is the easier way to run code at that point. A game loaded from a save never raises `Events.NewGameStarted`.

## Details

Target: Obenseuer 0.5.08 (buildid 25969064), Unity 6000.0.77f1 Mono, BepInEx 6 Unity.Mono ([be.788](https://builds.bepinex.dev/projects/bepinex_be/788/BepInEx-Unity.Mono-win-x64-6.0.0-be.788%2B5b766a3.zip), the tested build) or BepInEx 5 (5.4.23 or later).
Namespace: `ObenseuerCore`. Logs go to `BepInEx\logs\obenseuer.moddingframework.startoptions.log`.

Everything not listed in this reference is internal and may change without notice. This page is experimental and may change too.

### Where the values live

The framework keeps one file per save for every mod's new game options: `Saves\<character>\<save>\mods\obenseuer.moddingframework.startoptions.json`. It is written when the save is written and read when the save loads, like the framework's other per-mod save files. A mod that is uninstalled leaves its entries in that file behind, where they do nothing.

### No New Game screen yet

The framework can already store and read these options, but nothing shows them to the player yet. The New Game screen is a later task, because its gamepad support depends on another branch that is not merged. Until it ships, a registered option keeps its default value and every read gives your fallback. You can write and test your mod's read path now, and the screen will fill in the values later.

### When you can call it

- **Call everything from your mod's normal code, not from a background thread you started.** The game and Unity only work safely on one thread, the main thread (see the [glossary](../glossary.md#main-thread)). That includes the code that handles `Events.NewGameStarted`.
- **Register during `FrameworkMod.OnLoad`.** Later registration returns `false` and writes one line to the log.
- **Reading is safe at any time.** Before a save loads and before a new game saves, you get the fallback you passed.
