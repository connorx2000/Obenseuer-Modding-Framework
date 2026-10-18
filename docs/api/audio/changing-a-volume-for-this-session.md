---
title: Changing a volume for this session
parent: Audio
grand_parent: Screen and UI
nav_order: 7
---

# Changing a volume for this session

`SetMixerVolume` changes how loud one of the game's volume channels is, until the game puts the player's own volumes back. Use it to make the game louder or quieter without touching the player's saved settings.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>Sounds</code> <span>volume channels</span></p>

Call `SetMixerVolume` with your mod id, a channel name and a value in decibels. It changes that channel for this session. The volumes the player saved are not changed.

<div class="api-pair" markdown="1">

```csharp
public static class Sounds
{
    public static bool SetMixerVolume(string modId, string group, float decibels);
}
```

| Method | Returns | What it does |
|---|---|---|
| `SetMixerVolume(modId, group, decibels)` | `bool` | Sets a channel's loudness. `false` when it was not set. |

</div>
</div>

## Things to know

- `group` must be `Master`, `Sound`, `Music` or `Nasty`. These are the game's own volume channels. Setting `Sound` also sets the other sound effects channel, the same as the game's Sound slider does. <span class="tech">(`Sound` also sets the mixer parameter `OtherSounds`.)</span>
- `decibels` must be a real number from -80 (silent) to 0 (loudest). The framework does not compare it with the game's sliders.
- `modId` is your mod's id. It has 1 to 64 letters, digits, `.`, `_` or `-`. See [Mod id](../../glossary.md#mod-id).
- The change lasts for the session only. It does not change the volumes the player saved, so [`MasterVolume`, `SfxVolume` and `MusicVolume`](volumes.md) keep reporting the saved numbers.

## Limits

- The game puts the player's volumes back whenever it sets up its options. This also happens when the pause menu builds its options page. If your value has to last, set it again after a [scene](../../glossary.md#scene) change. <span class="tech">(The game restores the saved mix when `OptionsController.Start` and `OptionsMenu.Start` run, because they call `VolumeSave.GetVolume`, and when the pause menu sets up the options menu.)</span>
- It returns `false` when nothing was changed. That happens for an invalid mod id, an unknown group or a value out of range. It also happens while a save is loading or saving, or when a game volume setting is missing. A wrong mod id, group or value is also written to the [log](../../guides/getting-started/logging.md) once.

**Common problems**

- Your volume change is gone: the game put the player's volumes back when it set up its options, or the pause menu built its options page. Set your value again after a scene change.
- `SetMixerVolume` returns `false`: the mod id is invalid, the group is unknown, or the value is out of range. A save loading or saving, or a missing game volume setting, has the same result. Check the log, which names a wrong mod id, group or value once, and pass values that follow Things to know.
