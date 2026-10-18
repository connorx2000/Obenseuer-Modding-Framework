---
title: Wallet
parent: Framework core classes
grand_parent: Framework basics
nav_order: 7
---

# Wallet

`Wallet` reads and spends the player's OC (Open Sewer Coins, the game's everyday money, see [OC](../glossary.md#oc)). It is the older API, kept so old mods keep working. For new code use `Cash`, which handles OC and RM (Real Money, see [RM](../glossary.md#rm)). `Cash` is described on the [economy](../api/economy/money-cash.md#money-cash) page.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>Wallet</code> <span>read and spend OC</span></p>

Read `Balance` for the player's OC. Use `Has`, `TrySpend` or `Give` to check and change it.

<div class="api-pair" markdown="1">

```csharp
public static class Wallet
{
    public static int Balance { get; }
    public static bool Has(int amount);
    public static bool TrySpend(int amount);
    public static bool Give(int amount);
}
```

| Method | Returns | What it does |
|---|---|---|
| `Has(amount)` | `bool` | `true` when the player has at least this much OC. |
| `TrySpend(amount)` | `bool` | Takes the OC away. `false` when it could not be done, see Things to know. |
| `Give(amount)` | `bool` | Gives the OC to the player. `false` when it could not be done, see Things to know. |

| Property | Type | What it means |
|---|---|---|
| `Balance` | `int` | How much OC the player has. |

</div>
</div>

## Putting it together

This [plugin](../glossary.md#plugin) takes 50 OC when it starts. If the payment worked, it [logs](../guides/getting-started/logging.md) the new balance.

<div class="api-example" markdown="1">

```csharp
using BepInEx;
using BepInEx.Unity.Mono;
using ObenseuerCore;

public class Toll : BaseUnityPlugin
{
    private void Start()
    {
        if (Wallet.TrySpend(50))
        {
            Logger.LogInfo("paid 50 OC, balance " + Wallet.Balance);
        }
    }
}
```

| Step | What it does |
|---|---|
| `Wallet.TrySpend(50)` | Takes 50 OC. `false` if it could not. |
| `Wallet.Balance` | Reads the balance after the payment. |

</div>

## Things to know

- `TrySpend` and `Give` never throw an error. If something is wrong, you get `false`.
- `TrySpend` and `Give` refuse an amount that is not a whole number from 1 to 16,000,000. You also get `false` when the player does not have enough OC to spend, or when the game's money is not ready. <span class="tech">(The cap of 16,000,000 stays below 2^24 (16,777,216), the largest whole number a float holds exactly. The game passes amounts as float, so larger values could be rounded.)</span>
- `TrySpend` and `Give` return `false` while the game is loading or saving, the same as `Cash.Give` and `Cash.TrySpend`. Try again later.
- On the main menu the balance is `0`, and `Has`, `TrySpend` and `Give` return `false`. There is no player money yet. <span class="tech">(This is when `Money.instance` is null.)</span>
- Rarely, a call can return `false` even though the balance did change. If the game fails right after it changes the money, the call returns `false`, but the new balance stays. <span class="tech">(The failure is in the game's `UpdateInteractableMoney` step, which runs after the balance is set.)</span>

**Common problems**

- `TrySpend` or `Give` returns `false`: the amount is not a whole number from 1 to 16,000,000, the player lacks the OC to spend, or the game's money is not ready. It is also `false` on the main menu and while the game is loading or saving. If the game was loading or saving, try again later. Rarely, the call returns `false` even though the balance changed, as the last bullet above explains.
