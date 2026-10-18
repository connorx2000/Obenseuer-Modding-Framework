---
title: Common tasks
parent: Audio
grand_parent: Screen and UI
nav_order: 1
---

# Common tasks

Short snippets for the usual jobs with `Sounds`: play the game's sounds, play a WAV file, read the volume sliders, and change the volume and ambient sound for this session.

This page uses `Vector3`, `AudioClip` from Unity. The [glossary](../../glossary.md#types-that-come-from-unity) says what each one is.
{: .unity-types}

Play the sound an item makes when you pick it up. [`PickupNames()`](playing-the-game-s-own-sounds.md) lists the names you can use, such as `Can`, `Cash` and `Keys`.

```csharp
bool played = Sounds.PlayPickup("Cash");
```

Play one of the game's random sound sets, either with no position or at a place in the world. `LibraryNames()` lists the sets.

```csharp
Sounds.PlayLibrary("scarySounds");
Sounds.PlayLibraryAt("foley", new Vector3(0f, 0f, 0f));
```

Load a WAV file from disk and play it. [`LoadWav`](loading-a-wav-file.md) gives you `null` when the file can't be used, so check for that.

```csharp
AudioClip clip = Sounds.LoadWav(@"C:\path\to\your\sound.wav");
if (clip != null) Sounds.PlayClipAt(clip, "Effects", new Vector3(0f, 0f, 0f), 1f);
```

Read the [volume sliders](volumes.md). The numbers are decibels, not a percentage.

```csharp
Log.Info("Master " + Sounds.MasterVolume + " dB, sound " + Sounds.SfxVolume + " dB, music " + Sounds.MusicVolume + " dB");
```

Make the game quieter for this session only, then change the ambient sound. See [Changing a volume for this session](changing-a-volume-for-this-session.md) and [Ambient sound (soundscape)](ambient-sound-soundscape.md).

```csharp
Sounds.SetMixerVolume("example.beep", "Music", -20f);
Log.Info("Now playing the ambient sound: " + Sounds.SoundscapeId);
Sounds.ResetSoundscape();
```
