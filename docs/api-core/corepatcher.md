---
title: CorePatcher
parent: Framework core classes
grand_parent: Framework basics
nav_order: 3
---

# CorePatcher

`CorePatcher` lets your mod change a game method, so your code runs before it, after it, or around it. Use it instead of calling [Harmony](../glossary.md#harmony) (the patching library [BepInEx](../glossary.md#bepinex) ships) yourself. See [Patching](../guides/getting-started/patching.md) for a longer walkthrough.

The [framework](../glossary.md#framework) changes some of the game's methods the same way, so it can tell your mod what is happening.

A change that runs before the game method is a prefix. One that runs after it is a postfix. One that runs even when the game method throws an error is a finalizer. Pass `null` for the ones you do not need.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>CorePatcher</code> <span>change a game method</span></p>

Call one of the `Patch` overloads with the game method and your prefix, postfix or finalizer. `IsDisabled` tells you whether the framework has turned off your changes.

<div class="api-pair" markdown="1">

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

| Method | Returns | What it does |
|---|---|---|
| `Patch(target, prefix, postfix, finalizer, core, owner)` | `bool` | Changes a game method given as a method object. |
| `Patch(type, name, args, prefix, postfix, finalizer, core, owner)` | `bool` | The same, but finds the game method by its type, name and parameter types. |
| `Patch(target, prefix, postfix, finalizer, core, owner, priority)` | `bool` | The same as the first, with a priority that sets the order against other mods' changes. |
| `Patch(type, name, args, prefix, postfix, finalizer, core, owner, priority)` | `bool` | The same as the second, with a priority. |
| `IsDisabled(owner)` | `bool` | `true` when the framework turned off all changes made under this mod id. |

</div>
</div>

## Putting it together

This plugin runs a small method after the game counts a new day. It logs a warning if the change cannot be made.

<div class="api-example" markdown="1">

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

| Step | What it does |
|---|---|
| `CorePatcher.Patch(typeof(TimeOfDayAzure), "IncreaseDayCounter", ...)` | Finds the game method and sets `AfterDayChanged` to run after it. |
| `CorePatcher.IsDisabled("example.mymod")` | Checks whether the framework turned off the changes. |

</div>

## Things to know

- Every `Patch` call returns `true` when the change was made, and `false` when it was not. In these calls, `owner` is your [mod id](../glossary.md#mod-id) (see [Owner](../glossary.md#owner) in the glossary).
- The short calls (without `priority`) keep the priority attributes you put on your methods. That means `[HarmonyPriority]`, `[HarmonyBefore]` and `[HarmonyAfter]`. <span class="tech">(Checked with HarmonyX 2.9.)</span>
- `priority` decides which of several mods' changes runs first. It applies to your prefix, postfix and finalizer, and it replaces a `[HarmonyPriority]` attribute. Pass `-1` to leave the attributes alone. A higher number runs earlier for prefixes. The normal value is 400 (`Priority.Normal`), and the highest common value is 800 (`Priority.First`). <span class="tech">(It is the HarmonyX priority, `HarmonyLib.Priority`.)</span>
- A prefix that returns `false` skips the game method, but it does not stop other mods' prefixes. Every prefix still runs, even when an earlier one returned `false`. A higher priority makes yours run before the others, but it cannot stop them. <span class="tech">(This is how HarmonyX works. A `false` from a prefix only skips the original method.)</span>
- The [framework](../glossary.md#framework) keeps track of every change. It undoes them together if a game update breaks one, and keeps different mods from stepping on each other.
- Each mod id gets its own [Harmony](../glossary.md#harmony) instance. Your changes are kept apart from everyone else's.
- When a change fails, you get `false` and a line in your mod's [log](../guides/getting-started/logging.md). The log is `BepInEx\logs\<owner>.log`.
- `core: true` means this change is critical for your mod. If it fails, the framework removes every change it made under your mod id, and `IsDisabled(owner)` becomes true. Changes you made yourself with `new Harmony(owner)` are not touched.
- Once your mod id is disabled, it stays disabled until the game restarts. Later `Patch` calls return `false` and write nothing to the log.

## Limits

- Your [mod id](../glossary.md#mod-id) must not start with `obenseuer.moddingframework` or `nomad.obenseuer.core`, and it must not be empty. Those two starts are reserved for the [framework's](../glossary.md#framework) own changes, in any mix of capital and small letters. If you break this rule, all four `Patch` calls throw an `ArgumentException` before they do anything else.
- The framework refuses to change anything when `Core.PatchingAllowed` is false. The call returns `false` and logs `patch skipped, unsupported game build`. That only happens when the game build is not 0.5.08 and the `[Safety] RefuseUnknownBuild` [config setting](plugin.md#config-file-settings) is set. By default an unknown build is changed anyway.

**Common problems**

- `Patch` returns `false` and writes a line to `BepInEx\logs\<owner>.log`: the change failed. Read that line.
- Later `Patch` calls return `false` and log nothing: a `core: true` change failed, so your mod id is disabled and `IsDisabled(owner)` is `true`. Restart the game to patch again.
- The log says `patch skipped, unsupported game build`: the game build is not 0.5.08 and `[Safety] RefuseUnknownBuild` is set. Without that setting, an unknown build is changed anyway.
- `Patch` throws `ArgumentException`: the mod id is empty or uses a reserved start. Use an id that does not start with `obenseuer.moddingframework` or `nomad.obenseuer.core`.
