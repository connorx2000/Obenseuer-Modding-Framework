---
title: Routines
parent: Mod NPC routines
grand_parent: People
nav_order: 3
---

# Routines

A routine is a weekly timetable for one [mod NPC](../../glossary.md#mod-npc) (a new person your mod adds). This page lists the types that build, store and read routines, and the rules the framework follows to run them.

This page uses `Vector3` from Unity. The [glossary](../../glossary.md#types-that-come-from-unity) says what each one is.
{: .unity-types}

You build a routine with `NpcRoutine`, then store it with `ModNpcRoutines.Set`. A routine lasts for the game session only. It is not written to the player's save, so you set it again each time the game starts.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>ModNpcRoutines</code> <span>stores, removes and reads routines</span></p>

Call `Set` with your mod's id, the NPC's id and an `NpcRoutine` to store a timetable. `Remove` or `Clear` takes it away. `Current` tells you which step the NPC is in.

<div class="api-pair" markdown="1">

```csharp
public static partial class ModNpcRoutines
{
    public static event Action<NpcStepArgs> StepChanged;
    public static bool Set(string modId, string npcId, int sourceCharacterId, string name, string lastName, NpcRoutine routine, bool keepSourceDialogue = false);
    public static bool Remove(string modId, string npcId);
    public static bool Clear(string modId);
    public static NpcStepKind Current(string modId, string npcId);
}
```

| Method | Returns | What it does |
|---|---|---|
| `Set(modId, npcId, sourceCharacterId, name, lastName, routine, keepSourceDialogue)` | `bool` | Stores a timetable for an NPC. |
| `Remove(modId, npcId)` | `bool` | Removes one timetable and its NPC. |
| `Clear(modId)` | `bool` | Removes all of your mod's timetables. |
| `Current(modId, npcId)` | `NpcStepKind` | The step the clock says the NPC is in now. |

| Event | Gives you | When it fires |
|---|---|---|
| `StepChanged` | `NpcStepArgs` | An NPC's timetable has moved to a different step. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>NpcStepArgs</code> <span>what you are told when the NPC changes step</span></p>

`StepChanged` gives you one of these when an NPC moves to another step. `From` and `To` are the old and new steps. `ModId` and `NpcId` say which NPC.

<div class="api-pair" markdown="1">

```csharp
public sealed class NpcStepArgs
{
    public string ModId { get; }
    public string NpcId { get; }
    public NpcStepKind From { get; }
    public NpcStepKind To { get; }
    public string Scene { get; }
    public int CharacterId { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `ModId` | `string` | Your mod's id. |
| `NpcId` | `string` | The id you gave the NPC. |
| `From` | `NpcStepKind` | The step it was in. |
| `To` | `NpcStepKind` | The step it is in now. |
| `Scene` | `string` | The scene the new step is in. |
| `CharacterId` | `int` | The NPC's character number. 0 when it is not in the loaded scene. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>NpcRoutine</code> <span>a weekly timetable, built one step at a time</span></p>

Build one with `new NpcRoutine()`, then add steps with `Stand`, `Wander`, `Patrol`, `Sit` or `Sleep`. Hand it to `ModNpcRoutines.Set` to store it.

<div class="api-pair" markdown="1">

```csharp
public sealed class NpcRoutine
{
    public NpcRoutine();
    public int Count { get; }
    public NpcRoutine Stand(NpcDays days, TimeSpan from, TimeSpan to, string scene, Vector3 point, float yaw);
    public NpcRoutine Wander(NpcDays days, TimeSpan from, TimeSpan to, string scene, Vector3 center, float radius);
    public NpcRoutine Patrol(NpcDays days, TimeSpan from, TimeSpan to, string scene, IList<Vector3> points);
    public NpcRoutine Sit(NpcDays days, TimeSpan from, TimeSpan to, string scene, Vector3 seat);
    public NpcRoutine Sleep(NpcDays days, TimeSpan from, TimeSpan to, string scene, Vector3 bed);
    public NpcRoutine Door(string scene, Vector3 point);
}
```

| Method | Returns | What it does |
|---|---|---|
| `NpcRoutine()` | `NpcRoutine` | Makes an empty timetable. |
| `Stand(days, from, to, scene, point, yaw)` | `NpcRoutine` | Adds a step where the NPC stands at a point, facing `yaw`. |
| `Wander(days, from, to, scene, center, radius)` | `NpcRoutine` | Adds a step where the NPC wanders around a spot. |
| `Patrol(days, from, to, scene, points)` | `NpcRoutine` | Adds a step where the NPC patrols along points. |
| `Sit(days, from, to, scene, seat)` | `NpcRoutine` | Adds a step where the NPC sits on the chair near this point. |
| `Sleep(days, from, to, scene, bed)` | `NpcRoutine` | Adds a step where the NPC sleeps in the bed near this point. |
| `Door(scene, point)` | `NpcRoutine` | Sets where the NPC walks in and out of this scene. |

| Property | Type | What it means |
|---|---|---|
| `Count` | `int` | How many steps it has. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>NpcStepKind</code> <span>what the NPC is doing in a step</span></p>

`Current` gives you one of these, and so do `NpcStepArgs.From` and `To`.

<div class="api-pair" markdown="1">

```csharp
public enum NpcStepKind
{
    Away,
    Stand,
    Wander,
    Patrol,
    Sit,
    Sleep
}
```

| Value | Meaning |
|---|---|
| `Away` | The NPC is in no scene. |
| `Stand` | Standing at a point. |
| `Wander` | Wandering around a spot. |
| `Patrol` | Walking between points. |
| `Sit` | Sitting on a chair. |
| `Sleep` | Sleeping in a bed. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>NpcDays</code> <span>the days a step covers</span></p>

Pass one of these to each step method. It is a flags value (several days can be combined), so `NpcDays.Monday | NpcDays.Friday` is a valid set.

<div class="api-pair" markdown="1">

```csharp
[Flags]
public enum NpcDays
{
    None = 0,
    Monday = 1,
    Tuesday = 2,
    Wednesday = 4,
    Thursday = 8,
    Friday = 16,
    Saturday = 32,
    Sunday = 64,
    Weekdays = 31,
    Weekend = 96,
    Every = 127
}
```

| Value | Meaning |
|---|---|
| `None` | No days. Not allowed in a step. |
| `Monday` | Monday. |
| `Tuesday` | Tuesday. |
| `Wednesday` | Wednesday. |
| `Thursday` | Thursday. |
| `Friday` | Friday. |
| `Saturday` | Saturday. |
| `Sunday` | Sunday. |
| `Weekdays` | Monday to Friday. |
| `Weekend` | Saturday and Sunday. |
| `Every` | Every day of the week. |

</div>
</div>

## Building a routine

- Each step method adds one step and gives the routine back, so you can chain calls. A step covers the time from `from` up to `to` on each day in `days`. Seconds are dropped, so a step works in whole minutes.
- `days` is a set of days. The week starts on Monday, like the game's own calendar. `NpcDays.None` is not allowed.
- `from` must be 00:00 to 23:59, and `to` must be 00:01 to 24:00. They must not be the same. If `to` is earlier than `from`, the step goes past midnight. The part after midnight belongs to the start day. For example, a Sunday 23:00 to 02:00 step is active on Sunday night only, and its part after midnight covers Monday 00:00 to 02:00.
- When steps overlap, the step you added first wins. This is how the game's own timetable works.
- A minute that no step covers is `Away`. The NPC is in no [scene](../../glossary.md#scene) at that time.
- Wrong values throw an `ArgumentException`. The scene name must not be empty. Points, centres, seats, beds, doors and `yaw` must be real numbers. `radius` must be 0.5 to 50. A patrol needs 2 to 32 points. A routine holds at most 64 steps.
- `Door(scene, point)` says where an NPC walks in and out of a scene. Each scene has one door point, and calling `Door` again for the same scene replaces the first. A door is not a level change. The NPC walks to the door and disappears, then appears in the other scene when that scene's step says so.

## Setting and removing

- A routine spawns its NPC the same way `ModNpcs.Spawn` does. If the source character has no body of its own, the NPC gets the default body. See [Mod NPCs](../modnpcs/spawn.md#spawn).
- `Set` stores a copy of the routine for your mod and the NPC id. Changing your `NpcRoutine` afterwards does not change the stored copy. Calling `Set` again with the same id replaces the routine. You can call it at any time, even on the main menu. The NPC appears once a save is loaded and its scene is active.
- `Set` throws an `ArgumentException` for wrong values. These are:
  - an invalid `modId` (see [Mod id](../../glossary.md#mod-id));
  - an `npcId` that is empty or longer than 64 characters;
  - an empty `name`;
  - a `null` routine, or a routine with no steps.
- `Set` returns `false` when it did not store the routine. That happens in these cases:
  - mod NPCs are off (`IsAvailable` is `false`);
  - your mod already has 64 routines;
  - an NPC that no routine started already uses the id;
  - a saved mod NPC (from [`ModNpcs.SpawnSaved`](../modnpcs/spawnsaved.md)) uses the id, even if that NPC is not in the loaded scene. So never use the same id for `SpawnSaved` and for a routine.
- `Remove` forgets the routine and removes its NPC if a routine spawned it. `Clear` does the same for every routine of your mod. Both return `false` when there was nothing to remove.
- `Current` tells you the step the game clock puts the NPC in, whether or not the NPC is in the loaded scene. It gives `Away` for an id it does not know.
- `StepChanged` fires once when the step in effect changes. Loading a scene alone does not fire it. The first check after `Set` always fires it, so `From` and `To` can both be `Away`.

**Common problems**

- `Set` returns `false`: mod NPCs are off, your mod already has 64 routines, or another NPC already uses the id. That NPC can be one that no routine started, or a saved mod NPC from `SpawnSaved`. Never use the same id for `SpawnSaved` and for a routine. If `IsAvailable` is `false`, see [Details](details.md#if-something-fails-to-set-up).
- `Set` throws an `ArgumentException`: one of the values in the `Set` bullet above is wrong.
- A step method throws an `ArgumentException`: one of the values listed under Building a routine is wrong.

## What the framework does each minute

The framework checks each routine once for every in-game minute, after a scene loads, and after `Set`. This is what it does for the [scene](../../glossary.md#scene) that is loaded:

- If the step in effect is this scene and the NPC is not in this scene, the NPC appears at the step's point. If the step changed while the player was already in this scene, it appears at the door and walks in. This needs a door point in the scene.
- If the step in effect is this scene and the NPC is in this scene, the step's order starts. That does not happen while a one-off order or the routine's own order is already running. A new step ends the routine's previous order first.
- If the step in effect is another scene or `Away`, an NPC in this scene that a routine spawned leaves. First it waits while someone is talking to it. If this scene has a door point, the NPC walks there and is then removed. Otherwise it is removed at once.
- If the step in effect is another scene or `Away` and the NPC is not in this scene, nothing happens.

- A routine step walks by the same rules as the [orders](orders.md). A one-off order (`WalkTo`, `Wander`, `Patrol`, `Sit` or `Sleep`) pauses the routine until the order ends. A step change does not end it, and the NPC does not leave its scene while it runs. `Stop` ends any order, including the routine's own. The routine then starts its step again at the next check, one in-game minute later.
- A routine walk that runs out of time moves the NPC to the step's point. This keeps the routine on time.
- A `Sit` or `Sleep` step with no chair or bed in range stands the NPC at the step's point instead. One line goes to the log.
- A routine NPC that dies stays dead until its scene loads again. It then appears alive at the step's point. Call `Remove` to stop it for good.
- A routine NPC cannot go through a locked door, the same as for orders.
- Nothing is saved for a routine. After a reload the NPC is placed where the clock says it should be. A new game works the same way once its first level has loaded.
- If the NPC has not started its first order within 300 frames of appearing, the framework tries again each in-game minute.
