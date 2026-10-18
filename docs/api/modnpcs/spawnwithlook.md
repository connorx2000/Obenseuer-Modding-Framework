---
title: SpawnWithLook
parent: Mod NPCs
grand_parent: People
nav_order: 4
---

# SpawnWithLook

[`SpawnWithLook`](types-and-members.md) adds a [mod NPC](../../glossary.md#mod-npc) (a copy of an existing person) like [`Spawn`](spawn.md) and gives them a look in the same call. It needs [framework](../../glossary.md#framework) 0.11.0 or later.

The look comes from [Scene looks](../scenelooks.md). Scene looks reads a character's look out of the installed game files while the game runs. So your mod does not need to ship any art.

## Things to know

- The arguments, the body rule and the reasons for a refusal are those of [`Spawn`](spawn.md). The version without `bodyCharacterId` passes `0`.
- A refusal returns `0` with no reason, as `Spawn` does. Read the reason in the [log](../../guides/getting-started/logging.md) file `BepInEx\logs\obenseuer.moddingframework.modnpcs.log`.
- If the look can't be applied, the person is taken out again. The call then returns `0`, writes the reason to the log once, and leaves nothing in the scene. A saved record from [`SpawnSaved`](spawnsaved.md) under the same id is left as it was.

## Limits

- A look that is not ready ([`SceneLook.Ready`](../scenelooks/loading-a-look.md) is `false`) fails the same way. Check `Ready` first when the look may not be installed.
- The default body is a different kind of skeleton, so some looks are refused. A look from a character such as Mr. Duck is built for a different skeleton and is very likely to need a conversion step. A look the framework can't put on the default body is refused. <span class="tech">(the default template is a Blender-style humanoid rig, and a `Bip001` body such as Mr. Duck's takes the retarget path)</span>
- The look is not saved. A person who must come back after a load uses [`SpawnSaved`](spawnsaved.md). Your mod then applies the look again from [`Events.WhenReady`](../core-helpers/events-whenready.md) with [`SceneLooks.Apply`](../scenelooks/putting-a-look-on-a-body.md).

**Common problems**

- `SpawnWithLook` returns `0`: one of the refusal reasons of `Spawn` applies, the look is not ready, or the look can't be applied. For example, a look the framework can't put on the default body is refused. Read the reason in the log file, and check `Ready` before the call.
- The person has lost the look after a load: the look is not saved. Use `SpawnSaved`, and apply the look again from `Events.WhenReady` with `SceneLooks.Apply`.
