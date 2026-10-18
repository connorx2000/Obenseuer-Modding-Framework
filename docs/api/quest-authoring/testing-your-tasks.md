---
title: Testing your tasks
parent: Quest authoring
grand_parent: Story
nav_order: 6
---

# Testing your tasks

Two entries in the framework's [config](../../guides/getting-started/config.md) help you check your tasks while you build them.

## Things to know

- `Debug.QuestAuthoringSelfCheck` makes the framework [log](../../guides/getting-started/logging.md) a check of the definition rules and the ownership rule at start. Set it in the framework's config.
- `Debug.QuestAuthoringCommand` adds the console command `core_quests_check`. The command makes, advances and finishes a test task in the loaded save, and logs the result. It adds a finished task to that save, so use a scratch save.
