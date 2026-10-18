---
title: Reloading
parent: Weapons
grand_parent: Player life and skills
nav_order: 3
---

# Reloading

`Weapons.Reload()` reloads the item in the player's right hand, the same way the reload key does. This page covers when the game takes the reload and when it refuses.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>Weapons</code> <span>the reload call</span></p>
`Reload` reloads the weapon in the player's hand.

<div class="api-pair" markdown="1">

```csharp
public static class Weapons
{
    public static bool Reload();
}
```

| Method | Returns | What it does |
|---|---|---|
| `Reload()` | `bool` | Reloads the item in the right hand. `true` when the game took the reload. |

</div>
</div>

## Things to know

- `Reload()` does what the player does when they press the reload key for the right-hand item. It returns `true` only when the game really took the reload. You can see that because an ammo item leaves the inventory.
- The ammo refills after the game's cooldown (1.6 seconds for a Detonator). `Reload` does not pull a trigger for you. <span class="tech">(It calls `PlayerHandItems.ReloadUse`, the call the Primary Reload key makes. `Reload` works on the right hand only.)</span>

## Limits

- `Reload()` returns `false` in these cases:
  - a menu is open
  - using items is switched off
  - nothing is held
  - the held item is `None` or has no ammo
  - a save is loading or being written.
- It also returns `false` when the game refuses. For example, the game refuses during its cooldown, when the ammo is already full, or when the player carries no ammo item. The game refuses without using anything up.
