---
title: Common tasks
parent: Scene looks
grand_parent: Places and travel
nav_order: 1
---

# Common tasks

Short [`SceneLooks`](putting-a-look-on-a-body.md) examples for the most common jobs. They put a look on a body your mod spawned or on an NPC already in the scene, and take a look off again.

This page uses `GameObject` from Unity. The [glossary](../../glossary.md#types-that-come-from-unity) says what each one is.
{: .unity-types}

Find a look by its name, load it, and put it on a body that your mod has spawned. `body` here is the NPC's object in the [scene](../../glossary.md#scene) (one loaded area of the game). `Mod.Id` is your mod's id (see the [glossary](../../glossary.md#mod-id)). When `Apply` refuses, it tells you why through `reason`.

```csharp
GameObject body = GameObject.Find("My Mod NPC");
SceneLookInfo info = SceneLooks.FindByName("Interior Tenement Speakeasy", "mr_duck");
if (body != null && info != null)
{
    SceneLook look = SceneLooks.Load(Mod.Id, info);
    string reason;
    if (!look.Ready)
        Log.Warn("Could not load the look: " + look.Reason);
    else if (!SceneLooks.Apply(Mod.Id, look, body, out reason))
        Log.Warn("Could not apply the look: " + reason);
    else
        Log.Info("The look is on the body.");
}
```

Put a look on an NPC that is already in the scene and moving about. `Reskin` does this, and it works the same as `Apply`.

```csharp
GameObject npc = GameObject.Find("Some Live NPC");
SceneLookInfo info = SceneLooks.FindByCharacter("Interior Tenement Speakeasy", 1);
if (npc != null && info != null)
{
    SceneLook look = SceneLooks.Load(Mod.Id, info);
    string reason;
    if (look.Ready && !SceneLooks.Reskin(Mod.Id, look, npc, out reason))
        Log.Warn("Reskin refused: " + reason);
}
```

Take the look off a body with `Remove`. Then let go of the look with `Release`, so its memory can be freed.

```csharp
GameObject body = GameObject.Find("My Mod NPC");
SceneLookInfo info = SceneLooks.FindByName("Interior Tenement Speakeasy", "mr_duck");
if (body != null && info != null)
{
    SceneLook look = SceneLooks.Load(Mod.Id, info);
    string reason;
    SceneLooks.Remove(Mod.Id, body, out reason);
    SceneLooks.Release(Mod.Id, look);
}
```
