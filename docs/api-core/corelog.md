---
title: CoreLog
parent: Framework core classes
grand_parent: Framework basics
nav_order: 4
---

# CoreLog

`CoreLog` writes to a [log](../guides/getting-started/logging.md) file that belongs to your mod. Use it instead of `Console.WriteLine` or `Debug.Log`, so players find your mod's messages in one place.

Each line is stamped with the game frame and the time. A line looks like `[frame 1234] [56.789] INFO: message`.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>CoreLog</code> <span>your mod's log file</span></p>

This is the file your mod writes its own messages to. Get one with `CoreLog.For` and your [mod id](../glossary.md#mod-id), then write with `Info`, `Warn`, `Error` or `Debug`.

<div class="api-pair" markdown="1">

```csharp
public class CoreLog
{
    public static CoreLog For(string modId);
    public void Info(string message);
    public void Warn(string message);
    public void Error(string message);
    public void Debug(string key, string message);
    public int LineLimit { get; set; }
}
```

| Method | Returns | What it does |
|---|---|---|
| `For(modId)` | `CoreLog` | The log for your mod id, written to `BepInEx\logs\<modId>.log`. |
| `Info(message)` | `void` | Writes a normal message. |
| `Warn(message)` | `void` | Writes something that looks wrong while the mod keeps working. |
| `Error(message)` | `void` | Writes something that failed. |
| `Debug(key, message)` | `void` | Writes a message only when [`[Debug] Verbose`](plugin.md#config-file-settings) is true. After that it writes again only when the text for this key changes. |

| Property | Type | What it means |
|---|---|---|
| `LineLimit` | `int` | Most lines this log accepts per second. Default 100. Values outside 1 to 100000 are moved into that range. See Writing a long list. |

</div>
</div>

## Putting it together

This plugin writes a start-up line, and a verbose line that only shows when the `[Debug] Verbose` [config setting](plugin.md#config-file-settings) is on.

<div class="api-example" markdown="1">

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

| Step | What it does |
|---|---|
| `CoreLog.For("example.mymod")` | Opens the log for this mod's id. |
| `Log.Info("started")` | Writes a normal line. |
| `Log.Debug("state", "idle")` | Writes a line only when `[Debug] Verbose` is on. |

</div>

## Writing a long list

Each log accepts 100 lines in a second, so one noisy mod cannot fill the file. Further lines in that second are dropped. One summary line such as `340 lines suppressed` (level `WARN`) is written with the next line your mod writes, or when the game quits.

A mod that prints a long list in one frame, for example every [NPC](../api/npcs.md) id, therefore stops after about 100 lines. The summary line follows.

Raise the limit on your own log:

<div class="api-example" markdown="1">

```csharp
CoreLog Log = CoreLog.For("my.mod");
Log.LineLimit = 1000;
Log.Info("first line of a long list");
```

| Step | What it does |
|---|---|
| `Log.LineLimit = 1000;` | Sets the limit to 1000 lines in a second, so the whole list is written. |
| `Log.Info(...)` | Writes a line as usual. |

</div>

The other way is to spread the writing over several frames, for example a few lines in each `Update`. Then no single second goes over the limit.

Errors have their own rules on top of this limit:

- The same error is written once in 10 seconds.
- At most 20 errors are written in 10 seconds.
- Dropped errors get their own summary line, such as `7 error lines suppressed`.

## Things to know

- The log file is `BepInEx\logs\<modId>.log`, and the last run is kept. When the game starts, the previous file is renamed to `<modId>.log.1`. The file is written at once, line by line. When it reaches 4 MB it is moved to `.1`, replacing the older `.1`.
- A [mod id](../glossary.md#mod-id) that is not a usable file name logs to the [framework's](../glossary.md#framework) own log instead, `BepInEx\logs\ObenseuerCore.log`. That means an id with a path separator, `..` or `:` in it. It also means a Windows device name such as `CON`, `NUL` or `COM1`, or an id that ends in a dot or space.
- Lines written to the framework's own log start with your id as `[id] `. `CoreLog.For(null)` does the same and never throws.
- `Debug` is for messages you only want while testing. It prints nothing unless the `[Debug] Verbose` [config setting](plugin.md#config-file-settings) is true. It prints a line only when the text for that `key` differs from the last one. A message sent every frame therefore does not fill the file.
