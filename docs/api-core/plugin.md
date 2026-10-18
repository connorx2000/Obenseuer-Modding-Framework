---
title: Plugin
parent: Framework core classes
grand_parent: Framework basics
nav_order: 2
---

# Plugin

The framework is itself a [BepInEx plugin](../glossary.md#plugin), a class named `Core`. This page covers the few public members of `Core`, which tell you about the game build and a debug check. It also lists the settings in the framework's [config file](../guides/getting-started/config.md).

BepInEx is the game's mod loader. `Core` checks which game build is running and reads its own config file. Most of what the framework offers is on the other pages in this section.

This page lists what the API gives you, one section for each part. Text in brackets at the end of a note is extra detail for experienced modders. You can skip it on a first read.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>Core</code> <span>the framework's own plugin class</span></p>

Read `IsSupportedBuild` or `GameBuild` to see whether this is the game build the framework supports. `PatchingAllowed` tells you whether the framework will change game methods.

<div class="api-pair" markdown="1">

```csharp
public class Core : BaseUnityPlugin
{
    public static readonly string GameBuild;
    public static readonly bool IsSupportedBuild;
    public static bool PatchingAllowed { get; }
    public static bool RawAccessAvailable { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `GameBuild` | `string` | The game version and build, such as "0.5.08 (buildid 25969064)". It reads "buildid unknown" when no Steam manifest is found. |
| `IsSupportedBuild` | `bool` | `true` when the game is version 0.5.08, and the build id matches when it is known. |
| `PatchingAllowed` | `bool` | `false` only when the build is unsupported and you turned on `Safety.RefuseUnknownBuild`. |
| `RawAccessAvailable` | `bool` | `true` when the optional access check passed. `false` when the check did not run. |

</div>
</div>

### Things to know

- `Core` is a [BepInEx](../glossary.md#bepinex) plugin, so its base type depends on which BepInEx you run. Its static members work from framework mods too. The [framework mods](../api/framework-mods.md) page lists the other members that depend on the loader.
- By default the framework still works on a game build it was not tested on. It logs a warning instead of stopping. A player who wants it to stop sets `[Safety] RefuseUnknownBuild` to true. Then `PatchingAllowed` is false on other builds.
- `RawAccessAvailable` means something only when the access check is turned on. The check runs at start when `[Debug] AccessProbe` is true. It tries to reach the game's private members and logs `L0 access probe: OK`, or `L0 access probe: <ExceptionType>: <message>`. The save data methods do not depend on it. <span class="tech">("L0" is the framework's name for plain access to the game's code with no workaround.)</span>
- While the ConfigurationManager window (the in-game config window, F1) is open, the player cannot look around or move. The framework freezes mouse-look and movement and keeps the cursor visible. The freeze ends when the window closes. Other game menus do not freeze the game this way. Nothing happens when ConfigurationManager is not installed. <span class="tech">(It sets the game's `GameController.ControlsDisabled`.)</span>

### Config file settings

The framework's config file is `BepInEx\config\obenseuer.moddingframework.cfg`. Players change its settings in a text editor or in the in-game config window. The table lists each setting and what it does.

| Setting | What it does |
|---|---|
| `[Safety] RefuseUnknownBuild = false` | <details>{::nomarkdown}<summary>{:/nomarkdown}When true, the framework changes no game method on a build other than 0.5.08 (buildid 25969064).{::nomarkdown}</summary>{:/nomarkdown}By default an unknown build is changed anyway, and a warning is logged. It takes effect on the next start.</details> |
| `[Debug] Master = false` | <details>{::nomarkdown}<summary>{:/nomarkdown}Turns on the framework's debug trace.{::nomarkdown}</summary>{:/nomarkdown}The trace is written to `BepInEx\logs\ObenseuerCore.debug.log`. It is also in the Mods menu and in the `core_debug` console command. See [Debug](../api/debug.md).</details> |
| `[Debug] TraceFilter` | Limits the trace to type names that start with the text you give. |
| `[Debug] Verbose = false` | When true, [`CoreLog.Debug`](corelog.md) writes its lines. |
| `[Debug] AccessProbe = false` | Runs the access check at start and logs the result. |
| `[Debug] PlayerLookSelfCheck = false` | Runs a self-check of the [look](playerlook.md) limits and gender rules at start, and logs the result. |
| `[Debug] ModIdsSelfCheck = false` | Runs a self-check of [mod ids](../glossary.md#mod-id) and save file names at start. It logs `modids self-check ok`, or the first case that failed. |
| `[Debug] NetBlobSelfCheck = false` | <details>{::nomarkdown}<summary>{:/nomarkdown}Runs a self-check of the way large data is prepared for the network.{::nomarkdown}</summary>{:/nomarkdown}The data is split into chunks, checked and compressed. It logs `netblob self-check ok`, or the first case that failed.</details> |
| `[Debug] SaveDataSelfCheck = false` | Adds the `core_savedata_check` console command, which logs `SaveData.SelfCheck()`. It changes the live inventory, so use a scratch save. See [Save data (game)](../api/savedata-game.md). |
| `[Debug] SaveJsonSelfCheck = false` | Runs a self-check at start that saves and loads a value as text. It logs `save json self-check ok`, or the first case that failed. |
| `[Debug] JsonProbeCommand = false` | Adds the `core_jsonprobe` console command. It logs one test value as the game's `JsonUtility` writes it and as [`SaveData`](savedata.md) writes it. |
| `[Debug] SceneLooksCommand = false` | <details>{::nomarkdown}<summary>{:/nomarkdown}Adds the `scenelooks` console command, which prints the look index of a scene.{::nomarkdown}</summary>{:/nomarkdown}Its sub-commands are `list`, `listall`, `find`, `findchar` and `here`. It only reads the installed game files and changes nothing in the game. See [Scene looks](../api/scenelooks.md). The framework's own Settings page in the Mods menu also has a `Scene looks` action, `Run scene looks checks`. It needs no flag and no console. It prints the same index for the loaded scene to the menu status line and the log.</details> |
| `[UI] ThemeConfigManager = true` | Gives the ConfigurationManager window (the in-game config window opened with F1) a dark theme. |
