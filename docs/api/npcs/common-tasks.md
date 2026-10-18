---
title: Common tasks
parent: NPCs
grand_parent: People
nav_order: 1
---

# Common tasks

Short examples for the most common jobs with NPCs, relationships, animals and the phone book. Each one writes its result to the [log](../../guides/getting-started/logging.md).

Find an NPC by name with [`Npcs`](finding-npcs.md) and log where it is.

```csharp
NpcInfo npc = Npcs.Find("Some Name");
if (npc != null) Log.Info(npc.Name + " is at " + npc.Position + " in " + npc.CurrentScene);
```

List every relationship with [`Relationships`](../relationships.md), with its disposition.

```csharp
foreach (RelationshipValue relationship in Relationships.All())
    Log.Info(relationship.Name + ": " + relationship.Disposition);
```

Log each change to a relationship.

```csharp
Relationships.RelationshipChanged += change => Log.Info(change.Name + " went from " + change.Previous + " to " + change.Current);
```

Count the animals that are following the player, with [`Animals`](../animals.md).

```csharp
int following = 0;
foreach (AnimalInfo animal in Animals.All())
    if (animal.Following) following++;
Log.Info(following + " animals are following you.");
```

List the numbers in the phone book with [`Phones`](../phones.md).

```csharp
foreach (PhoneContact contact in Phones.All())
    Log.Info(contact.Id + ": " + contact.Number);
```
