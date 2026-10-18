---
title: Moving and hurting
parent: NPC actions
grand_parent: People
nav_order: 3
---

# Moving and hurting

How to move an NPC to a spot, kill it, or make it react to a hit. The page also lists what each call skips. Use these calls for scripted moments in the [scene](../../glossary.md#scene) (one loaded area of the game).

This page uses `Vector3` from Unity. The [glossary](../../glossary.md#types-that-come-from-unity) says what each one is.
{: .unity-types}

<div class="api-type" markdown="1">
<p class="api-type-head"><code>NpcActions</code> <span>moving and hurting an NPC</span></p>

`Teleport` moves the NPC to a spot, `Kill` makes it dead, and `Hit` makes it react to a hit.

<div class="api-pair" markdown="1">

```csharp
public static class NpcActions
{
    public static bool Teleport(int npcId, Vector3 position);
    public static bool Kill(int npcId);
    public static bool Hit(int npcId);
}
```

| Method | Returns | What it does |
|---|---|---|
| `Teleport(npcId, position)` | `bool` | Moves the NPC to a spot in the scene. |
| `Kill(npcId)` | `bool` | Stops the NPC and puts it in the dead state. |
| `Hit(npcId)` | `bool` | Makes the NPC react as if the player had hit it. |

</div>
</div>

## Things to know

- `Kill` ends what the NPC was doing and makes it dead. The body stays where it is, and the NPC stops moving and talking. The framework's [`Npcs.Died`](../npcs/finding-npcs.md) event fires, because it is the same change the game makes. The base game never kills an NPC this way, so only mods do it. <span class="tech">(It stops the current activity and the path, then calls the public `NPCInfo.Kill()`, which calls `ChangeState(State.Dead)`. The game's Dead state turns off the controller, head look, bark, interaction and field of view.)</span>
- `Kill` gives `false` for an NPC that is already dead. Killing it again would do nothing in the game, so it is not reported as done.
- `Hit` makes the NPC react the way the game does when the player hits it with a melee weapon, but it does no damage. It gives `true` once it has run all these steps, in this order:
  1. The NPC's "on hit" trigger fires. This is an output of the [scene logic](../scenelogic.md).
  2. The NPC says a line, unless it is set not to comment on hits. The line is its own hit line, or the game's `Generic/Hit` line when it has none. It is spoken as a [bark](speaking.md), or as a conversation when the NPC is set to start one.
  3. The player is given a violence [crime](../../api-story/law-experimental-for-arrest.md), unless the NPC is set not to give a crime on hits. The game then checks whether anyone saw it, through its [witnesses](../../glossary.md#witness) (components that let people or cameras see).
  4. The NPC plays its "hit" animation.
  5. A police officer starts to chase the player. A robber gives up the robbery. <span class="tech">(The steps follow `MeleeWeapon.Hit`: the `onHit` relay, `onHitDialogue` (as a bark, or as a dialogue with `startDialogueOnHit`), `Crime.instance.CommitCrime(npc.character.NPCReference(), 100, Crime.CrimeType.Violence)` then `Witness.IsWitnessed()`, the `hit` animator trigger, and `Act_Police.StartChase` or `Act_Robber.AbortRobbing` read from the NPC's current activity.)</span>
- `Hit` leaves out what belongs to the weapon. It plays no hit sound or effect and no first-person animation. It applies no push and does no damage. An NPC has no health that the game reads, so nothing about its health changes. <span class="tech">(The skipped parts are the weapon's sound and effect, the first-person animation, the rigidbody force and `IDamageReceiver` damage.)</span>

## Limits

- `Teleport` moves the NPC to `position`, but it does not stop what the NPC was doing, so the NPC may walk back toward its target. The spot is not moved onto the walkable ground, so give a spot that is on it. `Teleport` gives `false` when the NPC cannot be moved this way. <span class="tech">(It calls `NPCController.richAI.Teleport(position)`, the same call the game's forced movement uses, and gives `false` when the NPC has no `RichAI`. `position` is not snapped to the navmesh.)</span>
- A killed NPC comes back alive after a [scene](../../glossary.md#scene) change or a load. The dead state is kept only for the session and is not saved. The game keeps no death for an NPC that is away. No way to bring one back is offered, so plan for the NPC to return. <span class="tech">(`NPCInfo.state` is marked `[fsIgnore]`. Nothing clears the animator's `Dead` flag or turns the controller back on, so a wake-up style restore is not possible.)</span>
- `Hit` gives `false` when the NPC has no speech component, has no character, or the game has no crime system loaded. It checks the speech component first, so an NPC without one gets no trigger, no line and no crime. <span class="tech">(It gives `false` when the NPC has no `CharacterBarkStarter` on its own object (the game runs the whole NPC part only when the struck object has one), no `Character`, or `Crime.instance` is null.)</span>
- `Hit` can give `false` after it has already done some of its steps. If a game call throws an error part way through, the trigger, the line, the crime or the animation may already have happened.
