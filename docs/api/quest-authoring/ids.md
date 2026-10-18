---
title: Ids
parent: Quest authoring
grand_parent: Story
nav_order: 2
---

# Ids

A task and its steps are named by ids, and the task id starts with your mod id. Read this before you choose a key for a new task.

## Things to know

- A task has a key you choose. The task id is your [mod id](../../glossary.md#mod-id), a dot and the key.
- The key is letters, digits, `_` and `-`, up to 48 characters, with no dot.
- Two mods can never share a task id, because the mod id comes first. [`QuestAuthoring.TaskId(modId, key)`](making-a-task.md) builds the id for you.
- An objective id is its key, and it must be unique inside its task. The game's scripts use these ids when they ask whether a task or objective is active. <span class="tech">(The game's Lua functions are `TaskActive("<taskId>")`, `ObjectiveActive` and the others.)</span>

## Limits

- Only the mod whose id starts the task id can change the task through [`QuestAuthoring`](changing-a-task.md). `AddObjective`, `SetObjective`, `SetCounter`, `Finish` and `Fail` return `false` for another mod's task or for one of the game's own. This works after a restart or a load, because the framework reads the mod from the id.
