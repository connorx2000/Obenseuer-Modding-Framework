---
title: What you can't do (yet)
parent: Mod NPCs
grand_parent: People
nav_order: 9
---

# What you can't do (yet)

What [`ModNpcs`](types-and-members.md) does not do for a [mod NPC](../../glossary.md#mod-npc) (a copy of an existing person). Each item names the page that covers it instead, where there is one.

- You can't make a mod NPC walk on its own or follow a timetable from this page. Use the [mod NPC routines](../modnpc-routines.md) page for walks, wandering, patrols and daily timetables.
- You can't add new dialogue lines here. New conversations come from [dialogue authoring](../dialogue-authoring.md). The source person's own conversation can be run with `keepSourceDialogue` (see [Spawn](spawn.md)).
- You can't build a body from nothing. A body is always an existing character's 3D model. The framework can't build a skeleton, so a source or body with no model and no default body can't be added.
