---
title: HUD
parent: Screen and UI
grand_parent: API reference
has_children: true
nav_order: 2
---

# HUD

[`Hud`](hud/hud.md) puts your mod's messages on the game's screen, in the game's own style. Use this API when your mod needs to tell the player something, or to change achievements, difficulty or quest items.

Status: experimental.

## What it is

The HUD is everything the game draws on top of the world:

- the notification box
- subtitles
- the hint box the card game uses for its rules
- the crosshair
- the tooltip
- the Notes page

[`Hud`](hud/hud.md) shows messages and controls in the game's own style. It can also hide the game's HUD or crosshair for a moment.

This page also covers four smaller parts:

- [`ConsoleCommands`](hud/consolecommands.md) adds commands to the developer console (opened with F1).
- [`Achievement`](hud/achievement.md) reads and unlocks the game's achievements.
- [`GameDifficulty`](hud/gamedifficulty.md) makes the game easier or harder from a mod.
- [`TaskItems`](hud/taskitems.md) reads and changes the quest items the player carries.

## When to use it

- Show a message when something happens, such as "Rent is due" (`Hud.Notify`).
- Show a short line of story text at the bottom of the screen (`Hud.ShowSubtitle`).
- Show a boss or progress bar on screen (`Hud.Bar`).
- Add a command to the developer console for testing (`ConsoleCommands.Register`).
- Make the game easier or harder from a mod, for example cheaper prices (`GameDifficulty.SetFactor`).
- Check or unlock an achievement, or count the quest items the player holds.

## Quick example

This mod waits until a save is loaded, then shows a notification and a hint.

```csharp
using ObenseuerCore;

[ModInfo("example.hudhello", "Hud Hello", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class HudHello : FrameworkMod
{
    protected override void OnLoad()
    {
        Events.WhenReady(() =>
        {
            Hud.Notify("Hud Hello", "The mod is running.");
            Hud.ShowHint(Mod.Id, "Hud Hello", new[] { "This hint comes from a mod.", "Press the Hints key to hide it." });
        });
    }
}
```

- `Hud.Notify(title, body)` shows the game's notification box.
- `Hud.ShowHint` puts text in the same hint box the card game uses for its rules.
- The game must be loaded first, so both calls sit inside [`Events.WhenReady`](core-helpers/events-whenready.md). See [Using events](../guides/getting-started/using-events.md).

The player can hide the hint box with the Hints key (K by default). The game has no hints option in its settings menu. If a player does not see your hint, tell them to press that key. The FAQ entry [My hint does not show](../faq.md#my-hint-does-not-show) covers the same case.
