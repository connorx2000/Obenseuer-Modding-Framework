---
title: Common tasks
parent: NPC animation
grand_parent: People
nav_order: 1
---

# Common tasks

Short snippets for the usual [NPC animation](../npc-animation.md) jobs: holding an NPC in place during a clip, playing a clip once, playing on a figure your mod owns, and stopping all your clips.

In these snippets, `npcId` is an NPC id, `clip` is your `AnimationClip`, and `extraObject` is the object of an extra from [Mod NPCs](../modnpcs.md).

Keep an NPC in place while a clip plays, and let it go afterwards. Holding the NPC is done with [NPC movement](../npc-movement.md).

```csharp
NpcMovement.SetActivityEnabled(Mod.Id, npcId, false);
NpcAnimation.Play(Mod.Id, npcId, clip, true, 0.3f);

NpcAnimation.Stop(Mod.Id, npcId, 0.3f);
NpcMovement.SetActivityEnabled(Mod.Id, npcId, true);
```

Play a clip once, with no fade, with [`Play`](playing-and-stopping-on-an-npc.md).

```csharp
NpcAnimation.Play(Mod.Id, npcId, clip, false, 0f);
```

Play a clip on a figure your mod owns, such as an extra, with [`PlayOn`](playing-on-any-animator.md).

```csharp
Animator animator = extraObject.GetComponentInChildren<Animator>();
if (animator != null) NpcAnimation.PlayOn(Mod.Id, animator, clip, true);
```

Stop every clip your mod has started. `StopAll` returns how many clips it stopped.

```csharp
int stopped = NpcAnimation.StopAll(Mod.Id);
```
