---
title: Saving
parent: Custom skills
grand_parent: Player life and skills
nav_order: 5
---

# Saving

Skill values are kept in the player's save, so progress survives between sessions. This page says where they are stored and when a stored value is used.

## Things to know

- Skill values are saved with the player's [save](../../guides/getting-started/save-data.md). They go in a file called `obenseuer.moddingframework.customskills.json` in the save's mods folder. The file has one entry for each mod and skill: the [mod id](../../glossary.md#mod-id), the skill id and the value.
- Every save writes every entry, including skills from mods that are not installed now. That way a player can remove a mod and put it back without losing progress.
- A saved value waits until the skill is registered again. On load, every valid entry is kept as it is. A value is only used for a registered skill. So an entry waits until its mod registers the skill again, and then `Add` or `Set` changes that same entry. If a file holds two entries for one skill, the last one is kept.
- The values are emptied when the player goes back to the main menu. <span class="tech">(It happens when the framework resets its save data as the game loads the main menu.)</span>
