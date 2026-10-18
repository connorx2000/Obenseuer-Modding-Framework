---
title: Common tasks
parent: World item events
grand_parent: Items and crafting
nav_order: 1
---

# Common tasks

Short code recipes for [`WorldItems`](events.md): watching item uses, finding and removing one world item, and changing how long new items last.

This page uses `GameObject` from Unity. The [glossary](../../glossary.md#types-that-come-from-unity) says what each one is.
{: .unity-types}

Log each time the player uses a held item.

```csharp
WorldItems.Used += args => Log.Info("Used item " + args.ItemId);
```

Find the world item under a game object and take its [id](../../glossary.md#object-id) (a piece of text that names it). Then look the item up again by that id, and remove it. [World item ids](world-item-ids.md) explains the ids.

```csharp
GameObject target = GameObject.Find("Some Item");
string id = target == null ? null : WorldItems.IdOf(target);
if (id != null)
{
    GameObject again = WorldItems.Find(id);
    if (again != null) WorldItems.Remove(id);
}
```

Make new items last 30 minutes on average. Set it again after every load, because the game does not keep it. [Expiry default](expiry-default.md) has the rules for this timer.

```csharp
Events.SceneLoaded += scene => WorldItems.SetExpiryOverride(Mod.Id, 30);
```

Put the game's own timer back.

```csharp
WorldItems.ClearExpiryOverride(Mod.Id);
```
