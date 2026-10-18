---
title: Finding NPCs
parent: NPCs
grand_parent: People
nav_order: 2
---

# Finding NPCs

This page covers the `Npcs` calls that list NPCs and look one up by id or name, and the events for spawns, despawns and deaths. It also describes the `NpcInfo` snapshot they give you.

This page uses `Vector3` from Unity. The [glossary](../../glossary.md#types-that-come-from-unity) says what each one is.
{: .unity-types}

Each `NpcInfo` is a snapshot. See also [Characters](../characters.md).

<div class="api-type" markdown="1">
<p class="api-type-head"><code>Npcs</code> <span>NPC list and events</span></p>

Lists the NPCs the game knows about, looks one up by `Id` or by name, and tells you when one spawns, despawns or dies.

<div class="api-pair" markdown="1">

```csharp
public static class Npcs
{
    public static List<NpcInfo> All();
    public static NpcInfo Get(int id);
    public static NpcInfo Find(string name);
    public static bool IsAvailable { get; }
    public static event Action<NpcInfo> Spawned;
    public static event Action<NpcInfo> Despawned;
    public static event Action<NpcInfo> Died;
}
```

| Method | Returns | What it does |
|---|---|---|
| `All()` | `List<NpcInfo>` | Every NPC the game knows about. |
| `Get(id)` | `NpcInfo` | The NPC with this id. `null` if there is none. |
| `Find(name)` | `NpcInfo` | The first NPC whose first or full name matches. `null` if none does. Throws if the name is empty. |

| Property | Type | What it means |
|---|---|---|
| `IsAvailable` | `bool` | `false` when the spawn, despawn or death events are off. |

| Event | Gives you | When it fires |
|---|---|---|
| `Spawned` | `NpcInfo` | An NPC appears. |
| `Despawned` | `NpcInfo` | An NPC leaves the scene. |
| `Died` | `NpcInfo` | Once, when an NPC dies. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>NpcInfo</code> <span>one NPC snapshot</span></p>

`Npcs.All()`, `Npcs.Get` and `Npcs.Find` give you these, and each of the three events hands you one. It is a copy taken when it is made. Call `Get` or `Find` again to see a change. <span class="tech">(It is built from the game's `NPCData` and its `Character`.)</span>

<div class="api-pair" markdown="1">

```csharp
public sealed class NpcInfo
{
    public int Id { get; }
    public string Name { get; }
    public string DisplayName { get; }
    public Vector3 Position { get; }
    public bool Alive { get; }
    public string CurrentScene { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `Id` | `int` | The number you pass to `Npcs.Get`. |
| `Name` | `string` | The first name. `null` if there is none. |
| `DisplayName` | `string` | The full name. `null` if there is none. |
| `Position` | `Vector3` | The live position, or the last known one if the NPC is not active in the scene. |
| `Alive` | `bool` | `false` only for a dead NPC that is in a scene. |
| `CurrentScene` | `string` | The scene the game reports. `null` if it is unknown. |

</div>
</div>

## Putting it together

This mod logs each spawn. Once a save is loaded, it finds one NPC by name and logs where it is.

<div class="api-example" markdown="1">

```csharp
using ObenseuerCore;

[ModInfo("example.npcroster", "NPC Roster", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class NpcRoster : FrameworkMod
{
    protected override void OnLoad()
    {
        Npcs.Spawned += npc => Log.Info("Spawned: " + (npc.DisplayName ?? "an NPC with no character") + " in " + npc.CurrentScene);
        Events.WhenReady(() =>
        {
            NpcInfo npc = Npcs.Find("Some Name");
            if (npc == null)
            {
                Log.Info("Nobody has that name.");
                return;
            }
            NpcInfo exact = Npcs.Get(npc.Id);
            if (exact != null)
                Log.Info(exact.Name + " is at " + exact.Position + " in " + exact.CurrentScene);
        });
    }
}
```

| Step | What it does |
|---|---|
| `Npcs.Spawned` | Logs each spawn. |
| `Npcs.Find("Some Name")` | Returns the first match, or `null` if there is none. |

</div>

## Things to know

- `All()` misses NPCs placed straight into a [scene](../../glossary.md#scene). Use [`InScene()`](loaded-scenes.md) for those.
- `Spawned` fires before a `Character.Spawn` NPC starts. Both `Spawned` and `Despawned` fire during save loads.
- `Died` does not repeat on load. `Despawned` ignores NPCs that were only switched off because they walked away.

## Limits

- The game forgets deaths of away NPCs, so `Died` misses them and `Alive` stays `true`.
- In 0.5.08 the game never kills NPCs, so `Alive` changes mostly when a mod kills one.
- Two NPCs can share a name. Use `Get(id)` to pick the right one.

**Common problems**

- An NPC is missing from `All()`: it was placed straight into a scene. Find it with `InScene()`.
- `Find` returns a different NPC with the same name: two NPCs can share a name. Use `Get(id)`.
