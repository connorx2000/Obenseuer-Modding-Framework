---
title: What you can't do (yet)
parent: NPC actions
grand_parent: People
nav_order: 8
---

# What you can't do (yet)

The things the [NPC actions](../npc-actions.md) cannot do today, with the reason and the closest call where one exists.

- You can't damage or heal an NPC. The game has no damage or health that NPCs use, so a health change would do nothing and would only look like it worked. [`Hit`](moving-and-hurting.md) is the closest call, and it does not change health. <span class="tech">(`NPCController` has no damage, health or `IDamageReceiver` member. The only health value is the public field `NPCStats.Health`, and no game code reads or writes it. Its `Activities_Health` triggers have no callers.)</span>
- You can't listen for when an NPC says a line, because there is no event for it. An event would need a change to the game's [bark](speaking.md) code that runs on every bark of every NPC. The speaker would also be unknown for lines said by world logic, which have no NPC. <span class="tech">(A `Barked` event would need a Harmony postfix on `SubtitleBarkUI.Bark`, with its own install and turn-off path. A world `logic_bark` speaker has no NPC, so its `NpcId` would be 0.)</span>
- You can't use the extra look calls the game has. The only look calls are the four on [Looking](looking.md). Looking at a named target, and turning off the look at the player, are not offered. <span class="tech">(The game's `LookAtNamedTarget` and `DisableLookAtPlayer` are not offered by the framework.)</span>
- You can't move an NPC that is not in the [scene](../../glossary.md#scene) (one loaded area of the game) one step at a time. Those NPCs move along the game's own routes between scenes and have nothing to teleport. [`ChangeLevel`](sending-an-npc-to-another-scene.md) sets the scene directly and does not walk the NPC.
