---
title: Scene loads and saves
parent: NPC control
grand_parent: People
nav_order: 7
---

# Scene loads and saves

This page covers what happens to your hold on an [NPC](../npcs.md) when a [scene](../../glossary.md#scene) (one loaded area of the game) loads or the game saves.

- A scene load releases every [claim](../../glossary.md#claim) (your mod's hold on an NPC). After a single-scene load, every claim is released and every value still alive is put back. If you still need the NPC, take it again from [`Events.WhenReady`](../core-helpers/events-whenready.md).
- A `Take` called straight from an `Events.SceneLoaded` handler is refused. The load gate is set for the whole of that event, and `Take` returns `false` while it is set.
- Saving does not change anything the framework holds. `Take` is refused while the save runs, and `Release` still works.
