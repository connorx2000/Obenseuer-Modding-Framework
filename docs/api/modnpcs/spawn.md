---
title: Spawn
parent: Mod NPCs
grand_parent: People
nav_order: 3
---

# Spawn

[`Spawn`](types-and-members.md) adds one [mod NPC](../../glossary.md#mod-npc) (a copy of an existing person) for the rest of the [scene](../../glossary.md#scene). When it can't, it returns `0` and does not throw. A failure inside the game is written to the [log](../../guides/getting-started/logging.md) once, and nothing half-built is left in the scene.

## Things to know

- `Spawn` copies a person from the game. The `sourceCharacterId` is a character number from [`Characters.All()`](../characters.md). The player and other mod NPCs can't be a source.
- The copy keeps the source's name data, outfit and appearance data. It is built on a 3D body (see the body notes below).
- The person stands on the ground below `position`, not in the air. The game drops the spot onto the first surface within 4 units below it, starting 1 unit above it. <span class="tech">(the ground is the first collider within 4 units below the position, with a 1 unit lift)</span>
- If something blocks the spot, the game moves the person aside. A scene with nothing there refuses the spawn.
- `name` and `lastName` become the person's name. The display name is the two joined by a space. It is the name shown when the player looks at the person.
- The body decides what the person looks like in 3D. `bodyCharacterId` picks the character whose 3D model is used (the prefab, the game's ready-made template of that model). The version of `Spawn` without it passes `0`.
- `0` uses the source's own 3D model when it has one, and otherwise a default body. <span class="tech">(the default body is the non-player character with the lowest number in `Characters.All()` that has a model with a slider rig (an `Appearance` component) and an `NPCInfo` component, and that is not a mod NPC; when there is none, the spawn returns `0` and the framework logs it once)</span>
- [`ModNpcInfo.BodyCharacterId`](types-and-members.md) is the body that was used, or `0` when the source's own model was used.
- By default the person can't be talked to. The game shows "Can't talk". The source's conversation would run the source's own quest and variable lines. That is not what you want for a new person.
- `keepSourceDialogue = true` lets the person use the source's conversation. The lines are the source's, under the new name. <span class="tech">(the copy keeps the source's own conversation only when `bodyCharacterId` is `0` and the source has a prefab, or when the body is the source's own number)</span>
- A new person does nothing on its own. It has no timetable and no game record, so it stands still. The [movement](../npc-movement.md), [action](../npc-actions.md) and [appearance](../npcappearance.md) pages can address it by `CharacterId`. See [mod NPC routines](../modnpc-routines.md) for walks, wandering, patrols and daily timetables. <span class="tech">(the person keeps the source's AI components and is found by the game's NPC lookup, so `NpcMovement.MoveTo`, `NpcActions` and `NpcAppearance` work on it)</span>
- `List(modId)` lists the people your mod has added right now. `CharacterIdOf` gives one person's character number, or `0` when there is no such person.

## Limits

- `Spawn` returns `0` for these plain reasons:
  - the game is loading or saving
  - no scene with people in it is loaded
  - the source character is unknown
  - the body has no usable 3D model
  - there is no ground under the spot
  - the same `modId` and `npcId` is already added
- Missing ground is also written to the log as a warning. Read the log to see which reason it was.
- Any other body number uses that character's model. The player, a mod NPC, an unknown number, a negative number, or a character with no model returns `0`. A negative number is written to the log once.
- Whether the person can walk depends on the scene having a navigation graph (the map the game uses for finding paths).
- Most named people in the game have no 3D model of their own, so their look may not carry over. They are placed straight into a scene. The copy gets the source's name and appearance data. Whether the look carries over depends on where the game stores that person's look, so the copy may come out in the body's default look.
- Pass the body number yourself, for example the gym person's body, rather than relying on the default.
- [NPC appearance](../npcappearance.md) can only switch parts that the body already has. If the body has none of the source's parts, the copy shows the body's own look, and appearance changes do not show on it. If it has some of them, the body's other parts are hidden.
- `keepSourceDialogue = true` changes the player's save. Every quest and variable change that conversation makes is a change for the source person, and it is written to the player's save. Use it only when that is what you want.
- `keepSourceDialogue = true` only works when the copy uses the source's own 3D model. With any other body, talk stays off and the framework writes one line to the log.
- New dialogue lines are not part of this API. Runtime conversations come from [dialogue authoring](../dialogue-authoring.md).

**Common problems**

- `Spawn` returns `0`: one of the reasons in the first bullet under Limits applies. A refused body number gives `0` too: the player, a mod NPC, an unknown or negative number, or a character with no model. So does a `0` body when no default body exists. So do mod NPCs that are switched off (`ModNpcs.IsAvailable` is `false`). Read the log to see which reason it was.
- The person can't be talked to and the game shows "Can't talk": talk is off by default. Set `keepSourceDialogue = true` to use the source's conversation, but note that it changes the player's save.
- Talk stays off with `keepSourceDialogue = true`, and the log has one line about it: the copy does not use the source's own 3D model. Pass `0` as the body for a source that has a prefab, or pass the source's own number.
