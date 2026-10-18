---
title: Self-check
parent: Dialogue authoring
grand_parent: Story
nav_order: 7
---

# Self-check

Two start-up checks for dialogue authoring can be switched on in the [framework's config](../../api-core/plugin.md#config-file-settings). Use them when you want the framework to test its own dialogue rules when the game starts.

- `Debug.DialogueAuthoringSelfCheck` logs a check of the conversation rules, the line links and the speakers when the game starts.
- `Debug.DialogueResponsesSelfCheck` logs a check of how anchors are matched and how ids are given out for responses.
