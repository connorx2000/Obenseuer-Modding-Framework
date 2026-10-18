---
title: Playing and stopping on an NPC
parent: NPC animation
grand_parent: People
nav_order: 2
---

# Playing and stopping on an NPC

This is the full list of what the API gives you. In these calls, `modId` is the [mod id](../../glossary.md#mod-id) of your mod, `loop` says whether the clip repeats, and `fadeSeconds` is how long the fade takes. The default fade is a fifth of a second.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>NpcAnimation</code> <span>clips on scene NPCs</span></p>

`Play` starts a clip on an NPC in the [scene](../../glossary.md#scene) (one loaded area of the game), and `Stop` fades it out. `IsPlaying` tells you whether one is running.

<div class="api-pair" markdown="1">

```csharp
public static class NpcAnimation
{
    public static bool Play(string modId, int npcId, AnimationClip clip, bool loop, float fadeSeconds = 0.2f);
    public static bool Stop(string modId, int npcId, float fadeSeconds = 0.2f);
    public static bool IsPlaying(int npcId);
    public static int StopAll(string modId);
}
```

| Method | Returns | What it does |
|---|---|---|
| `Play(modId, npcId, clip, loop, fadeSeconds)` | `bool` | Starts a clip on an NPC. |
| `Stop(modId, npcId, fadeSeconds)` | `bool` | Fades the clip out. |
| `IsPlaying(npcId)` | `bool` | `true` while a clip runs on the NPC. |
| `StopAll(modId)` | `int` | Ends all clips of your mod, and returns how many it ended. |

</div>
</div>

## Things to know

- A clip plays only on an NPC that is in the [scene](../../glossary.md#scene), switched on, and alive. For an NPC that is far away, switched off, or dead, `Play` gives `false`. <span class="tech">(The NPC needs a loaded scene instance that is active in the hierarchy and not dead. `Play` also gives `false` for an NPC with no `Animator` and while a save is loading or being written.)</span>
- `Play` starts your clip and fades it in over `fadeSeconds`. A fade of `0` starts it at once.
- Each NPC has one clip at a time. Your mod may play a new clip on an NPC that already has one of yours. Then the old clip fades into the new one over `fadeSeconds`. A fade of `0` switches at once. <span class="tech">(A `Play` while a crossfade is still running first jumps to the newer clip, then crossfades from it.)</span>
- For a clip that plays once, the fade is at most half the clip's length. This keeps the fade from taking longer than the clip.
- With `loop` set to `true`, the clip repeats until you stop it. With `loop` set to `false`, it plays once. It starts to fade out `fadeSeconds` before its last frame, so it ends on the last frame, and then it stops. <span class="tech">(A clip that already repeats in Unity (`isLooping`) keeps repeating. For a clip that does not, the framework loops the clip's time at its length.)</span>
- `Stop` fades your clip out over `fadeSeconds`, and the NPC goes back to its own animation. A fade of `0` ends it at once. Only the mod that started the clip can stop it. `Stop` gives `false` for another mod's clip and when nothing is playing.
- A second `Stop` on a clip that is already fading out gives `false`. The exception is a fade of `0`, which ends the clip at once and gives `true`.
- `IsPlaying` gives `true` while a clip runs and is not fading out. It gives `false` in these cases: once the clip starts to fade out, after `Stop`, after a clip that plays once has ended, and for an NPC with no clip. It only looks at NPC clips, so it does not see a clip started with [`PlayOn`](playing-on-any-animator.md).
- `StopAll` ends every clip your mod has started, at once and without a fade. It gives how many clips it ended, counting both NPC clips and `PlayOn` clips. It still works while a save is loading.

## Limits

- If another mod's clip is playing, `Play` gives `false`. The exception is a clip of another mod that is already fading out, which is replaced.
