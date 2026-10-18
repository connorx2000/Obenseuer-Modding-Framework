---
title: Framework mods
parent: Framework basics
grand_parent: API reference
has_children: true
nav_order: 5
---

# Framework mods

A framework mod is one class in one DLL (the mod file) that the framework loads for you. This section is the reference for that class, its labels, settings, loading and dependencies. If you have not made a mod yet, start with [Your first mod](../guides/getting-started/first-mod.md), which builds one step by step.

Status: experimental. Needs framework 0.11.0 or later.

## What it is

A framework mod is the simplest kind of mod for the game. It is one DLL with one class. The framework finds the DLL in `BepInEx\plugins\` when the game starts and loads it for you.

The same DLL runs on both BepInEx 5 and BepInEx 6, the two versions of [BepInEx](../glossary.md#bepinex) (the game's mod loader) that the framework supports. You do not need one build for each. Your mod does not reference BepInEx at all.

The pages in this section cover the pieces of a framework mod:

- the class you write, and the labels that give it a name and a version;
- its settings;
- how it loads;
- how it depends on other mods.

## When to use it

- Start a new mod. This is the form to use unless you have a reason not to.
- Add settings to a mod that show up in the [Mods menu](framework-mods/mods-menu.md) in the game.
- Make your mod wait for another mod, or only run when it is there.

Use a [BepInEx plugin](../guides/getting-started/packaging.md#bepinex-plugins-advanced) instead when you need something from BepInEx itself, such as `KeyboardShortcut`. A table of those cases is on [Loader-specific members](framework-mods/loader-specific-members.md), near the end of the reference.

## Quick example

This is a complete mod. It writes a line to its [log](../guides/getting-started/logging.md) and adds a speed setting to the Mods menu. Once a save is loaded, it shows a hint on screen. [Your first mod](../guides/getting-started/first-mod.md) builds a mod like this one step by step.

```csharp
using ObenseuerCore;

[ModInfo("example.myfirstmod", "My First Mod", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public class MyFirstMod : FrameworkMod
{
    protected override void OnLoad()
    {
        Log.Info("loaded");
        ModSetting<float> speed = Config.Bind("General", "Speed", 1f, "Walk speed factor.", 0.5f, 2f);
        Events.WhenReady(() => Hud.ShowHint(Mod.Id, "My First Mod", new[] { "Hello." }));
    }
}
```

What each part does:

- `[ModInfo]` gives the mod its id, its name and its version.
- `[ModDependency]` says the framework must be at least version 0.11.0.
- `OnLoad` is where you start your work.
- `Config.Bind` makes the setting. The [Config](../guides/getting-started/config.md) guide explains settings step by step.
- [`Events.WhenReady`](core-helpers/events-whenready.md) waits until a save is loaded before it runs your code. See [Using events](../guides/getting-started/using-events.md) for more on it.
