---
title: Framework core classes
parent: Framework basics
grand_parent: API reference
has_children: true
nav_order: 3
---

# Framework core classes

This page is the reference for the [framework](glossary.md#framework)'s core classes: patching, logging, events, game time, the player's needs, the camera, the built-in tweaks, the Mods menu, computer apps and the player's look. Use it when your mod needs one of these building blocks rather than one area of the game.

Status: experimental.

## What it is

The classes on this page are the building blocks that nearly every mod uses, plus a few bigger ones. None of them is tied to one part of the game. [`CoreLog`](api-core/corelog.md) gives your mod its own [log](guides/getting-started/logging.md) file. [`CorePatcher`](api-core/corepatcher.md) changes game methods in a way that stays safe next to other mods. [`Events`](api-core/events.md) tells you when the clock moves and when the game saves or loads. [`GameTime`](api-core/gametime.md) gives the date and time. The framework also gives the player's [OC](glossary.md#oc) (Open Sewer Coins) balance and a place to keep your mod's own [save data](guides/getting-started/save-data.md).

The rest of the page covers the player's needs ([`Survival`](api-core/survival.md): hunger, thirst, health and so on) and the camera ([`CameraView`](api-core/cameraview.md)). It also covers the built-in gameplay [tweaks](api-core/tweaks.md), the Mods menu ([`ModMenu`](api-core/modmenu.md)), apps on the in-game computers ([`ComputerApps`](api-core/computerapps.md)) and the player's look ([`PlayerLook`](api-core/playerlook.md)). The list of game areas and the known API inconsistencies are on the [API reference](API.md).

The samples on this page are written as [BepInEx](glossary.md#bepinex) (the game's mod loader) 6 [plugins](glossary.md#plugin). On BepInEx 5, delete the `using BepInEx.Unity.Mono;` line from each sample. The plugin class still derives from `BaseUnityPlugin`. The `CoreTour` sample under GameTime uses the simpler [framework mod](api/framework-mods.md) form, which needs no BepInEx line at all.

## When to use it

- Write a line to your own log file and read it in `BepInEx\logs`.
- Run code when a new hour or a new day starts, or when a save has loaded.
- Keep a number or a list of your own in the player's save, so it is still there after a reload.
- Make the player hungrier, slow down thirst, or react when the player faints.
- Add a page to the [Mods menu](api-core/modmenu.md), or an app to the in-game computers.
- Change what a game method does, in a way that stays safe when another mod changes the same method.

## Quick example

The example is the `CoreTour` framework mod under GameTime, below. It writes one log line at the start of every game day. Once a save is loaded, it also logs the hour and the player's hunger value. [`Events.DayPassed`](api-core/events.md) is an event: the framework calls your code each time it happens. [`Events.WhenReady`](api/core-helpers/events-whenready.md) waits until a save is loaded and then runs your code once. [Using events](guides/getting-started/using-events.md) explains both.

The framework's own config file, `BepInEx\config\obenseuer.moddingframework.cfg`, has these settings. Most of them are also listed on [Plugin](api-core/plugin.md#config-file-settings).

- `[Safety] RefuseUnknownBuild = false`. When true, every patch is refused on a game build other than 0.5.08 (buildid 25969064). By default an unknown build is patched, and a warning is logged. Takes effect on the next start.
- `[Debug] Master = false`. The framework's debug trace, written to `BepInEx\logs\ObenseuerCore.debug.log`. It is also in the Mods menu and the `core_debug` console command. `[Debug] TraceFilter` limits it to type name prefixes. See [Debug](api/debug.md).
- `[Debug] Verbose = false`. When true, [`CoreLog.Debug`](api-core/corelog.md) writes.
- `[Debug] AccessProbe = false`. Runs the L0 access probe at start and logs the result.
- `[Debug] PlayerLookSelfCheck = false`. Runs the [PlayerLook](api-core/playerlook.md) clamping and gender-rule self-check at start and logs the result.
- `[Debug] ModIdsSelfCheck = false`. Runs the [mod id](glossary.md#mod-id) and save file name self-check at start. It logs `modids self-check ok`, or the first failing case.
- `[Debug] NetBlobSelfCheck = false`. Runs the blob chunking, CRC32 and gzip self-check at start. It logs `netblob self-check ok`, or the first failing case.
- `[Debug] SaveDataSelfCheck = false`. Registers the `core_savedata_check` console command, which logs `SaveData.SelfCheck()`. The check changes the live inventory, so use a scratch save (see [Game save data entries](api/savedata-game.md)).
- `[Debug] SaveJsonSelfCheck = false`. Runs the save JSON round trip self-check at start. It logs `save json self-check ok`, or the first failing case.
- `[Debug] JsonProbeCommand = false`. Registers the `core_jsonprobe` console command, which logs one probe value as `JsonUtility` and as `SaveData` writes it.
- `[Debug] SceneLooksCommand = false`. Registers the `scenelooks` console command, which prints the scene look index for a [scene](glossary.md#scene) (`list`, `listall`, `find`, `findchar`, `here`). It reads the installed game files and changes nothing in the game (see [Scene looks](api/scenelooks.md)). The framework's own Settings page in the Mods menu also has a `Scene looks` action, `Run scene looks checks`. It needs no flag and no console: it prints the same index for the loaded scene to the menu status line and the log.
- `[Debug] ModMenuSelfCheck = false`. Runs the classic Mods menu self-check at start and logs `modmenu self-check ok`, or the first failing case. It covers the row kind, value and default helpers, and the config list filter and sort. It also covers the action registry with its confirm flag and expiry, the navigation order and focus-keep helpers, and the action row confirm rule. It calls no Unity engine code and drives no live control, so it runs at start before the menu is built.
- `[UI] ThemeConfigManager = true`. Gives the ConfigurationManager window a dark theme (see ModMenu, below).

While the ConfigurationManager window (F1) is open, the framework freezes mouse-look and player movement with the game's own `GameController.ControlsDisabled`, and keeps the cursor visible. It does nothing when ConfigurationManager is not installed. Gameplay is not frozen while a game menu is open. The freeze is released when the window closes.

## CorePatcher

`CorePatcher` changes a game method with [Harmony](glossary.md#harmony) (the patching library BepInEx ships), under your mod's own [owner](glossary.md#owner) id. The [Patching](guides/getting-started/patching.md) guide shows the steps.

```csharp
public static class CorePatcher
{
    public static bool Patch(MethodBase target, MethodInfo prefix, MethodInfo postfix, MethodInfo finalizer, bool core, string owner);
    public static bool Patch(Type type, string name, Type[] args, MethodInfo prefix, MethodInfo postfix, MethodInfo finalizer, bool core, string owner);
    public static bool Patch(MethodBase target, MethodInfo prefix, MethodInfo postfix, MethodInfo finalizer, bool core, string owner, int priority);
    public static bool Patch(Type type, string name, Type[] args, MethodInfo prefix, MethodInfo postfix, MethodInfo finalizer, bool core, string owner, int priority);
    public static bool IsDisabled(string owner);
}
```

- `owner` must be a non-empty [mod id](glossary.md#mod-id) that does not start with `obenseuer.moddingframework` or `nomad.obenseuer.core` (any letter case). Otherwise all four overloads throw `ArgumentException` before anything else runs. Both prefixes are reserved for the framework's own patches.
- The short overloads keep the `[HarmonyPriority]`, `[HarmonyBefore]` and `[HarmonyAfter]` attributes of each method (checked with HarmonyX 2.9).
- `priority` is the HarmonyX priority (`HarmonyLib.Priority`) given to the prefix, postfix and finalizer. It overrides a `[HarmonyPriority]` attribute. `-1` leaves the attributes alone. `Priority.Normal` is 400 and `Priority.First` is 800. For prefixes, a higher value runs first.
- HarmonyX runs every prefix, even when an earlier prefix returns `false`. A `false` only skips the original method. A higher priority prefix runs before the others, but it cannot stop them.
- One Harmony instance per `owner`.
- Returns `false` and logs to `BepInEx\logs\<owner>.log` on failure.
- `core: true` marks the patch as critical. If it fails, every patch the framework made under that owner is removed and `IsDisabled(owner)` becomes true. Your own `new Harmony(owner)` patches are not touched.
- A disabled owner cannot patch again this session. `Patch` returns `false` for it without logging, until the game restarts.
- Refuses to patch (returns `false`, and logs `patch skipped, unsupported game build`) when [`Core.PatchingAllowed`](api-core/plugin.md) is false. That is only when the build is not 0.5.08 and `[Safety] RefuseUnknownBuild` is set. By default an unknown build is patched.

**Common problems**

- `Patch` returns `false`: the patch failed, the owner is disabled, or `Core.PatchingAllowed` is false. A failure is logged to `BepInEx\logs\<owner>.log`. The next bullets cover the other two causes.
- `Patch` returns `false` and logs nothing: the owner was disabled by a failed `core: true` patch. `IsDisabled(owner)` is true. Restart the game.
- `ArgumentException` from `Patch`: the owner is empty or starts with a reserved prefix.
- The log says `patch skipped, unsupported game build`: `Core.PatchingAllowed` is false. That is only when the build is not 0.5.08 and `[Safety] RefuseUnknownBuild` is set.

```csharp
using System;
using System.Reflection;
using BepInEx;
using BepInEx.Unity.Mono;
using ObenseuerCore;

public class MyMod : BaseUnityPlugin
{
    private void Awake()
    {
        MethodInfo postfix = typeof(MyMod).GetMethod("AfterDayChanged", BindingFlags.Static | BindingFlags.NonPublic);
        bool ok = CorePatcher.Patch(typeof(TimeOfDayAzure), "IncreaseDayCounter", Type.EmptyTypes, null, postfix, null, false, "example.mymod");
        if (!ok || CorePatcher.IsDisabled("example.mymod")) Logger.LogWarning("MyMod patch unavailable");
    }

    private static void AfterDayChanged() { }
}
```

## CoreLog

`CoreLog` writes your mod's own log file under `BepInEx\logs`.

```csharp
public class CoreLog
{
    public static CoreLog For(string modId);   // BepInEx\logs\<modId>.log; the previous file moves to <modId>.log.1 on start; AutoFlush; at 4 MB the file moves to .1, replacing the older .1
    public void Info(string message);
    public void Warn(string message);
    public void Error(string message);
    public void Debug(string key, string message);   // only when Debug.Verbose; writes when the text for key changes
    public int LineLimit { get; set; }   // lines per second, default 100, clamped 1 to 100000
}
```

Lines look like `[frame 1234] [56.789] INFO: message`.

Some ids are not usable file names: a path separator, `..`, `:`, a Windows device name such as `CON`, `NUL` or `COM1`, or a trailing dot or space. Such an id logs to `BepInEx\logs\ObenseuerCore.log`, with the id as a `[id] ` prefix. `For(null)` does the same, and never throws.

```csharp
using BepInEx;
using BepInEx.Unity.Mono;
using ObenseuerCore;

public class MyMod : BaseUnityPlugin
{
    private static readonly CoreLog Log = CoreLog.For("example.mymod");

    private void Awake()
    {
        Log.Info("started");
        Log.Debug("state", "idle");
    }
}
```

## Events

`Events` holds the framework's clock, [scene](glossary.md#scene) and save events. The [Using events](guides/getting-started/using-events.md) guide shows how to subscribe. Every event the framework raises is listed on [Events](events.md).

```csharp
public static class Events
{
    public static event Action<int> HourPassed;     // hour 0..23, when the game day and hour change
    public static event Action<int> DayPassed;      // day counter after the change
    public static event Action<string> SceneLoaded; // scene name, for every load, additive ones included
    public static event Action Saved;               // after the save files are written
    public static event Action Loaded;              // after a save has been loaded
}
```

- Handlers run in subscription order. An exception in one handler is logged, and the other handlers still run. The first failure of each handler for each event is logged; later failures of it are not. A handler may subscribe or unsubscribe during a dispatch; the change takes effect at the next event.
- `HourPassed` is checked on each game minute, so a time change is seen on the next minute. A change that lands on the same hour number on another day still fires.
- A level change does not repeat the hour it lands on. After it, `HourPassed` fires only when the hour or the day differs from the last one reported. A load or new game started from the main menu fires `HourPassed` on its first game minute, even for the same hour. That is because the main menu resets the last hour. `Saved` and `Loaded` also fire on every level change, because a level change is a save followed by a load.
- Subscribers that live in scenes must unsubscribe (for example in `OnDestroy`). Otherwise they are called after their objects are destroyed, and only the first failure of each handler is logged.

```csharp
using BepInEx;
using BepInEx.Unity.Mono;
using ObenseuerCore;

public class MyMod : BaseUnityPlugin
{
    private void Awake()
    {
        Events.HourPassed += hour => Logger.LogInfo("hour " + hour);
        Events.DayPassed += day => Logger.LogInfo("day " + day);
        Events.Loaded += () => Logger.LogInfo("loaded");
    }
}
```

## GameTime

`GameTime` reads the game's day, weekday, hour and minute. It only reads. To change the date or the time of day, see [World](api/world.md). To make the clock run faster or slower, see [Time control](api/timecontrol.md).

```csharp
public static class GameTime
{
    public static int Day { get; }                 // dayCounter, starts at 0
    public static System.DayOfWeek DayOfWeek { get; }
    public static int Hour { get; }                // 0..23
    public static int Minute { get; }              // 0..59
    public static bool IsPaused { get; }           // Time.timeScale == 0
}
```

This [framework mod](api/framework-mods.md) logs the new day, and after a load the hour and the hunger value.

```csharp
using ObenseuerCore;

[ModInfo("example.coretour", "Framework Tour", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class CoreTour : FrameworkMod
{
    protected override void OnLoad()
    {
        Events.DayPassed += day => Log.Info("A new day started: day " + day);
        Events.WhenReady(() => Log.Info("It is " + GameTime.Hour + ":00 and the hunger value is " + Survival.Get(Stat.Hunger)));
    }
}
```

## Survival

`Survival` reads and changes the player's needs, slows or speeds their decay, and intercepts item use, damage and breakdowns. Stats live in `PlayerStats.instance`. Mods reach them through this facade and never touch the game objects.

Game-facing paths never throw into the game. A call that needs a hook group which is missing returns its default (`0`, `false`, or no-op). `Survival.IsAvailable` is true only when every [feature group](glossary.md#feature-group) is installed. A throwing mod handler is logged and the next handler still runs. Argument validation throws to the caller, even when `IsAvailable` is false: an empty `modId` throws `ArgumentException`, and a null callback or handler throws `ArgumentNullException`.

```csharp
public enum Stat
{
    Hunger = 0, Thirst = 1, Bowel = 2, Bladder = 3, Tiredness = 4, Hygiene = 5, Mood = 6, Health = 7,
    Bleeding = 8, Alcohol = 9, High = 10, Smv = 11, Radiation = 12, SaunaHeat = 13, DuckNeed = 14,
    AlcoholAddiction = 15, MushroomAddiction = 16, SmokingAddiction = 17, GamblingAddiction = 18,
    AlcoholNeed = 19, MushroomNeed = 20, SmokingNeed = 21, GamblingNeed = 22
}

public static partial class Survival
{
    public static bool IsAvailable { get; }                // false until the Survival patches are installed; false when they were refused (`Core.PatchingAllowed` is false) or failed, or when the owner is disabled
    public static float Get(Stat stat);                    // 0 when PlayerStats.instance is null
    public static bool Set(Stat stat, float value);        // clamped to the stat range; false when PlayerStats.instance is null or a save loads or saves
    public static bool OnCrossed(string modId, Stat stat, float threshold, Action<Stat, float, bool> callback); // false when IsAvailable is false
    public static bool RemoveCrossed(string modId, Action<Stat, float, bool> callback); // true when that mod's watch was removed
    public static void RemoveAll(string modId);            // removes this mod's decay modifiers, intercepts, damage handlers and OnCrossed watches
    public static bool InterceptDamage(string modId, Action<DamageArgs> handler); // false when the damage hook group is not installed
    public static bool RemoveDamage(string modId, Action<DamageArgs> handler);    // true when that mod's handler was removed
    public static event Action<DamageTypeNames> PlayerDied;
    public static event Action PassedOut;
    public static event Action<float, DamageTypeNames> Damaged;
    public static event Action<BreakdownArgs> Breakdown;

    public static class Decay
    {
        public static bool Supports(Stat stat);
        public static bool Set(Stat stat, string modId, float multiplier, float offsetPerHour);
        public static bool Remove(Stat stat, string modId);
    }

    public static class Consumables
    {
        public static void Intercept(string modId, ConsumeHandler handler);
        public static bool Remove(string modId);           // removes this mod's intercepts; false when it had none
        public static bool Remove(string modId, ConsumeHandler handler); // removes one handler of that mod; false when it was not registered
        public static event Action<ConsumedArgs> Consumed; // after the game applied a use; not for a cancelled or refused use
    }
}

public delegate void ConsumeHandler(ItemData itemData, ItemConsumable itemConsumable, ConsumeArgs args);
public class ConsumeArgs { public bool Cancel { get; set; } }
public sealed class ConsumedArgs { public int ItemId { get; } }
public sealed class DamageArgs { public DamageTypeNames Reason { get; } public bool Silent { get; } public float Original { get; } public float Amount { get; set; } public bool Cancel { get; set; } }
public enum BreakdownKind { Suicide, SelfHarm, Mushrooms, Alcohol }
public sealed class BreakdownArgs { public BreakdownKind Kind { get; } public bool Cancel { get; set; } }
```

The enum values are fixed. New stats are appended, and existing values do not change.

The table lists each stat, the game field it reads and writes, and the range `Set` clamps to.

| Stat | Game field | Range |
|---|---|---|
| Hunger | `PlayerStatsHungerAndThirst.Hunger` | -100..100 |
| Thirst | `PlayerStatsHungerAndThirst.Thirst` | 0..100 |
| Bowel | `PlayerStatsHungerAndThirst.Bowel` | 0..100 |
| Bladder | `PlayerStatsHungerAndThirst.Bladder` | 0..100 |
| Tiredness | `PlayerStatsTiredness.currentStatus` | 0..100 |
| Hygiene | `PlayerStatsHygiene.Hygiene` | 0..100 |
| Mood | `PlayerStatsMentalHealth.currentStatus` | 0..100 |
| Health | `PlayerStatsHealth.currentStatus` | 0..100 |
| Bleeding | `PlayerStatsHealth.Bleeding` | 0..100 |
| Alcohol | `PlayerStatsAlcohol.AlcoholInBlood` | 0..100 (the game does not clamp above 100) |
| High | `PlayerStatsMushrooms.MushroomsHigh` | 0..100 |
| Smv | `PlayerStatsMushrooms.SMVProgression` | 0..100 |
| Radiation | `PlayerStatsRadiation.Dose` | 0 and up |
| SaunaHeat | `PlayerStatsSauna.Overheat` | 0 and up |
| DuckNeed | `PlayerStatsDuck.currentStatus` | 0..100 |
| AlcoholAddiction, MushroomAddiction, SmokingAddiction, GamblingAddiction | `PlayerStatsAddictions.<name>` | 0..100 |
| AlcoholNeed, MushroomNeed, SmokingNeed, GamblingNeed | `PlayerStatsAddictions.<name>` | 0..100 |

Reading, writing and watching a stat:

- `Get` returns the raw game value. `Set` returns `false` when the value is NaN, `PlayerStats.instance` is null, or a save loads or saves.
- `OnCrossed` fires `callback(stat, value, rising)` when the value moves from below to at-or-above `threshold` (rising), or from at-or-above to below it (falling). The value is sampled once per game second. Stats change only while the player is active, so no crossing fires while frozen. The first sample after a load sets the baseline without firing.
- `RemoveCrossed(modId, callback)` removes the watch that `OnCrossed` added, and returns `true` when one was removed.

Decay rates:

- `Decay.Set` uses one entry per `(stat, modId)`. The effective rate for a stat is `base × product(multipliers) + sum(offsetPerHour)`, where `base` is the game's rate for that stat. A second `Set` from the same mod replaces the first. Multipliers must be `>= 0`. Otherwise `Set` returns `false`.
- For `AlcoholNeed`, `MushroomNeed`, `SmokingNeed` and `GamblingNeed` the effective rate is never below 0. A negative result is clamped to 0, so an offset cannot make a need rise. The game lowers these needs with no upper clamp when the player is not addicted.
- For the other stats a negative total rate is allowed and makes the stat move the other way. Hunger and Thirst rise at a positive base rate, so a negative total lowers them. Hygiene falls at a negative base rate, so a positive total makes it rise. Offsets may be negative, so the total can be negative for those stats.
- The game scales the patched rate afterwards, so an offset is not an absolute amount per hour. Awake hunger is multiplied by `(1 - High / 100)` and `(1 - AlcoholAddiction / 200)`. Sleeping hunger and thirst are multiplied by 0.33. Sleeping hygiene is multiplied by 0.5.
- The four addiction need getters drive both the rise and the fall. The rise is `Addiction × 0.01 × rate / 60` (the same form applies to the other needs). The fall is `rate / 60`. A modifier on an addiction need changes both.
- `Decay.Supports` returns `true` only for: `Hunger`, `Thirst`, `Hygiene`, `AlcoholNeed`, `MushroomNeed`, `SmokingNeed`, `GamblingNeed`. These are the stats with a per-hour rate getter that the game uses to change the stat.

Item use ([interceptors](glossary.md#interceptor) and the `Consumed` event):

- `Consumables.Intercept` runs once per consumable use, before the game does anything for that use. The framework dispatches at the use entry point: `ItemInfo.UseItem(int, bool)` for inventory use, including a drink from a liquid container, and `CollectibleItem.InvokeActions()` for world use. `ItemConsumable.Invoke(ItemData, bool)` is the fallback for other callers.
- If any handler sets `args.Cancel = true`, the whole use is skipped. Nothing else happens for that use. There is no cooldown, no sound, no consume events, no meta change, no stat change and no liquid change. There are no outputs, no item removal or ammo change, no `RequiredItems` change and no `ItemsToSpawn`.
- A use the game would refuse never reaches the handlers. That is: no use button, cooldown, active trade, `CanUse` false, or an owner permission that denies use.
- Handlers run once per use and share one `ConsumeArgs` per use. A handler that throws is logged, and the other handlers still run. The `ItemConsumable` is shared by the game and is never changed by the framework.
- `Consumables.Remove(modId)` removes that mod's intercepts, and `Remove(modId, handler)` removes one handler of that mod. `RemoveAll` calls the first.
- `Consumables.Consumed(ConsumedArgs)` is for observing only. It fires once for each `ItemConsumable.Invoke` the game runs. By then the game has applied the effects, removed or used up the item and spawned its `ItemsToSpawn`. `ItemId` is the item's `Item.ID`.
- `Consumed` does not fire when an intercept cancels the use, or when `Invoke` throws. It does not fire when the game refuses the use before `Invoke` (the list under `Intercept`). It does not fire when `Survival.IsAvailable` is false.
- A use that runs more than one consumable `Invoke` fires `Consumed` once per `Invoke`: `ItemInfo.UseItem` runs the child item's actions and then the item's own action, and `Item.InvokeActions` runs each action of an item. Depression-forced uses fire it too. Handlers are isolated from each other. Unsubscribe with `-=`; `RemoveAll` does not remove them. The payload has no stat deltas.

Damage, passing out and breakdowns:

- `InterceptDamage(modId, handler)` runs before the game applies health damage, in the order the handlers were registered. The handler gets a `DamageArgs` holding the amount the game asked for (`Original`), the working amount (`Amount`), the `Reason` and `Silent`. It can change `Amount` or set `Cancel = true`. A `Cancel` from any handler wins, and the later handlers still run and see it. Handlers do not run while a save loads or saves, for an amount of 0 or less, or from inside another damage handler. `RemoveDamage(modId, handler)` removes one handler of that mod, and `RemoveAll` removes them all.
- `Damaged(lost, reason)` fires when `PlayerStatsHealth.Damage` lowers `currentStatus`. `lost` is the amount actually removed, after the clamp at 0. Bleed damage raises `Bleeding` and not health, so it does not fire. Mushroom and gas protection return before the health change, so they do not fire.
- `PassedOut` fires when `WaitingController.PassOut(..., Type.Passout)` changes `waitingType` to `Passout`. It does not fire for a refused call, or for a call that starts normal sleep (`Sleep`, which sets `Sleeping`). It does not fire for `Type.Arrest`, or for a call made while the player is already passed out.
- `Breakdown` fires when the game starts a breakdown step, with a `BreakdownArgs` whose `Kind` is `Suicide`, `SelfHarm`, `Mushrooms` or `Alcohol`. Set `Cancel = true` to stop that step. A `Cancel` from any handler wins, and the later handlers still run and see it. Handlers do not run while a save loads or saves, and the step goes ahead.

Open questions:
- The cancel path at the entry points (`ItemInfo.UseItem`, `CollectibleItem.InvokeActions`).

Not supported:
- `DuckNeed` decay. Its base-rate getter only sets the level when the player has enough ducks, so it is not a rate.
- Decay for the addiction levels (`AlcoholAddiction` etc.), `Bowel`, `Bladder`, `Health`, `Mood`, `Tiredness`, `Radiation` and `SaunaHeat`. Hunger decay also scales `Bowel` build-up, and thirst decay scales `Bladder`.
- Scaling of item effects. `Intercept` can cancel an item use. It cannot change an effect's size.
- A HUD for custom stats.
- `PlayerDied`, `PassedOut` and `Damaged` are plain C# events, so `RemoveAll` cannot remove their handlers. Unsubscribe with `-=`.
- A depression-forced use calls `ItemConsumable.Invoke` directly, so only the `Invoke` fallback applies. A cancel skips the item's effects and its removal, and the item stays in the inventory. `ItemInfo.Activate` still runs. `Depression` still returns `true` after a cancel, so the game's mental health code skips its fallback. The game treats the use as handled.

This plugin halves hunger decay, logs when hunger crosses 80, and logs the reason when the player dies.

```csharp
using System;
using BepInEx;
using BepInEx.Unity.Mono;
using ObenseuerCore;

public class SlowHunger : BaseUnityPlugin
{
    private static readonly CoreLog Log = CoreLog.For("SlowHunger");

    private void Awake()
    {
        Survival.Decay.Set(Stat.Hunger, "example.slowhunger", 0.5f, 0f);
        Survival.OnCrossed("example.slowhunger", Stat.Hunger, 80f, (stat, value, rising) =>
            Log.Info(stat + " crossed 80, rising " + rising + ", value " + value));
        Survival.PlayerDied += reason => Log.Info("player died, reason " + reason);
    }
}
```

The example is `Examples\SlowHunger\`.

## CameraView

`CameraView` switches between first and third person, driven through the game's own `ThirdPersonCameraCollision` state. Nothing moves a camera by hand.

```csharp
public static class CameraView
public static bool IsAvailable { get; }          // a ThirdPersonCameraCollision exists (false on the main menu)
public static bool IsFirstPerson { get; }        // the game's firstperson flag
public static bool ThirdPersonAllowed { get; }   // false with no camera, while a PlayerCameraTrigger forces first person, or while disabledZoomList is not empty
public static bool SetThirdPerson(float distance = 2.5f);  // false when not allowed. Sets currentDistance, clamped to minDistance..maxDistance
public static void SetFirstPerson();             // currentDistance = 0 and FirstPerson() now
public static bool Toggle();                     // third person when first, first person otherwise. false when not available or refused
public static event Action<bool> ModeChanged;    // true = first person. Forwarded from ThirdPersonCameraCollision.CameraModeChange
```

- Vanilla 0.5.08 is first person only. Third person is a dormant game mode, so the player body, animations and clipping may look rough.
- `SetThirdPerson` clears `disableThirdPerson` and `firstpersonOnly` and keeps the vanilla values, so `Tweaks.Camera` can put them back. The game's `FixedUpdate` then calls `ThirdPerson()` on the next physics step. `ModeChanged` fires then, not inside the call.
- A wall closer than about 0.6 m keeps first person, because the game's collision sets `distance` below 0.5.
- `SetFirstPerson` does not change the camera rotation. The controller's `rotX`/`rotY` stay as they are, so the view direction does not jump.
- A forced zone is detected by a postfix on `PlayerCameraTrigger.FirstPersonOnly(bool)`. A trigger counts as forcing from its `FirstPersonOnly(true)` call until its `FirstPersonOnly(false)` call, or until it is destroyed. A trigger whose [GameObject](glossary.md#gameobject) is only disabled, with no `false` call, stays counted. No game code calls `PlayerCameraTrigger.FirstPersonOnly`, so how a zone is entered and left is set in the [scene](glossary.md#scene) and not verified.
- If the postfix cannot be patched, `ThirdPersonAllowed` stays false. Third person is then refused, never forced.
- `firstpersonOnly` alone is not a forced zone. `ThirdPersonCameraCollision.Start` sets it from `disableThirdPerson` for the whole game.
- Save: the game saves `firstperson`, `currentDistance`, `disableThirdPerson` and `firstpersonOnly` (the public, non-`fsIgnore` fields in game `SaveController`). A save made in third person stores the cleared flags, so loading it restores them.
- A new `ThirdPersonCameraCollision` runs `Start` again, which sets the first-person lock again. While `ThirdPersonToggle` or `AllowScrollZoom` is on, the lock is cleared again on the next frame. Not verified that `Start` runs on every scene load.
- Null-safe: with no camera (main menu) the properties return false and the methods return false or do nothing.
- Hand items: `PlayerHandItems.CameraModeChange` is subscribed to the same static event, and `FirstPerson()`/`ThirdPerson()` fire it. Going through them keeps the held items right.

## Tweaks

The built-in gameplay tweaks change needs, stacks, spoilage, crafting, renovations, the camera and the clock without any code. They have no C# API. Settings live in `BepInEx\config\obenseuer.moddingframework.cfg`, one section per tweak, and in the in-game config window (BepInEx ConfigurationManager, F1). The [Config](guides/getting-started/config.md) guide explains config files.

- Every tweak is off by default: multipliers are `1`, bools are `false`.
- Changes apply live from the in-game config window. A tweak's patches are installed the first time its value becomes non-default, and stay installed. At the default, a patch does nothing and returns the game value unchanged.
- Multipliers are clamped to `0.01` to `100`. NaN becomes `1`. Each clamp is logged to `BepInEx\logs\obenseuer.moddingframework.tweaks.log`.
- Each change is logged once as `tweak <Section>.<Key>: <old> -> <new>`. A change is logged after its value has been still for 0.5 s, so a slider drag logs one line. A patch install logs `tweak <Section> patched` or `tweak <Section> patch not applied`.
- Patches are owned by `obenseuer.moddingframework.tweaks`. A failed tweak logs and does nothing else.

| Section | Key | Default | What it does | Save caveat |
|---|---|---|---|---|
| `Tweaks.Needs` | `HungerDrainMultiplier`, `ThirstDrainMultiplier`, `HygieneDrainMultiplier`, `AlcoholNeedDrainMultiplier`, `MushroomNeedDrainMultiplier`, `SmokingNeedDrainMultiplier`, `GamblingNeedDrainMultiplier` | `1` | <details>{::nomarkdown}<summary>{:/nomarkdown}Scales how fast the stat drains, through `Survival.Decay`.{::nomarkdown}</summary>{:/nomarkdown}The multiplier is set or removed live, with no patch. For the four addiction needs it scales both the rise and the fall.</details> | Not saved. |
| `Tweaks.Stacks` | `StackMultiplier` | `1` | Scales the stack size of items that stack (above 1). Liquid capacity per slot scales too. | Stacks over the limit stay over it when the value is lowered or set back to `1`, until that slot is filled again. |
| `Tweaks.Spoilage` | `SpoilageMultiplier` | `1` | Scales how fast items spoil. | Decay already applied is saved permanently. |
| `Tweaks.CraftingSpeed` | `CraftingSpeed` | `1` | <details>{::nomarkdown}<summary>{:/nomarkdown}Scales manufacturing craft time and blueprint craft time. `2` is twice as fast.{::nomarkdown}</summary>{:/nomarkdown}Below `1` the blueprint wait speed-up is slower than its target: the craft ends at the scaled length, but the speed-up uses the unscaled time.</details> | <details>{::nomarkdown}<summary>{:/nomarkdown}A running craft keeps its scaled length after the value changes.{::nomarkdown}</summary>{:/nomarkdown}Manufacturing crafts are checked against the current length, so raising the value mid-craft can finish it at once, lowering it can make its progress jump, and changing it between sessions does the same.</details> |
| `Tweaks.DishwasherSpeed` | `DishwasherSpeed` | `1` | Scales dishwasher recipe process time, for recipes generated from now on. `2` is twice as fast. Stacks with `CraftingSpeed`. | Not saved. |
| `Tweaks.NoMinimumSkill` | `NoMinimumSkill` | `false` | <details>{::nomarkdown}<summary>{:/nomarkdown}Lifts the minimum skill gate for manufacturing and removes the low-skill craft time penalty.{::nomarkdown}</summary>{:/nomarkdown}Manufacturing only. The gate includes `HardMinimumSkill`. Recipes whose craft roll always fails stay disabled. Skill still affects fail chance and quality. Blueprints are not covered. Turning it off while a station is open takes effect only when the recipe list refreshes; the craft path has no skill check of its own.</details> | Not saved. |
| `Tweaks.Renovations` | `CostMultiplier` | `1` | Scales the [OC](glossary.md#oc) first payment and the [RM](glossary.md#rm) final payment. Rounded to whole units. | Not saved. Orders already placed keep their saved progress. |
| `Tweaks.Renovations` | `MoneyOnly` | `false` | <details>{::nomarkdown}<summary>{:/nomarkdown}Renovations need only money; the item requirement is skipped.{::nomarkdown}</summary>{:/nomarkdown}The item requirement is skipped for the upgrade check and removal. A mod that takes renovation items itself does not read this flag: its prefix on `TenementController.RemoveRequiredItems` still runs, because HarmonyX runs every prefix, and it may still take items. See `TweakState`.</details> | Not saved. |
| `Tweaks.Renovations` | `BuildTimeMultiplier` | `1` | Scales the build time when an order is placed, and the displayed total. Rounded up to whole hours for the total. | Not saved. Changing it while an order is placed can leave that order's progress wrong. |
| `Tweaks.Renovations` | `InstantBuild` | `false` | Places renovation orders with zero build time. Scripted `TenementUpgradeStart` orders keep their authored time. | Not saved. |
| `Tweaks.Camera` | `ThirdPersonToggle` | `false` | <details>{::nomarkdown}<summary>{:/nomarkdown}Turns on the `ToggleKey` switch between first and third person.{::nomarkdown}</summary>{:/nomarkdown}Off clears the toggle's lock and goes back to first person when the game's first-person lock was on.</details> | Not saved by the framework. The game saves `firstperson` and the cleared flags, see `CameraView`. |
| `Tweaks.Camera` | `ToggleKey` | `F6` | <details>{::nomarkdown}<summary>{:/nomarkdown}The key that switches the view while `ThirdPersonToggle` is on.{::nomarkdown}</summary>{:/nomarkdown}Ignored while a game menu, shop, pause, dialogue, the console or game controls are active. `V` is bound to "Rotation Mode", so it is not the default.</details> | Not saved. |
| `Tweaks.Camera` | `AllowScrollZoom` | `false` | <details>{::nomarkdown}<summary>{:/nomarkdown}Clears the first-person lock so the scroll wheel moves between first and third person.{::nomarkdown}</summary>{:/nomarkdown}Restored to the game's values when it and `ThirdPersonToggle` are both off. Not applied while a trigger forces first person.</details> | Not saved by the framework. A save made while it is on stores the cleared flags. |
| `Tweaks.Time` | `TimeScale` | `1` | <details>{::nomarkdown}<summary>{:/nomarkdown}Makes the game clock run faster; `2` is twice as fast.{::nomarkdown}</summary>{:/nomarkdown}Need drain, spoilage, craft timers and renovation timers all run on game time, so they speed up too. If Mono inlines `AzureTimeController.SetNewDayLength`, this does nothing. Check that the clock speeds up after setting it.</details> | Not saved. The saved time of day is unchanged. |

The NPC route timing and the blueprint wait speed-up still use the unscaled day length.

## TweakState

`TweakState` exposes the one tweak value a mod has to read for itself.

```csharp
public static class TweakState
public static bool RenovationsMoneyOnly { get; }  // Tweaks.Renovations MoneyOnly, the value the game sees
```

A mod that takes renovation items itself must check this in its own `TenementController.RemoveRequiredItems` prefix. When it is true, return `false` early, before taking anything. HarmonyX still runs every prefix, so the check belongs in that mod's code.

Read the flag from a helper, and call the helper from the prefix inside try/catch. An older framework version without the member fails when the code that reads it is compiled. That is a `TypeLoadException` or a `MissingMethodException`, and it is thrown at the call into the helper. A try/catch inside the helper does not catch it (checked with a two-version probe on .NET Framework).

```csharp
private static bool MoneyOnlyOn()
{
    return ObenseuerCore.TweakState.RenovationsMoneyOnly;
}

private static bool MyPrefix()
{
    try
    {
        if (MoneyOnlyOn()) return false;
    }
    catch (Exception)
    {
    }
    return true;
}
```

## ModMenu

`ModMenu` adds a page of your own to the Mods menu, the window the player opens from the pause menu or the main menu.

```csharp
public static class ModMenu
{
    public static void AddPage(string modId, string title, Action<RectTransform> build);
    public static ModMenuWidgets WidgetsFor(RectTransform content);  // null when content is not a page's build target
}
```

`ModMenuWidgets` (what `WidgetsFor` returns) is obsolete (`[Obsolete("Use ObenseuerCore.Ui.ModUi")]`). It still works, but new pages should build with `ModUi` from the [UI kit](api/ui.md). A mod that uses it gets compiler warning CS0618.

The Mods menu opens as a web page by default (`[UI] ModMenuStyle = web`). Each [plugin](glossary.md#plugin) with visible config entries gets one "Settings" page. Classic `AddPage` pages are added under their [mod id](glossary.md#mod-id). When no plugin has that mod id, the page becomes a new entry named after its title. The classic window opens when `[UI] ModMenuStyle` is `classic`, or when the web page fails to start. If the page does not say hello within 8 s twice in a row, the menu stays classic until the game restarts.

The [framework](glossary.md#framework) itself gets a Settings page like any plugin, and it is the one page that also carries actions. The framework's page has a `Scene looks` section with a `Run scene looks checks` button. Clicking it runs the check for the loaded [scene](glossary.md#scene) and shows the result on the menu status line. The classic window shows the same button under the framework's group, with the result next to it. The action needs no `[Debug]` flag.

The classic Mods menu is one `ModWindow` in the Full dock. Full is used because the Center dock (640 px) cannot hold the sidebar and the settings rows. The window's `Side` column is a sidebar with one button per page, and `Body` shows the selected page. The sidebar lists "Mod Settings" first (the framework's config editor), then every added page sorted by title. `AddPage` adds a sidebar entry only. It never adds a game tab. The sidebar is read when the window opens, so a page added while the window is open appears the next time it opens.

`ModMenu.AddPage` pages are classic only. The page's `build` callback runs in the classic uGUI window, whether the player opens the page from the classic sidebar or from the web menu. The web menu opens the classic pane for a page it does not render itself, with a `Back to Mods` button. A page never renders as HTML. A mod that needs a rich control, such as a searchable dropdown, a colour picker or multi-line text, builds that in the [WebUi](api/webui.md) menu instead. It keeps the classic page to labels, buttons and the `ModUi` kit.

Entry points:

- In game: the Mods tab is one entry in `UIPageButtons.MenuPages`. Its button opens the window, and arriving at the tab opens it too. Closing the window while the tab is selected returns the pause menu to the page it came from. The tab's id is the first unused `MenuPageNames` value above `MenuPageNames.Items`, checked against `MenuPages`, so two mods never get the same id.
- Main menu: a "MODS" button sits under the main menu grid, in the row below "???". It opens the same window.
- The tab is built once `UIPageButtons` and `PauseMenu` exist, and again when the game recreates them. A menu whose tab failed to build is not retried for that menu instance. There is no runtime removal.
- The game's bumper cycling uses a fixed list of six names (`_tabs`), and the framework does not change that list. Mouse and click reach the tab.

Keys: Esc closes the window (the game's Pause Menu key), and so does Joystick1Button1, unless a text field or key row is capturing. While a text field has focus, or a key row is capturing, that field or row uses the Esc press. The window does not close. The web menu registers a back action for its classic pane. When a back action is registered, Esc and Joystick1Button1 run that action first and the window stays open.

Controller and keyboard navigation: the sidebar tabs, the search box, every control of every row and the footer buttons form one ordered focus list in visual order. The list is wired into `Selectable.navigation` in Explicit mode. The game's own input module keeps the left stick and the arrow keys. Up and down move along the current column, and both wrap at the ends. Left and right step a focused slider or stepper, and otherwise move between columns. The framework adds the d-pad (the `DPadHorizontal` and `DPadVertical` axes that the game's own menus read). It adds the bumpers (`Switch Tab Left` and `Switch Tab Right`) for the sidebar pages. It also adds Enter or A on a focused slider, which steps it by 5% of its range. A focused button, toggle or text field is activated by the game's Submit key. The footer shows these keys, with the A and B sprites from `InputManager.inputImages` when the player uses a controller.

Pages:

- A page's `build` receives the page's content area. It runs when the page is selected, so the rows are built once per page open. The content is cleared before each build.
- If `build` throws, the page shows the notice "This page failed to load. Details are in BepInEx\logs\obenseuer.moddingframework.ui.log", the error is logged there, and the window stays open. The other pages still work.
- Controls from `ModUi` run their callbacks under `ModWindow.Run`. A callback that throws closes the window with reason `Fault`. This is the kit's rule for every window, not a page rule.

**Common problems**

- The page shows "This page failed to load": your `build` callback threw. Read `BepInEx\logs\obenseuer.moddingframework.ui.log`.
- The window closes with reason `Fault`: a callback of a `ModUi` control threw.
- The menu opens as the classic window although `[UI] ModMenuStyle` is `web`: the web page failed to start. After two misses of the 8 s hello, it stays classic until the game restarts.

Mod Settings:

- Lists every ConfigEntry of the loaded plugins, grouped by plugin (the first group starts expanded, the others collapsed) and then by section. Rows are sorted by section and key.
- A search box at the top filters the rows by plugin, section or key. While a search is active, every group is open.
- Row controls: bool is a toggle (`ModUi.Toggle`). An enum, and a list of acceptable values (`AcceptableValueList<T>`), is a stepper (`ModUi.Stepper`). A `[Flags]` enum is a text field. A KeyboardShortcut is a key binding (`ModUi.KeyBinding`): click it, then press a key. Ctrl, Shift and Alt held at the press are kept. Esc cancels, and mouse buttons are ignored. A ranged int or float is a slider (`ModUi.Slider`). A string, and any other type that converts from text, is a text field (`ModUi.TextField`). Other types are read-only. A row shows its description under the label, and the same text is still the hover tooltip. A row whose value differs from its default shows an accent dot and a Reset button; a row at its default shows neither. Entries that ConfigurationManager marks `Browsable = false` are not listed.
- Saving: toggles, steppers, key bindings and text fields write the ConfigEntry and save at once. A text field writes when its edit ends (Enter, or a click away), not on each key. Sliders write at once and save 0.25 s after the last change. Quitting to the desktop from the pause menu or the main menu saves a change that is still waiting.

Lock and inventory:

- While a text field has focus, or a key row is capturing, the game's controls and keys are locked and its controller update is skipped. The lock is released when the field loses focus or the capture ends.
- A custom page does not reach the inventory: while one is selected, the inventory's tab switch and its per-frame input (Drop, Move Item, Use) do nothing.

Open questions:

- The in-game Mods tab: the window opens on arrival, and closing it returns to the previous page. The pause menu does not react to keys while the window is open.
- The main-menu MODS button: placement under the grid, the click, left and right navigation to Multiplayer, and the navigation restore on close.
- Esc and Joystick1Button1 closing, and Esc inside a text field and inside a key capture. That the Pause Menu does not open on the same Esc press.
- Search, grouping, every row control, the 0.25 s save, and Reset.
- The page failure notice, and a throwing callback closing the window with reason Fault.

Config `[UI] ThemeConfigManager = true` gives the ConfigurationManager (F1) window a dark theme. It takes effect on the next start. The framework patches the private `OnGUI` of ConfigurationManager through `CorePatcher`: a prefix swaps `GUI.skin` for a dark skin built from the menu palette, and a finalizer restores the previous skin. Without ConfigurationManager, nothing is patched.

## MainMenuButtons

- `MainMenuButtons.Add(string id, string label, Action onClick)` adds a button to the main menu row under the grid, next to MODS. `MainMenuButtons.SetControlsEnabled(bool enabled)` and `MainMenuButtons.IsOpen` turn the menu's own controls off and on. Buttons from other mods in that row are kept clear. Details: [Main menu buttons](api/mainmenu.md).

## ComputerApps

`ComputerApps` puts your mod's apps on the in-game computer desks and opens one as a full-screen window. It also guards typing in app inputs and adds use prompts to world objects. The desk patches install on the first `Register` call, not at start-up. `IsAvailable` is true when [`Core.PatchingAllowed`](api-core/plugin.md) is true and the owner has not been disabled.

```csharp
public static class ComputerApps
{
    public static bool IsAvailable { get; }
    public static event Action<ComputerAppClosedArgs> AppClosed;
    public static event Action<RectTransform> ScreenChanged;
    public static RectTransform ActiveScreen { get; }
    public static bool Register(string modId, string appId, string title, Sprite icon, Action<RectTransform> open);
    public static void Close(string appId);
    public static void RemoveAll(string modId);
    public static bool OpenStandalone(string modId, string appId);
    public static bool AddInteraction(GameObject target, string modId, string title, string message, Action onUse);
}

public sealed class ComputerAppClosedArgs
{
    public string ModId { get; }
    public string AppId { get; }
}
```

Desk apps:

- `Register` adds an icon to every desk, both the desks that exist now and the ones created later. `title` must not be empty. `icon` (a [Sprite](glossary.md#sprite-and-texture2d)) may be null, and then the title is drawn on the tile.
- `Register` returns `false` when the desk patches cannot be installed (`CorePatcher` refuses patches when `Core.PatchingAllowed` is false), or when `appId` is already registered. The first registration wins, and a warning is logged for the second. It throws `ArgumentException` for a null or empty [mod id](glossary.md#mod-id), and `ArgumentNullException` for a null `open`. An empty `appId`, an `appId` that contains `/`, or an empty `title` returns `false` and logs once.
- Icons form a grid at the top right of each desk: 84 x 72 tiles, 5 rows, columns 96 px apart, rows 84 px apart, 12 px margin. An icon is created once per desk and app, and it is reused after that.
- A click calls `open(host)`. `host` is a `RectTransform` that fills the desk's wallpaper, one per desk and app. The framework clears the host before every `open` call: each child is renamed, deactivated and destroyed. So `open` always builds your UI into an empty host, the same as `OpenStandalone`. Anything kept in the host's children is gone on the next click, so keep your state in your mod.
- An exception in `open` is logged to the app's mod log, and the host is hidden.
- The host must sit under a `GraphicRaycaster`, or its widgets receive no clicks. The desk logs one warning when none is found above the host. Each time an app is shown, the framework makes sure an EventSystem exists.
- Apps hide when the player leaves the computer. `Close(appId)` hides the app on every desk, and it closes the full-screen window if that window shows the app. `Close` always runs, also while a save loads or saves. Each close raises `AppClosed` (below).
- `AppClosed` (`ComputerAppClosedArgs`: `ModId` is the mod that registered the app, `AppId` is its id) is raised once for each view that closes. A desk host raises it when it was showing and something hides it: `Close(appId)`, the leave-the-computer hide (`HideAll`), a single [scene](glossary.md#scene) load (not an additive one), `RemoveAll` for its mod, or an `open` that throws. A host that was already hidden does not raise it.
- The full-screen window raises `AppClosed` when it closes: its Close button, Esc, a single scene load, `RemoveAll` for its mod, or `OpenStandalone` of another app. The event fires for both the desk host and the full-screen window. The args carry only `ModId` and `AppId`, so a handler cannot tell which view closed. A handler that must tell them apart has to track its own view. Handlers are isolated from each other, and unsubscribe with `-=`.
- While an input field inside an app host has focus, the desk's own key handling is skipped, so typing does not leave the computer.
- `ActiveScreen` is the desk `RectTransform` (the monitor's wallpaper) of the computer the player is using now, or null. It is set when the player starts using a computer (`InteractableComputer.Use`). It is cleared when the use stops (`StopUse`) or a single scene load clears it. `ScreenChanged` is raised with the new value on each change, not on every frame. A `WebView` can follow it with `PinTo` (see [WebUi](api/webui.md)).

**Common problems**

- `Register` returns `false`: the desk patches cannot be installed (`Core.PatchingAllowed` is false), `appId` is already registered, `appId` is empty or contains `/`, or `title` is empty.
- The app's widgets receive no clicks on the desk: the host has no `GraphicRaycaster` above it. The desk logs one warning.
- Your UI is gone the next time the icon is clicked: the host is cleared before every `open`. Keep state in your mod, not in the host's children.

Full-screen window:

- `OpenStandalone` opens the app over the game. It uses a ScreenSpaceOverlay canvas at sort order 500, a CanvasScaler at 1920x1080 (match 0.5), a GraphicRaycaster, and a Close button at the top right. The game's controls are locked with `ControlsDisabled`, and the cursor is shown. It changes no saved state, so it does not check the save gate.
- The window creates an EventSystem with a StandaloneInputModule when the scene has none. The last window to close destroys it, if the framework created it.
- It returns `false` and logs a warning when the app is unknown, or when the game is not ready or is busy. Busy means a menu, shop, pause, dialogue or the console is open. It returns `true` when that app is already open.
- The window closes on Esc, on its Close button, on a single scene load (not an additive one), and when the window throws. Esc does not close it while an input field in it has focus or a key row is capturing: that Esc belongs to the field or row, and the desk computer and the Pause Menu ignore it too. Otherwise Esc closes the window and does not open the Pause Menu on the same press. The close runs after the game's `Update`, while the controls lock still holds.

World objects:

- `AddInteraction` gives `target` (a [GameObject](glossary.md#gameobject)) the game's use prompt. It returns `false` while a save loads or saves. `target` needs a `Collider`, and the game's `InteractObjects` must exist.
- It returns `false` when patching is not allowed (`Core.PatchingAllowed`), or when there is no Collider or interact system. It returns `false` when no usable layer is found, or when another mod owns the object. It also returns `false` when the object already has a game `IInteractable` MonoBehaviour (any MonoBehaviour that implements `IInteractable`, except the framework's own prompt). The game would show its own prompt in that case, so the mod's prompt would never appear. A warning is logged once.
- Calling it again from the same mod updates `title`, `message` and `onUse`. The prompt shows `title` and `(message)`, the way the game shows its own prompts. `onUse` runs on use. An exception in `onUse` is logged to the mod's log.
- The object keeps its layer when that layer is usable and in `InteractObjects.MaskList`. Otherwise the first usable layer of 17, 23, 29, 20 and 15 is used. `RemoveAll` puts the original layer back.
- `RemoveAll(modId)` removes that mod's apps and icons, closes its window, and removes its interactions.

This plugin registers a Notes app and puts a "Read" prompt on a whiteboard that opens it full screen.

```csharp
using BepInEx;
using BepInEx.Unity.Mono;
using UnityEngine;
using ObenseuerCore;

public class NotesApp : BaseUnityPlugin
{
    private void Awake()
    {
        ComputerApps.Register("example.notesapp", "notes", "Notes", null, host => BuildNotes(host));
    }

    private void Start()
    {
        GameObject board = GameObject.Find("Whiteboard");
        if (board != null) ComputerApps.AddInteraction(board, "example.notesapp", "Whiteboard", "Read", () => ComputerApps.OpenStandalone("example.notesapp", "notes"));
    }

    private void BuildNotes(RectTransform host)
    {
    }
}
```

Open questions:

- Icon placement and draw order on a real desk, and that clicks reach the icons. The desk raycaster warning should not appear there.
- Clicks in the full-screen window, both with a scene EventSystem and without one (the framework creates it).
- Whether the game's `MaskList` includes the chosen layers at runtime. The check is the same bit test that the raycast uses.
- Esc is also the Pause Menu key. The window's close runs in `LateUpdate`, so the game's `Update` still sees the lock and returns before its Pause Menu check.
- Movement and cursor state after the window closes while the game UI is open.
- `AddInteraction` on an object that already has another `IInteractable` returns `false`.

## PlayerLook

`PlayerLook` reads and sets the player's look as plain data, through the game's own path for applying it.
```csharp
public static class PlayerLook
{
    public static event Action<Look> Changed;
    public static bool IsAvailable { get; }
    public static Look Capture();
    public static Look Capture(Appearance appearance);
    public static bool Apply(Look look);
    public static bool Apply(Look look, Appearance appearance);
    public static bool Live(Look look);
    public static bool Live(Look look, Appearance appearance);
    public static Look Randomise();
    public static Look Randomise(Appearance appearance);
    public static Look Vanilla();
    public static IList<string> Options(string slot, bool male);
    public static IList<string> Options(string slot, bool male, Appearance appearance);

    [Serializable]
    public sealed class Look
    {
        public bool set;
        public bool male;
        public bool advanced;
        public float height = 0.8f;
        // Public fields, see the list below. Dials are floats, and the wearables are strings.
    }
}
```

`Look` fields, all public: `set`, `male`, `advanced`, `height` (0.8), `shape`, `gender`, `headSize` (1), `faceSize` (1), `neckLength`, `shoulderWidth`, `breastSize`, `hipWidth`, `handSize` (1), `footSize` (1), `lipThickness`, `jawPosition`, `chinPosition`, `eyeSize` (1), `eyeDistance`, `eyePosition`, `eyeDepth`, `earSize` (1), `earRotation`, `noseRootRotation`, `noseTipRotation`, `noseWidth`. The floats default to 0 unless shown. The strings are `head`, `hat`, `hair`, `glasses`, `beard`, `shirtT`, `shirtB`, `pantsT`, `pantsB`, `socks` and `shoes`. They hold wearable asset names, and `""` means none.

- `Capture()` returns the player's look, or `null` when `IsAvailable` is false. `Capture(appearance)` reads any Appearance. A look from `Capture` has `set` true.
- `Apply(look)` sets the player's look. Each dial is clamped to its range: Height 0 to 1, Shape -1 to 1, and every other dial 0 to 1. A new costume is built at runtime, so the game's shared Random NPC asset is never written. The game's `LoadCharacter` then rebuilds the body and clothes, and `Changed` fires. It returns `false` when not available, or when the game throws (the error is logged).
- Wearables are matched by name within the list for the look's gender. A name that is not found clears the slot, except Head, which falls back to the first head of the gender. A female beard is matched by name in the same full list as a male one. The game's editor offers every beard to either gender. Leggings are matched by name.
- `Apply(look, appearance)` does the same for any Appearance, without `Changed`.
- `Live(look, appearance)`, `Randomise(appearance)` and `Options(slot, male, appearance)` are the player versions for any Appearance, and they do not fire `Changed`. `Live` and `Randomise` return `false` or `null` when the Appearance is not ready, and `Options` returns an empty list then.
- `Live(look)` sets the dial values (body and face) on the player and calls `UpdateShapes`. Clothes do not change. The clamps apply, and `Changed` fires.
- `Randomise()` captures the current look, rolls every dial except Height, and rolls each wearable. Leggings and a female beard take their first entry, as the game's Random NPC rule does. It applies the result, fires `Changed`, and returns the new look.
- `Vanilla()` returns a new look made with the game's own random NPC rules (`Appearance.LoadCharacterValues`). Male and advanced come from a coin toss. Breast size is 0 to 0.3 for male and 0.3 to 1 for female. Gender is rounded to 0 or 1, and Height is included. It does not apply the look.
- `Options(slot, male)` lists the wearable names of a slot for that gender, in the game's order. The slots are `Head`, `Hat`, `Hair`, `Glasses`, `Beard`, `Jacket`, `Shirt`, `Pants`, `Leggings`, `Socks` and `Shoes`. A female beard gives the same full list as a male one. An unknown slot throws `ArgumentException`. When not available, it returns an empty list.
- `Changed` fires after `Apply(look)`, `Live(look)` and `Randomise()`, and it receives the captured look. The Appearance overloads do not fire it. That look is null when the capture fails. A handler that throws is logged, and the other handlers still run.
- Gender rules: the Head list follows `male`. A female beard is not pinned in `Options` or `Apply`, and only `Randomise` and `Vanilla` roll it as its first entry. Breast size is not limited by gender in `Apply`, `Live` or `Randomise`: `Randomise` rolls it over 0 to 1 for either gender, and keeps the captured `male` and `advanced`. Only `Vanilla` uses the game's gender ranges (male 0 to 0.3, female 0.3 to 1).
- Nothing is saved. Store a `Look` with [`SaveData.Register`](api-core/savedata.md) if you need it.
- A null `look` or `appearance` throws `ArgumentNullException`, except that `Options` returns an empty list for a null `appearance`. Game errors are logged to `BepInEx\logs\obenseuer.moddingframework.playerlook.log`, and the call returns `false` or `null`.

Debug check:

- `[Debug] PlayerLookSelfCheck = false` in `BepInEx\config\obenseuer.moddingframework.cfg`. When it is true, the framework runs the check at Awake and logs `playerlook self-check ok`, or `playerlook self-check failed: <check>`, to `BepInEx\logs\ObenseuerCore.log`. The check covers the dial clamps and the wearable rules, on temporary `Wearable` assets. It does not touch an Appearance.

This plugin logs each look change and takes the player's hat off at start.

```csharp
using BepInEx;
using BepInEx.Unity.Mono;
using ObenseuerCore;

public class LookTweak : BaseUnityPlugin
{
    private void Awake()
    {
        PlayerLook.Changed += look => Logger.LogInfo("look changed");
    }

    private void Start()
    {
        PlayerLook.Look look = PlayerLook.Capture();
        if (look == null) return;
        look.hat = "";
        PlayerLook.Apply(look);
    }
}
```

Open questions:

- `Apply`, `Live` and `Randomise` on a running player: the body and clothes after `LoadCharacter`.
- `IsAvailable` on the main menu and during scene changes.

UI kit (`ModUi`, `UiTheme`, `UiSprites`): see [UI kit](api/ui.md).
