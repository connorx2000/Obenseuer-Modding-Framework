---
title: Under the hood
parent: NPCs
grand_parent: People
nav_order: 5
---

# Under the hood

How the framework reads the game for the NPC API, and which feature groups it uses. This page is for modders who want to dig deeper.

This page uses `Transform` from Unity. The [glossary](../../glossary.md#types-that-come-from-unity) says what each one is.
{: .unity-types}

## Npcs

- `NPCDirector.globalDatabase` holds rows from `StreamingAssets/NPCBehavior.json`.
- `NPCManager.AddNpcToList` raises `Spawned`, before `Start` runs. `NPCManager.RemoveNpcFromList` raises `Despawned`.
- `NPCInfo.ChangeState` raises `Died` on a move to `Dead`. Only `NPCInfo.Kill` sets `Dead`, and nothing in the game calls it.
- `Position` is the live [transform](../../glossary.md#transform). Otherwise it is `NPCData.state.currentPosition`.
- `UpdateFade` hides NPCs farther than `hideDistance` (20). `hideAlways` (`ChangeHiddenStatus`) hides them at any distance.
- `FadeGameObject.Hide` and `FadeGameObject.Show` call `SetActive`. Culling sets only `culled`. `FadeGameObjectController.ShowAll` runs on save.
- `Present` reads `isHidden`, `hideAlways`, the parent and `CurrentScene`, and `culled` via `AccessTools.Field`.
- `InScene()` lists `NPCInfo` components. `IsDirector` checks for a database entry.
- `Handle` is the component's `GUID` field. Otherwise it is `scene:Name[index]/...`. Copies of a prefab can share it.

## Feature groups

- Spawn and despawn use `obenseuer.moddingframework.npcs`. Death uses `obenseuer.moddingframework.npcs.died`. Relationships use `obenseuer.moddingframework.npcs.relationship`. Each is a [feature group](../../glossary.md#feature-group) (a named set of the framework's changes to the game).
