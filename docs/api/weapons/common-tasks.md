---
title: Common tasks
parent: Weapons
grand_parent: Player life and skills
nav_order: 1
---

# Common tasks

Short snippets for the usual jobs with `Weapons` and `Osrats`: hit factors, ammo and reloading, the flashlight and the osrat.

Make mining twice as effective. `Mod.Id` is your mod's id, passed as the [owner](../../glossary.md#owner) of the factor. The factor is kept only until the next scene load, so set it again after each load.

```csharp
Weapons.SetHitFactor(Mod.Id, HitTarget.Mining, 2f);
```

Give a small hit a chance to land one more point, instead of always rounding.

```csharp
Weapons.SetHitFactor(Mod.Id, HitTarget.Chopping, 1.3f, true);
```

Remove every hit factor your mod has set. The rules for factors are on [Hit power](hit-power.md).

```csharp
Weapons.ClearHitFactors(Mod.Id);
```

Count the Batteries D (item 10580) the player carries, and [reload](reloading.md) the weapon in the right hand.

```csharp
Log.Info("Batteries D: " + Weapons.Ammo(10580));
if (Weapons.Reload()) Log.Info("Reloading.");
```

Turn the [flashlight](flashlight.md) on, if the player holds one.

```csharp
if (Weapons.FlashlightOn() == false) Weapons.SetFlashlightOn(true);
Log.Info("Charge: " + Weapons.FlashlightCharge());
```

Read the [osrat](osrat-signal-detector.md) (the hand-held signal detector).

```csharp
Log.Info("Osrat signal " + Osrats.Strength + ", beeping: " + Osrats.IsBeeping);
```
