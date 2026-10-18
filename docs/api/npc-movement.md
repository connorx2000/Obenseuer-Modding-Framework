---
title: NPC movement
parent: People
grand_parent: API reference
has_children: true
nav_order: 9
---

# NPC movement

[`NpcMovement`](npc-movement/moving-an-npc.md) lets your mod take over where an [NPC](npcs.md) goes for a while, and then hand it back to its schedule. Use it to walk an NPC somewhere, make it patrol or stand, or keep it where it is.

This page uses `Vector3` from Unity. The [glossary](../glossary.md#types-that-come-from-unity) says what each one is.
{: .unity-types}

Status: experimental.

## What it is

Every NPC (a character in the game) follows a daily [schedule](npc-schedules.md). The schedule gives the NPC an activity at each time of day. Examples are walking to a place, standing at a spot or patrolling a route.

This API lets your mod take over an NPC's movement for a while. Your mod can:

- walk the NPC to a position;
- make it patrol or stand at a group of waypoints;
- switch its schedule off, so it stays where it is.

When you are done, `Clear` hands the NPC back to its normal schedule.

NPC ids on this page are the ones from the [NPCs](npcs.md) page (`NpcInfo.Id`, `Npcs.All()`). Use that page to choose an NPC and read its state.

## When to use it

- Walk an NPC to a place for a quest or a scripted scene.
- Make an NPC patrol a route or stand at a spot.
- Freeze an NPC where it is, so the schedule does not pull it away.
- Stop whatever an NPC is doing right now.

## Quick example

This mod waits for a save to load. It then walks the first living NPC it can to a spot 5 metres away. After 20 seconds it gives the NPC back to its schedule.

```csharp
using ObenseuerCore;
using UnityEngine;

[ModInfo("example.npcwalk", "NPC Walk", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class NpcWalk : FrameworkMod
{
    private int walkingNpc = -1;
    private float clearAt;

    protected override void OnLoad()
    {
        Events.WhenReady(StartWalk);
    }

    private void StartWalk()
    {
        foreach (NpcInfo npc in Npcs.All())
        {
            if (!npc.Alive) continue;
            Vector3 target = npc.Position + new Vector3(5f, 0f, 0f);
            if (!NpcMovement.MoveTo(Mod.Id, npc.Id, target)) continue;
            walkingNpc = npc.Id;
            clearAt = Time.time + 20f;
            Log.Info("Walking " + (npc.Name ?? "an NPC") + " to " + target);
            return;
        }
        Log.Info("No NPC could be moved.");
    }

    private void Update()
    {
        if (walkingNpc < 0 || Time.time < clearAt) return;
        NpcMovement.Clear(Mod.Id, walkingNpc);
        walkingNpc = -1;
    }
}
```

`MoveTo` returns `false` for an NPC that is not in the loaded [scene](../glossary.md#scene), so the example tries each NPC until one works. `Mod.Id` is your mod's own [mod id](../glossary.md#mod-id). It is how the framework knows the [claim](../glossary.md#claim) (your mod's hold on the NPC) is yours. See [Using events](../guides/getting-started/using-events.md) for more on `WhenReady`.
