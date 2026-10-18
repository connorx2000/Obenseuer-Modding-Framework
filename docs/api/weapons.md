---
title: Weapons
parent: Player life and skills
grand_parent: API reference
has_children: true
nav_order: 10
---

# Weapons

[`Weapons`](weapons/held-weapons.md) covers what the player holds in their hands: melee weapons, the Detonator (the hand-held trigger for explosives), the flashlight and the osrat. Use it to read or react to these items, or to change how hard melee hits land.

Status: experimental.

## What it is

This API is about the things the player holds in their hands. It gives your mod these jobs:

- See which weapon is held in each hand, and how much ammo it has.
- Reload the held weapon.
- Get told when a Detonator goes off, when a melee swing starts and when a melee hit lands.
- Make melee hits stronger or weaker for each kind of target: rocks, trees, breakable things and animal props.
- Read and switch the flashlight.
- Read the osrat, a hand-held signal detector.

The game has no guns the player can hold, so there is no gun in this API. See [What you can't do (yet)](weapons/what-you-can-t-do-yet.md).

## When to use it

- Log or react when the player swings a melee weapon or hits something with it.
- Make mining, chopping or breaking things easier or harder.
- Check what the player holds, and how much ammo it has left.
- Turn the flashlight on or off from your own code.
- Read the osrat's signal strength.

## Quick example

This mod writes a [log](../guides/getting-started/logging.md) line for every melee swing. It also logs every hit, with the damage before and after any change. Once a save is loaded, it logs what the player holds in the right hand.

```csharp
using ObenseuerCore;

[ModInfo("example.weaponwatch", "Weapon Watch", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class WeaponWatch : FrameworkMod
{
    protected override void OnLoad()
    {
        Weapons.MeleeSwung += weapon => Log.Info("Swing with item " + weapon.ItemId);
        Weapons.Hit += hit => Log.Info(hit.Target + " hit for " + hit.BaseDamage + ", final " + hit.FinalDamage);
        Events.WhenReady(() =>
        {
            WeaponInfo held = Weapons.Equipped();
            Log.Info(held == null ? "Nothing in the right hand." : "Right hand: item " + held.ItemId + ", " + held.Kind);
        });
    }
}
```

`MeleeSwung` and `Hit` are events: the framework calls your code each time they happen. [`Events.WhenReady`](core-helpers/events-whenready.md) waits until a save is loaded before it runs your code. See [Using events](../guides/getting-started/using-events.md) for more on events and `WhenReady`.
