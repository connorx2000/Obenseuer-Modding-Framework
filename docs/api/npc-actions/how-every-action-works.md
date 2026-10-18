---
title: How every action works
parent: NPC actions
grand_parent: People
nav_order: 2
---

# How every action works

The rules every [NPC action](../npc-actions.md) follows: what the result means, what is saved, and when an action is refused. Each action's own page adds its own rules. Text in brackets at the end of a note is extra detail you can skip on a first read.

## Things to know

- Every action returns `true` when it was done and `false` when it was not. Nothing throws an error. An NPC id the game does not know, a dead NPC, or a bad value gives `false`.
- Most actions change only the NPC in the [scene](../../glossary.md#scene) (one loaded area of the game). [`FollowPlayer`](following-the-player.md), `StopFollowing` and [`Kill`](moving-and-hurting.md) also change what the NPC's saved record says it is doing. `Hit` can too, when it finds a police or robber activity. <span class="tech">(The game's controller writes the NPC's current activity into its saved state.)</span>

## Limits

- Almost every action works only on an NPC that is standing in the [scene](../../glossary.md#scene) you are in. An NPC that is far away is not in the scene and has nothing to act on, so the action gives `false`. The one exception is [`ChangeLevel`](sending-an-npc-to-another-scene.md), which only works on an NPC that is not in the scene. <span class="tech">(The NPC needs a loaded scene instance (`NPCInfo`) with an `NPCController` on an active object, and it must not be dead. An away NPC has no controller, so the calls do not touch the game's NPC records (`NPCDirector` data). `ChangeLevel` is the only call that sets the scene or position in that record.)</span>
- Every action gives `false` while a save is loading or being written. [`Teleport`](moving-and-hurting.md) and [`StartDialogue`](speaking.md) also give `false` during a loading screen, the death screen, a waiting screen, a level change, an arrest or a sleep event. They also give `false` while any conversation is running. <span class="tech">(`Teleport` and `StartDialogue` check the framework's `SaveSlots.WorldBusy`, which covers all of those. The other live actions check only the save gate.)</span>
