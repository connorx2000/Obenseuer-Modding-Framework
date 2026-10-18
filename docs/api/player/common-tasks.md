---
title: Common tasks
parent: Player
grand_parent: Player life and skills
nav_order: 1
---

# Common tasks

Short snippets for frequent player jobs: skills, effects, damage, god mode, subscriptions and travel. Each one runs after a save has loaded, for example inside `Events.WhenReady(() => { ... })` (see [Events.WhenReady](../core-helpers/events-whenready.md)).

Give the player half a level of Cooking. [`Add`](skills.md) returns `false` when the game refuses, for example at the maximum level.

```csharp
Skills.Add(Skill.Cooking, 0.5f);
```

Check whether the player is drunk right now, with [`Effects`](effects.md).

```csharp
bool drunk = Effects.IsActive(Effect.Drunk);
```

Hurt the player. The reason is the kind of damage ([`DamageReason`](playerstate.md)). The game's normal sounds and protection rules apply.

```csharp
PlayerState.Damage(10f, DamageReason.Fall);
```

Make the player unkillable for a while. Pass your mod's id as the [owner](../../glossary.md#owner) (the mod id that holds a change), so your claim does not clash with another mod's. Release the claim when you are done.

```csharp
PlayerState.SetGodMode("example.playerbasics", true);
PlayerState.SetGodMode("example.playerbasics", false);
```

Check a [subscription](subscriptions.md) and list the level's [named spawn points](../../glossary.md#entrypoint), then send the player to one with [`Travel`](travel.md).

```csharp
bool has = Subscriptions.IsActive("some-id");
foreach (string name in Travel.Entrypoints()) Log.Info(name);
Travel.TeleportTo("some-name");
```
