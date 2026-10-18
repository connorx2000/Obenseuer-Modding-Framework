---
title: Playing notes and songs
parent: Instruments
grand_parent: Objects and devices
nav_order: 3
---

# Playing notes and songs

These calls make a guitar sound: one strum, a timed run of strums, or a long held note. Use them to play a guitar from your mod instead of the player.

The first argument is your [mod id](../../glossary.md#mod-id) (`Mod.Id`). It must be 1 to 64 letters, digits, dots, underscores or dashes. A bad id throws an `ArgumentException`. Two mods can use the same guitar without a clash, but a guitar plays only one song and one held note at a time.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>Instruments</code> <span>play and hold guitar notes and songs</span></p>

This is the class you start from. `PlayNote` plays one strum, `PlaySong` plays a timed run, and `HoldNote` starts a long note. Each call takes your mod's id, so `Clear` and `StopSong` stop only your own sounds.

<div class="api-pair" markdown="1">

```csharp
public static class Instruments
{
    public static bool PlayNote(string modId, string guitarId);
    public static bool PlaySong(string modId, string guitarId, IList<float> delays);
    public static bool StopSong(string modId, string guitarId);
    public static bool HoldNote(string modId, string guitarId, int index);
    public static bool StopHold(string modId, string guitarId);
    public static int HoldCount(string guitarId);
    public static bool Clear(string modId);
}
```

| Method | Returns | What it does |
|---|---|---|
| `PlayNote(modId, guitarId)` | `bool` | Plays one strum, the same as the player pressing the Primary Use key. `true` when it sounded. |
| `PlaySong(modId, guitarId, delays)` | `bool` | Plays a timed run of strums. `true` when the song is set. |
| `StopSong(modId, guitarId)` | `bool` | Stops your song on this guitar. `true` when a song was stopped. |
| `HoldNote(modId, guitarId, index)` | `bool` | Starts one of the guitar's long notes. `true` when the sound started. |
| `StopHold(modId, guitarId)` | `bool` | Stops your long note. `true` when a note was stopped. |
| `HoldCount(guitarId)` | `int` | How many long notes this guitar has. |
| `Clear(modId)` | `bool` | Stops all your songs and long notes. `true` when anything was stopped. |

</div>
</div>

## Things to know

- `PlaySong` plays a run of strums on one guitar. `delays` is the list of waits in seconds. The first number is the wait before the first strum. Each later number is the wait after the strum before it. For example `{ 0f, 0.5f, 0.5f, 1f }` plays four strums at about 0, 0.5, 1 and 2 seconds. A first number of `0` plays the first strum on the next frame. <span class="tech">(Each strum is the game's own `onPrimaryUse`, so `GuitarPlayed` fires for each one.)</span>
- A strum that falls due at a bad moment is skipped, not delayed. That is when the guitar is no longer held, is resting, or the player cannot use items at that moment. The song carries on with the next number, so nothing sounds late.
- A guitar plays one song at a time. A new `PlaySong` on the same guitar replaces the song that was playing, even if another mod started it. One warning then goes to the [log](../../guides/getting-started/logging.md), naming both mods.
- A song can run a few frames late but never loses a strum. A song plays at most four strums in one frame, and the rest play on the next frames. A song stops if its guitar is removed or switched off. <span class="tech">(The framework finds the guitar once, when `PlaySong` is called. It advances the song once per frame with `Time.deltaTime`.)</span>
- `HoldNote` starts one of the guitar's long notes. The note keeps sounding until it ends or you stop it. `index` counts from `0` up to `HoldCount(guitarId) - 1`. <span class="tech">(It plays an entry of the guitar's `primaryHoldAudio` array. It also refuses a sound that is already playing, and lets go of it if the player starts a hold on the same sound.)</span>
- A guitar has one held note at a time. A new `HoldNote` on the same guitar replaces the old one and stops its sound, even if another mod started it. One warning goes to the log, naming both mods.
- A held note ends when you stop it with `StopHold` or `Clear`, or when a level loads. It also ends when the player puts the guitar away, the guitar is destroyed, or the player cannot use items. The note also ends when its sound ends on its own. <span class="tech">(The end check runs each frame only while a held note exists. "A level loads" means a Single scene load.)</span>
- `StopHold` and `StopSong` only stop what your own mod started.
- `Clear(modId)` stops every song and every held note that your mod started.
- `HoldCount(guitarId)` gives how many long notes the guitar has. It gives `0` when no guitar has that id. It does not check `CanPlayNote`.
- Loading a level ends every song and every held note. Start them again after the load if you still need them. <span class="tech">(This is a Single scene load.)</span>

## Limits

- `PlayNote` returns `false` in these cases: no guitar has that id, `CanPlayNote` is `false`, a save is loading or being written, or the player cannot use items. <span class="tech">(The player cannot use items when the game is paused (`GameController.GameIsPaused`), a dialogue is active (`DialogueController.DialogueActive`), a game menu is open (`GameUIController.IsMenuVisible`) or item use is switched off (`PlayerHandItems.itemUseDisabled`). These are four of the checks the game's Primary Use path makes.)</span>
- `PlaySong` returns `false` when `delays` is `null`, empty, or has a number that is negative, infinite or not a real number. It also returns `false` when no guitar has that id or `CanPlayNote` is `false` at the start. The same goes while a save is loading or being written.
- `HoldNote` returns `false` in these cases: `index` is out of range, no guitar has that id, `CanPlayNote` is `false` at the start, or that sound is missing. It also returns `false` while a save is loading or being written, and while the player cannot use items.

**Common problems**

- A song or held note stopped after a level load: loading a level ends every song and every held note. Start them again after the load.
- A call throws an `ArgumentException`: the mod id is not 1 to 64 letters, digits, dots, underscores or dashes. Pass `Mod.Id`, which is always valid.
