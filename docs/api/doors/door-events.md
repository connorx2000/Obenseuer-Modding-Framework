---
title: Door events
parent: Doors
grand_parent: Places and travel
nav_order: 5
---

# Door events

The two events that tell you when a door has finished moving. Use them instead of checking `IsOpen` on every frame.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>Doors</code> <span>the two door events</span></p>

Subscribe to `Opened` and `Closed` on `Doors` to hear when a door has finished moving. See [Using events](../../guides/getting-started/using-events.md) for how events work.

<div class="api-pair" markdown="1">

```csharp
public static class Doors
{
    public static event Action<DoorChangedArgs> Opened;
    public static event Action<DoorChangedArgs> Closed;
}
```

| Event | Gives you | When it fires |
|---|---|---|
| `Opened` | `DoorChangedArgs` | Right after a door finishes opening. |
| `Closed` | `DoorChangedArgs` | Right after a door finishes closing. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>DoorChangedArgs</code> <span>what happened</span></p>

`Opened` and `Closed` give you one of these. Its `Id` is the same as `DoorInfo.Id`, so you can look the door up with [`Doors.Get`](finding-doors.md).

<div class="api-pair" markdown="1">

```csharp
public sealed class DoorChangedArgs
{
    public string Id { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `Id` | `string` | The same id as `DoorInfo.Id`. |

</div>
</div>

## Things to know

- `Opened` and `Closed` fire when the door reaches fully open or fully shut, not when it starts to move. <span class="tech">(They come from the game's private `OnFullyOpen` and `OnFullyClosed`, which only `Door.Update` calls.)</span>
- A door that is stopped halfway fires neither event. <span class="tech">(`Door.Stop` does not reach `OnFullyOpen` or `OnFullyClosed`.)</span>
- Loading a save does not fire the events by itself. A move that a save was in the middle of finishes after the load, and fires then.
- The events do not tell you who opened the door. The game does not record who started a move, and many things can start one: the player, NPCs, elevators, [relays](../../glossary.md#relay) (hidden switch-like scene objects) and delayed key unlocks. <span class="tech">(Callers include `Interact`, NPC `ForceOpen`, elevators, relays and a delayed key unlock.)</span>
- Lock changes are on the [Interaction page](../interaction.md). Use [`Locks.Locked` and `Locks.Unlocked`](../interaction/locks-and-doors.md) for those.
