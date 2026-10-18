---
title: Common tasks
parent: Mod NPCs
grand_parent: People
nav_order: 1
---

# Common tasks

Short snippets for the usual [`ModNpcs`](types-and-members.md) jobs: adding, finding and removing [mod NPCs](../../glossary.md#mod-npc) (copies of existing people), adding background figures, and giving a person a look from the game's files.

This page uses `Vector3`, `GameObject` from Unity. The [glossary](../../glossary.md#types-that-come-from-unity) says what each one is.
{: .unity-types}

The snippets use a source character number (`sourceId`), a body character number (`bodyId`) and a position (`spot`) that you already have.

Add a person who comes back after every load, with [`SpawnSaved`](spawnsaved.md). Then look up their character number and list all the people your mod has added.

```csharp
string scene = SceneManager.GetActiveScene().name;
int saved = ModNpcs.SpawnSaved(Mod.Id, "shopkeeper", sourceId, bodyId, "Sam", "Stone", spot, 90f, scene, false);

int characterId = ModNpcs.CharacterIdOf(Mod.Id, "shopkeeper");
foreach (ModNpcInfo npc in ModNpcs.List(Mod.Id))
    Log.Info(npc.DisplayName + " is at " + npc.Position + ", alive: " + npc.Alive);
```

Remove one person, or all of them.

```csharp
ModNpcs.Despawn(Mod.Id, "shopkeeper");
ModNpcs.Clear(Mod.Id);
```

Add a background figure (an [extra](extras.md)), take it away again, and remove all your figures.

```csharp
GameObject extra = ModNpcs.SpawnExtra(Mod.Id, "crowd-1", sourceId, bodyId, spot + Vector3.right, 180f);
if (extra != null) ModNpcs.DespawnExtra(Mod.Id, "crowd-1");
int removed = ModNpcs.ClearExtras(Mod.Id);
```

Add a person with a look taken from the game's own files, with [`SpawnWithLook`](spawnwithlook.md). The look comes from [Scene looks](../scenelooks.md).

```csharp
SceneLookInfo info = SceneLooks.FindByCharacter(sourceId);
if (info != null)
{
    SceneLook look = SceneLooks.Load(Mod.Id, info);
    ModNpcs.SpawnWithLook(Mod.Id, "lookalike", sourceId, bodyId, look, "Lou", "Lookalike", spot, 0f);
}
```
