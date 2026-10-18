---
title: What you can't do (yet)
parent: Crafting stations
grand_parent: Items and crafting
nav_order: 9
---

# What you can't do (yet)

The jobs this API does not cover, and why. Check here before you plan a mod around starting, stopping or pausing a station.

- You can't stop a `manufacturing` or a `Growing` station. A craft takes its ingredients when it starts, and a plant takes its seeds. The game has no cancel that hands them back. The only way the game removes them is by paying out the result.
- You can't start a station that needs the player to stand at it. Its craft speeds up the game clock behind the panel's waiting screen. Its timer freezes when the player leaves the panel.
- You can't pause a growing station from this page. Use `GrowingStations.SetPaused`, on the [Growing stations](../growingstations.md) page.
