---
title: When the framework forgets your settings
parent: Time control
grand_parent: Places and travel
nav_order: 3
---

# When the framework forgets your settings

The framework keeps [`TimeControl`](clock-speed-timers-and-the-minute-event.md) speeds and timers in memory only, and writes nothing to the save. This page lists when they survive and when they are dropped.

## Things to know

- Moving to another level keeps them. A speed or timer set in one level still applies in the next.

## Limits

- Your speed and timers are dropped in these cases:
  - a save is loaded from the main menu or pause menu;
  - the main menu loads;
  - a new game starts.
- A timer set before a save is saved is not in the save. It is gone when that save is loaded from the main menu.
- A call made while a save is loading can be dropped. Set your speed again from [`Events.WhenReady`](../core-helpers/events-whenready.md), which runs after the load. <span class="tech">(`SetTimeScale`, `ClearTimeScale`, `In` and `Cancel` are not blocked while a save loads (they are not gated by `SaveData.Busy`), so the load can clear a call made during it. They do work inside `Events.SceneLoaded`.)</span>

**Common problems**

- Your speed or timer is gone after a load: the framework drops them in the cases listed under Limits. A call made while a save is loading can be dropped too. Set your speed and timers again after the load, from `Events.WhenReady`.
