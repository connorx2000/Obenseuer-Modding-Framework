---
title: SpawnSaved
parent: Mod NPCs
grand_parent: People
nav_order: 6
---

# SpawnSaved

[`SpawnSaved`](types-and-members.md) adds a [mod NPC](../../glossary.md#mod-npc) (a copy of an existing person) like [`Spawn`](spawn.md). It is for a person who must be there again after the player loads a save. Without it, a person goes away with the [scene](../../glossary.md#scene).

## Things to know

- The framework remembers the request and adds the person again after each load of that scene. It writes the request into its own file in the save folder. Once the save is ready, it spawns the person again. <span class="tech">(the file is `mods/obenseuer.moddingframework.modnpcs.json` in the save folder; the game's own save holds nothing about the copy, as for `Spawn`)</span>
- The person comes back with the same `npcId`, source, body, name, position, yaw and dialogue setting. The body is stored as the number you passed, so a default body is chosen again at each load. <span class="tech">(a file from before `body` existed loads with body `0`)</span>
- Only people from installed mods come back. A mod is installed when it is in the game's [plugins](../../glossary.md#plugin) or has called `SpawnSaved` this session.
- `Despawn` and `Clear` remove the record too.
- A `SpawnSaved` for an `npcId` that has a record in another scene replaces that record.
- The look is saved as well. The look and blend shape changes are written at each game save and when the person is released. They are applied at each respawn (see [Look and shape](look-and-shape.md)).
- A person with no saved look starts from the source's look. <span class="tech">(the saved request holds the mod, id, source, body, name, position, yaw, scene, dialogue setting and the look difference)</span>

## Limits

- `scene` must be the name of the scene that is loaded now: the name `Events.SceneLoaded` gives you. Otherwise the call returns `0`. The other arguments and the reasons for a refusal are those of [`Spawn`](spawn.md#limits).
- The person stands where it was added, not where it walked to.
- A record from a mod that is not installed is kept. The person comes back when the mod is installed again.
- If a person can't be built at a load, it is tried again at the next load. That happens when there is no ground under the spot, the source is gone, or the body is unknown or has no model.
- Each mod keeps at most 256 saved people per save. Other mods' records are not counted.
- `SpawnSaved` returns `0` while the same `npcId` is on the stage now.
- Changes to the meshes and materials last for the session only.

**Common problems**

- `SpawnSaved` returns `0`: `scene` is not the name of the loaded scene, or the same `npcId` is on the stage now. Any refusal reason of `Spawn` also applies.
- A saved person does not come back after a load: the mod is not installed, or the person could not be built. A build fails when there is no ground under the spot, the source is gone, or the body is unknown or has no model. The record is kept. The person comes back when the mod is installed again, and a failed build is tried again at the next load.
