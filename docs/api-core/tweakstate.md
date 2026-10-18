---
title: TweakState
parent: Framework core classes
grand_parent: Framework basics
nav_order: 12
---

# TweakState

`TweakState` lets your own code check whether one of the framework's [tweaks](tweaks.md) is on. Today it has one member, for the renovations tweak.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>TweakState</code> <span>is a tweak on right now</span></p>

Read `RenovationsMoneyOnly` before your mod takes renovation items itself. Your code then follows the same setting as the game.

<div class="api-pair" markdown="1">

```csharp
public static class TweakState
{
    public static bool RenovationsMoneyOnly { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `RenovationsMoneyOnly` | `bool` | `true` when `Tweaks.Renovations` `MoneyOnly` is on, as the game sees it. |

</div>
</div>

## Putting it together

Your own renovation-item [prefix](../guides/getting-started/patching.md) returns early when `MoneyOnly` is on.

<div class="api-example" markdown="1">

```csharp
private static bool MoneyOnlyOn()
{
    return ObenseuerCore.TweakState.RenovationsMoneyOnly;
}

private static bool MyPrefix()
{
    try
    {
        if (MoneyOnlyOn()) return false;
    }
    catch (Exception)
    {
    }
    return true;
}
```

| Step | What it does |
|---|---|
| `MoneyOnlyOn()` | Reads the flag through a helper, so a missing member is touched only at the call. |
| `if (MoneyOnlyOn()) return false;` | Skips your own item taking while `MoneyOnly` is on. |
| `catch (Exception)` | Keeps the prefix running on an older framework that does not have the flag. |

</div>

## Things to know

- If your mod takes renovation items itself, check this flag first. In your own prefix on `TenementController.RemoveRequiredItems`, return `false` early when it is `true`, before you take anything. The framework cannot do this for you. HarmonyX ([Harmony](../glossary.md#harmony), the library that changes game methods) runs every prefix, so yours runs no matter what the tweak says.
- Read the flag from a small helper, and call the helper from inside try/catch. Your mod may run with an older framework that does not have `RenovationsMoneyOnly`. Then the mod fails when the code that reads it first runs, with a `TypeLoadException` or a `MissingMethodException`. It is thrown at the call into the helper, so a try/catch inside the helper does not catch it. <span class="tech">(Checked with a two-version probe on .NET Framework.)</span>
