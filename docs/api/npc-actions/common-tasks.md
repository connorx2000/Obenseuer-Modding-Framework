---
title: Common tasks
parent: NPC actions
grand_parent: People
nav_order: 1
---

# Common tasks

Short snippets for the usual [NPC actions](../npc-actions.md): moving an NPC, making it follow the player, starting a conversation, sending it to another scene, and hitting or killing it.

This page uses `Vector3` from Unity. The [glossary](../../glossary.md#types-that-come-from-unity) says what each one is.
{: .unity-types}

In these snippets, `npcId` is the number from [`NpcInfo.Id`](../npcs/finding-npcs.md). `conversationTitle` is the title of a conversation in the game, and `sceneName` is the name of a [scene](../../glossary.md#scene) (one loaded area of the game).

Move an NPC to a spot with [`Teleport`](moving-and-hurting.md). Its routine keeps running, so the NPC may walk back toward what it was doing.

```csharp
bool moved = NpcActions.Teleport(npcId, new Vector3(10f, 0f, 5f));
```

Make an NPC follow the player, then let it go. Pass `false` as the last argument of [`FollowPlayer`](following-the-player.md), so the NPC's own routine can take it back.

```csharp
if (NpcActions.FollowPlayer(npcId, false, false)) Log.Info("The NPC is following you.");
NpcActions.StopFollowing(npcId);
```

Start a conversation the game already has, by its title, with [`StartDialogue`](speaking.md).

```csharp
bool started = NpcActions.StartDialogue(npcId, conversationTitle);
```

Send an NPC who is not in the scene to another scene with [`ChangeLevel`](sending-an-npc-to-another-scene.md). The scene name must be one the game has.

```csharp
bool sent = NpcActions.ChangeLevel(npcId, sceneName);
```

Hit an NPC the way a melee hit does, or kill it.

```csharp
NpcActions.Hit(npcId);
NpcActions.Kill(npcId);
```
