---
title: Notes
parent: Items
grand_parent: Items and crafting
nav_order: 4
---

# Notes

These notes explain what item quality means here, and how an [`ItemDef`](itemdef.md) behaves once you have it. Read them before you keep an `ItemDef` for later use.

## Things to know

- Quality and condition belong to each single item the player holds, not to the catalogue. `Quality` here is only the starting value in the catalogue.
- An `ItemDef` is a snapshot made when you asked, and reading a property allocates nothing. `Price` and `MaxStack` show the difficulty at the time of the call. Do not keep an `ItemDef` and treat it as a live value. <span class="tech">(`EquipSlot`, `EquipType`, `AmmoItemIds`, `AmmoCategories`, `FuelBurnTime`, `Electronic` and `ElectronicType` are read once, when the `ItemDef` is built.)</span>
