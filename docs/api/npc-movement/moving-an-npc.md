---
title: Moving an NPC
parent: NPC movement
grand_parent: People
nav_order: 2
---

# Moving an NPC

This page lists every call of `NpcMovement`, which moves an [NPC](../npcs.md) (a character in the game), holds it still and hands it back. Taking over an NPC's movement is called holding it.

This page uses `Vector3` from Unity. The [glossary](../../glossary.md#types-that-come-from-unity) says what each one is.
{: .unity-types}

Text in brackets at the end of a note is extra detail for experienced modders. You can skip it on a first read. Every call takes `modId`, the id of your mod. The framework uses it to remember which mod holds which NPC (see [Owner](../../glossary.md#owner) in the glossary).

<div class="api-type" markdown="1">
<p class="api-type-head"><code>NpcMovement</code> <span>move, hold and release an NPC</span></p>

`MoveTo`, `Patrol` and `Stand` give the NPC a job. `SetActivityEnabled` holds it still. `Clear` ends everything your mod has on it.

<div class="api-pair" markdown="1">

```csharp
public static class NpcMovement
{
    public static bool SetActivityEnabled(string modId, int npcId, bool enabled);
    public static bool MoveTo(string modId, int npcId, Vector3 position);
    public static bool Patrol(string modId, int npcId, string nodeGroupId);
    public static bool Stand(string modId, int npcId, string nodeGroupId);
    public static bool ForceStopActivity(string modId, int npcId);
    public static bool Clear(string modId, int npcId);
}
```

| Method | Returns | What it does |
|---|---|---|
| `SetActivityEnabled(modId, npcId, enabled)` | `bool` | Holds the NPC still when `enabled` is `false`. Gives your hold back when it is `true`. |
| `MoveTo(modId, npcId, position)` | `bool` | Walks the NPC to a spot in the world. |
| `Patrol(modId, npcId, nodeGroupId)` | `bool` | Makes the NPC patrol a group of waypoints. |
| `Stand(modId, npcId, nodeGroupId)` | `bool` | Makes the NPC stand at a group of waypoints. |
| `ForceStopActivity(modId, npcId)` | `bool` | Stops what the NPC is doing now, whoever started it. |
| `Clear(modId, npcId)` | `bool` | Ends everything your mod has on this NPC. |

</div>
</div>

## Things to know

- `SetActivityEnabled(modId, npcId, false)` holds the NPC still. The game stops what the NPC is doing. While the hold lasts, the [schedule](../npc-schedules.md) does not start or change the NPC's activity. It returns `false` for an NPC that is not in the loaded [scene](../../glossary.md#scene). <span class="tech">(The NPC's controller does not update while the hold lasts.)</span>
- `SetActivityEnabled(modId, npcId, true)` gives your hold back. It returns `false` when your mod holds nothing. The NPC's schedule goes on again once no mod holds it any more. The framework removes your own [claim](../../glossary.md#claim) (your mod's hold on the NPC) before it checks, so the release does not wait for itself.
- `MoveTo` walks the NPC to `position` by the game's own pathfinding. It also holds the NPC, so the schedule can't pull it back. It returns `false` for an NPC that can't walk, because it has no path-following part. <span class="tech">(It needs a `RichAI` agent. The walk path was read from the code, not observed in the game.)</span>
- `Patrol` and `Stand` start the game's own patrol or stand activity for a group of waypoints. `nodeGroupId` is the id written on the group in the scene. A group with no id can't be named, and the framework has no call that lists the groups. A patrol needs a group that has waypoints. A stand needs a group that points at a waypoint in the loaded scene. <span class="tech">(`nodeGroupId` is the `GUID` field of the group's `NodeGroup` component. A stand needs a `nodeReference` that points at a node in the loaded scene.)</span>
- `Patrol` and `Stand` refuse while any mod holds the NPC. Release the hold first. The game may still interrupt a patrol or stand that it marks as interruptible. That is the game's own schedule rule.
- `ForceStopActivity` stops the NPC's current activity, whoever started it. The schedule takes over again at its next check. It returns `false` when the NPC has no current activity.
- `Clear(modId, npcId)` ends everything your mod has on that NPC. It stops the patrol or stand your mod started, if that is still what the NPC is doing. It also gives your hold back. It returns `false` when your mod has no claim on the NPC.

## Limits

- `Clear` does not cancel a walk that `MoveTo` started. The NPC finishes the walk, or the schedule's next activity sets a new target.

**Common problems**

- `Patrol` or `Stand` does not start: a patrol needs a group that has waypoints, and a stand needs a group that points at a waypoint in the loaded scene. Both also refuse while any mod holds the NPC, and your own `MoveTo` holds it too. Release your own hold with `SetActivityEnabled(modId, npcId, true)` or `Clear(modId, npcId)`.
