---
title: Common tasks
parent: Settings
grand_parent: Framework basics
nav_order: 1
---

# Common tasks

Short snippets for the usual settings jobs: show the player's key in a prompt, read the sprint mode, and change the field of view for a while.

Tell the player which key to press. [`Keybinds.Label`](keybinds.md) gives the text for the device in use, so a controller player sees a button name.

```csharp
Hud.ShowHint(Mod.Id, "Door", new[] { "Press [" + Keybinds.Label("Interact") + "] to open." });
```

Find out whether the player runs in toggle or hold mode with [`GameSettings`](gamesettings.md). `true` means toggle: press once to start, press again to stop.

```csharp
bool toggleSprint = GameSettings.ToggleSprint();
```

Change the field of view without touching the player's saved setting. The value is only in memory, and `ClearFov` hands the camera back. `Mod.Id` is the [owner](../../glossary.md#owner) that holds the change.

```csharp
GameSettings.SetFov(Mod.Id, 90);
GameSettings.ClearFov(Mod.Id);
```
