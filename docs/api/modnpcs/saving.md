---
title: Saving
parent: Mod NPCs
grand_parent: People
nav_order: 7
---

# Saving

This page covers how a [mod NPC](../../glossary.md#mod-npc) (a copy of an existing person) stays out of the game's save and when it is removed. It also says how to keep one after a load.

## Things to know

- A mod NPC has no save record of its own. So it can't damage the save of a vanilla person, one from the game's own cast. The game writes no entry for it and can't mix it up with a person from the game when a save loads. A person from the game is not touched by the copy. <span class="tech">(every script on it has its save id cleared; its position recording component is removed and its bark recording components are turned off, because they would write into the save under the source's name and overwrite the source's saved position; turning the position component off is not enough, because the game's save recorder still reaches it)</span>
- The only way a mod NPC can reach the save is `keepSourceDialogue = true`. That applies to a copy built from the source's own model. See [Spawn](spawn.md).
- `Despawn` and `Clear` hide the person and remove the character straight away. The object is destroyed half a second later. They don't check whether the game is loading or saving, because the person is switched off at once. <span class="tech">(the NPC is set inactive at once and destroyed 0.5 s later)</span>
- Call `Clear` when your mod stops needing its people. The framework does not do it for you.
- `Despawn` and `Clear` return `false` when your mod has no matching person.

## Limits

- A person lasts for the [scene](../../glossary.md#scene). A scene change destroys the person, and the framework then removes the character.
- For a person who must come back after a load, use [`SpawnSaved`](spawnsaved.md). With `Spawn`, add the person again from [`Events.WhenReady`](../core-helpers/events-whenready.md), using the same `npcId` to get the same character number. See [Extras](extras.md) for why not `Events.SceneLoaded`.
- A saved person is the exception to `Despawn` and `Clear` working straight away. They return `false` during a load or a save when that would remove the record.

**Common problems**

- A person from `Spawn` is gone after a scene change or a load: the person lasts for the scene only. Use `SpawnSaved`, or add the person again from `Events.WhenReady` with the same `npcId`.
