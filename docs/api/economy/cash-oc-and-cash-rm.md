---
title: Cash.Oc and Cash.Rm
parent: Economy
grand_parent: Money and shops
nav_order: 3
---

# Cash.Oc and Cash.Rm

These two properties read the player's balance directly. [OC](../../glossary.md#oc) (Open Sewer Coins, the everyday currency) comes back as a whole number, [RM](../../glossary.md#rm) (Real Money) as a decimal number.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>Cash</code> <span>the balance, read directly</span></p>

Read `Cash.Oc` or `Cash.Rm` whenever you need the balance. They are plain reads, not events, so read them again after a change.

<div class="api-pair" markdown="1">

```csharp
public static class Cash
{
    public static int Oc { get; }
    public static float Rm { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `Oc` | `int` | The OC balance as a whole number. `0` while money is not loaded. |
| `Rm` | `float` | The RM balance. The same value as `Cash.Balance(MoneyCurrency.RM)`. |

</div>
</div>

### Before and after

Before:

```csharp
if (Cash.Balance(MoneyCurrency.OC) >= 50f) { }
```

After:

```csharp
if (Cash.Oc >= 50) { }
```

### Things to know

- Both properties are read-only, and they work at any time. Like `Balance`, they are not blocked while a save is loading.
- Use `Oc` for big OC balances. `Oc` reads the game's whole number directly. [`Balance(MoneyCurrency.OC)`](money-cash.md) gives the same number as a decimal, which is only exact up to 16,777,216.
- `Rm` gives the RM balance as the game stores it, with no rounding.
- `Cash.Oc` has the same value as [`Wallet.Balance`](../../api-core/wallet.md).
