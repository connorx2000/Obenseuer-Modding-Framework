---
title: Treatments and prices
parent: Medical
grand_parent: Player life and skills
nav_order: 2
---

# Treatments and prices

This is the full list of what the API gives you. Text in brackets at the end of a note is extra detail for experienced modders; you can skip it on a first read. [OC](../../glossary.md#oc) is Open Sewer Coins and [RM](../../glossary.md#rm) is Real Money.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>Medical</code> <span>list, check, buy and hear about treatments</span></p>

Call `Treatments()` to see what the doctor sells and `CanTreat(kind)` to check one. `Treat(kind)` buys it, and `Treated` tells you about each purchase. The doctors it reads are the ones in the [scene](../../glossary.md#scene) you are in.

<div class="api-pair" markdown="1">

```csharp
public static class Medical
{
    public static bool IsAvailable { get; }
    public static List<MedicalTreatmentInfo> Treatments();
    public static bool CanTreat(MedicalTreatment kind);
    public static bool Treat(MedicalTreatment kind);
    public static event Action<MedicalTreatedArgs> Treated;
}
```

| Method | Returns | What it does |
|---|---|---|
| `Treatments()` | `List<MedicalTreatmentInfo>` | The treatments that the doctors in the scene sell. |
| `CanTreat(kind)` | `bool` | `true` when the doctor's screen is open, the doctor offers the treatment, and the player can pay. |
| `Treat(kind)` | `bool` | Buys the treatment for the player. `true` when it was bought. |

| Property | Type | What it means |
|---|---|---|
| `IsAvailable` | `bool` | `false` when the `Treated` event could not be set up. |

| Event | Gives you | When it fires |
|---|---|---|
| `Treated` | `MedicalTreatedArgs` | A treatment was bought. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>MedicalTreatmentInfo</code> <span>one treatment and its price</span></p>

`Treatments()` gives you one of these for each treatment the doctor sells. `Price` says what it costs, and `Available` says whether the doctor offers it now. <span class="tech">(It is built from the game's `MedicalTreatments` slots.)</span>

<div class="api-pair" markdown="1">

```csharp
public sealed class MedicalTreatmentInfo
{
    public MedicalTreatment Kind { get; }
    public int Price { get; }
    public MoneyCurrency Currency { get; }
    public bool Available { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `Kind` | `MedicalTreatment` | Which treatment it is. |
| `Price` | `int` | What it costs, in `Currency`. |
| `Currency` | `MoneyCurrency` | OC (Open Sewer Coins) or RM (Real Money). |
| `Available` | `bool` | `true` when the doctor offers it right now. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>MedicalTreatment</code> <span>the treatments the doctor sells</span></p>

`Treat` and `CanTreat` take one of these, and `MedicalTreatmentInfo.Kind` gives you one. The table below says what each value does for the player.

<div class="api-pair" markdown="1">

```csharp
public enum MedicalTreatment
{
    PrescriptionShroomSubstitute,
    Fungicide,
    PrescriptionFungicide,
    GrowthsRemovalBasic,
    DangerousExperimentalTreatment,
    Antidepressants,
    PrescriptionAntidepressants,
    RubberDuck,
    PhysicalPainkillers,
    PhysicalPrescriptionPainkillers,
    AdhesiveBandages,
    Bandages,
    HealWounds,
    RadiationPainkillers,
    RadiationPrescriptionPainkillers,
    IodinePills
}
```

| Value | Meaning |
|---|---|
| `PrescriptionShroomSubstitute` | Gives the item "Mushroom Substitute Prescription" (id 15810). |
| `Fungicide` | Gives the item "Fungicide" (id 15020). |
| `PrescriptionFungicide` | Gives the item "Fungicide Prescription" (id 15040). |
| `GrowthsRemovalBasic` | Hurts the player and lowers the SMV progression (the fungal disease stat) and its rate. |
| `DangerousExperimentalTreatment` | Hurts the player more, and lowers the SMV progression and its rate by more. |
| `Antidepressants` | Gives the item "Antidepressants" (id 140020). |
| `PrescriptionAntidepressants` | Gives the item "Antidepressants Prescription" (id 12852). |
| `RubberDuck` | Gives the item "Rubber Duck" (id 11). |
| `PhysicalPainkillers` | Gives the item "Painkillers" (id 140010). |
| `PhysicalPrescriptionPainkillers` | Gives the item "Painkillers Prescription" (id 12851). |
| `AdhesiveBandages` | Gives the item "Adhesive Bandages" (id 12670). |
| `Bandages` | Gives the item "Bandage" (id 10940). |
| `HealWounds` | Sets the player's health to 100 and bleeding to 0. |
| `RadiationPainkillers` | Gives the item "Painkillers" (id 140010). |
| `RadiationPrescriptionPainkillers` | Gives the item "Painkillers Prescription" (id 12851). |
| `IodinePills` | Gives the item "Iodine Pills" (id 15360). |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>MedicalTreatedArgs</code> <span>what <code>Treated</code> tells you</span></p>

`Treated` gives you one of these when a treatment is bought. `Kind` says which one. `Price` and `Currency` say what it cost.

<div class="api-pair" markdown="1">

```csharp
public sealed class MedicalTreatedArgs
{
    public MedicalTreatment Kind { get; }
    public int Price { get; }
    public MoneyCurrency Currency { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `Kind` | `MedicalTreatment` | Which treatment was bought. |
| `Price` | `int` | What it cost. |
| `Currency` | `MoneyCurrency` | The currency it was paid in. |

</div>
</div>

## Things to know

- The list of treatments is fixed. New ones are only ever added at the end, so the numbers behind the names will not change between versions.
- `Treatments()` lists every treatment that a doctor in the [scene](../../glossary.md#scene) sells, whether or not the doctor offers it right now. `Available` tells you which ones the doctor's screen shows at the moment. The list is empty when no doctor is loaded. With more than one doctor in the scene, you get each treatment once, from the first doctor that sells it. <span class="tech">(`Available` is the result of the game's `GetAvailableTreatments()` for that treatment's selection (the object holding one group of treatments), which is the list the doctor's menu shows. The framework finds the doctors with `SceneIds.Active<Doctor>()`.)</span>
- `CanTreat` is `true` only when the doctor's screen is open and the player can buy that treatment. It needs three things: the treatment belongs to the doctor whose screen is open, the doctor offers it, and the player has at least `Price` in `Currency`. It is `false` on the main menu, in any other menu, and with no menu open. <span class="tech">(it requires `GameUIController.IsMenuVisible`, the Doctor panel as the current panel, a `DoctorController.currentDoctor`, an active `DoctorPanel`, and the `BlackCanvas`, `Inventory` and `PlayerStats` instances.)</span>
- `Treat` buys the treatment the same way the doctor's buy button does. The game charges the price, plays the sound, and gives the item or applies the effect. Your mod never takes the money itself. `Treat` returns `true` when the treatment was bought. <span class="tech">(it calls `DoctorPanelSelection.BuyTreatment(Treatment)`, which charges through `Money.Buy`.)</span>
- `Treated` fires every time a treatment is bought, also when the player uses the doctor's own button. It fires after the purchase went through, and not when the purchase failed. <span class="tech">(it is raised after the game's `BuyTreatment` returns `true`, from a postfix on the `BuyTreatment` of the five treatment classes, not from `Treat`.)</span>
- Reading the list can update the price of healing wounds. The game works out that price when it builds its list of offered treatments. `Treatments()` and `CanTreat` build that list too, as the doctor's screen does. <span class="tech">(`GetAvailableTreatments` on the physical health selection sets the `HealWounds` price.)</span>
- Conditions and survival values are on other pages. The conditions the player has are on the [Conditions](../conditions.md) page. Addictions, mood and radiation values are read with [`Survival`](../../api-core/survival.md). <span class="tech">(`Conditions.All`, and `Survival.Get` for mood, radiation and fungal progression.)</span>

## Limits

- `Treat` checks `CanTreat` first. If the doctor's screen is closed, the treatment is not offered, or the player can't pay, it returns `false`. It then takes no money and shows no "not enough money" popup. It also returns `false` while a save is loading or being written.
- `Treat` needs the doctor's screen to be open, because the game was written for it. Healing wounds fades the screen, closes the menu, and can set off the examination bed. With no menu open, that would run in the middle of play. On the main menu it would fail after the money was taken. Opening and closing the doctor's screen is not part of this API, because it is user interface. <span class="tech">(the `HealWounds` branch of `BuyTreatment` calls `BlackCanvas.StartFadeInAndOut` with `closeMenu: true` and can fire the examination bed's relays. `BlackCanvas.instance` is null on the main menu. `DoctorController.ShowMenu(Doctor)` opens the menu through `GameUIController`.)</span>
- Heal wounds closes the menu and may set off the examination bed. When the bed is loaded and the player is hurt, the screen fades while the bed does its own actions. The game's own button does the same. What those actions do has not been checked. <span class="tech">(the relays are the bed's `teleportRelay` and `endRelay`.)</span>
- The money is taken first, so a failure afterwards can cost the player money. If the sound or the item step fails after the payment, the money is gone. `Treat` returns `false` and `Treated` does not fire.
- A treatment that gives an item is still paid for when the inventory is full. The game then gives nothing, but still counts the purchase as done. The doctor's own button does the same. `Treat` does not check for free space in the inventory. <span class="tech">(the game calls `Inventory.AddItemDialogue`, which gives nothing when the inventory is full, and `BuyTreatment` still returns `true`.)</span>
- `IsAvailable` tells you whether `Treated` works. It is `false` when the framework could not set up the event. `Treatments`, `CanTreat` and `Treat` still work.
- Buying through the doctor's info panel is not part of this API. That button charges the price of the current selection and records the purchase in the menu state. So it depends on what the menu has selected. <span class="tech">(`DoctorPanel.OnBuyButtonPressed` charges `currentSelection.price`.)</span>

**Common problems**

- `Treat` returns `false`: the doctor's screen is closed, the treatment is not offered, the player can't pay, or a save is loading or being written. It also returns `false` when the sound or the item step fails after the payment, and then the money is gone. Any other game failure during the call also gives `false`. `CanTreat` checks the screen, the offer and the price before you buy.
- The player paid but got no item: the inventory was full. The game still counts the purchase as done, and `Treat` does not check for free space.
- `Treated` does not fire: the purchase failed, or the framework could not set up the event. In the second case `IsAvailable` is `false`, and the error is in `BepInEx\logs\obenseuer.moddingframework.medical.log` (see [Details](details.md#if-something-fails-to-set-up)).
