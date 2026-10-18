---
title: Ordering a renovation
parent: Renovations
grand_parent: Places and travel
nav_order: 4
---

# Ordering a renovation

`Start` orders a renovation for the player through the game's own order path. Use it when your mod places an order while the contractor menu is open.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>Renovations</code> <span>order a renovation</span></p>

Call `Start` with a renovation's `Id` to order it for the player. It works only while the contractor menu is open and a contractor is chosen.

<div class="api-pair" markdown="1">

```csharp
public static class Renovations
{
    public static bool Start(string id);
}
```

| Method | Returns | What it does |
|---|---|---|
| `Start(id)` | `bool` | Orders a renovation the way the game does. `true` when the game took the order. |

</div>
</div>

## Things to know

- `Start` uses the game's own order path, so the game takes the first payment and the items. It gives `true` only when the game accepted the order. If `Start` gives `false` before it calls the game, nothing has been charged. <span class="tech">(It calls `TenementApartment.StartUpdate` or `TenementGeneral.StartUpdate`. The game takes the OC first payment and the required items inside that call (`TenementUpgrade.TakeRequiredItemsAndMoney`).)</span>
- After an order, `Start` tidies the screen the way the upgrade panel does:
  - It removes the ordered renovation, and the lower ones in its line, from the player's marked list.
  - It closes the upgrade panel if it is open.
  - It refreshes the floor buttons.

  This stops a panel that was open before the call from ordering the same renovation a second time. <span class="tech">(It calls `TenementController.RemoveSupersededMarkedUpgrades`, `CloseUpgradePanel`, and `BlueprintManager.UpdateAll`.)</span>

## Limits

- `Start` gives `false` unless all of these are true: <span class="tech">(The checks are `TenementController.contractorOpen` with a `currentContractor`, `TenementContractor.CanUpgrade`, `Money.CheckMoney` for the OC price, and `TenementUpgrade.CanUpgrade` with items, unless `TenementController.ignoreUpgradePricesAndItems` is set.)</span>
  - The contractor menu is open and a contractor is chosen.
  - The renovation is `Available`.
  - The contractor offers that upgrade for that size of apartment.
  - The player can pay the first payment in [OC](../../glossary.md#oc) (Open Sewer Coins) and has the required items. The one exception is when the game's setting that ignores upgrade prices and items is on, which skips this check.
- `Start` gives `false` for an id that is `null` or empty, and [logs](../../guides/getting-started/logging.md) once for it. It also gives `false` while a save is loading or being written.
- `Start` does not do the developer shortcut that finishes a renovation at once.

**Common problems**

- `Start` gives `false` and nothing is ordered: one of the conditions in the first two bullets under Limits is not met. Check that the contractor menu is open with a contractor chosen, and that the renovation's `State` is `Available`.
