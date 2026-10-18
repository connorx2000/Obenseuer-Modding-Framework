---
title: Characters from the game and characters your mod added
parent: NPC appearance
grand_parent: People
nav_order: 6
---

# Characters from the game and characters your mod added

How long an appearance change lasts depends on whether the character comes from the game or is a [mod NPC](../../glossary.md#mod-npc) (a person your mod adds). This page also says what `Restore` and `Clear` undo in each case.

## Things to know

- A change to a character from the game lasts for this play session only. The framework changes the parts that are on screen and never edits the game's shared character data. Nothing carries over to another spawn of the same character, to another [scene](../../glossary.md#scene) or to a save.
- A change to a [mod NPC](../../glossary.md#mod-npc) can last longer. The change is made on the parts on screen and on the mod NPC's own copy of the character data. So it survives the game setting the looks again. `Restore` and `Clear` undo both.
- The look and blend shape changes are kept with the character. For a mod NPC added with [`SpawnSaved`](../modnpcs/spawnsaved.md), they are saved with the game and come back after a load. For one added with [`Spawn`](../modnpcs/spawn.md), they last for the session.
- Changes to parts and materials last for the session only. See [Look and shape](../modnpcs/look-and-shape.md#look-and-shape).

## Limits

- The game sets a character's looks again when the character is created. A change made in the same frame the scene loads can be lost.
- `Restore` and `Clear` do not undo looks and blend shapes while a save is loading or being written. Your look and blend shape changes stay, and a later call undoes them. The part and material changes are undone at once.
