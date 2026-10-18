---
title: Changing respawn settings
parent: Spawners
grand_parent: Objects and devices
nav_order: 3
---

# Changing respawn settings

These two calls change whether a spawner respawns and how long it waits. Use them to switch respawning on or off, change its chance, or set your own wait.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>Spawners</code> <span>respawn on or off, and the wait</span></p>

`SetRespawn` turns respawning on or off and sets its chance. `SetRespawnTiming` sets how long a spawner waits.

<div class="api-pair" markdown="1">

```csharp
public static class Spawners
{
    public static bool SetRespawn(string id, bool enabled, float chance);
    public static bool SetRespawnTiming(string id, RespawnTime time, bool useCustomSpawnTime, int minSeconds, int maxSeconds);
}
```

| Method | Returns | What it does |
|---|---|---|
| `SetRespawn(id, enabled, chance)` | `bool` | Turns respawning on or off and sets the chance. `true` when written. |
| `SetRespawnTiming(id, time, useCustomSpawnTime, minSeconds, maxSeconds)` | `bool` | Sets the wait. `true` when written. |

</div>
</div>

## Things to know

- `SetRespawn` sets the chance of a spawn. The chance is limited to 0 through 1. <span class="tech">(it sets `spawnSettings.respawnEnabled` and `spawnSettings.spawnChance`.)</span>
- `SetRespawnTiming` sets how long a spawner waits. The two seconds values are game seconds. They only count when `useCustomSpawnTime` is `true`. The presets for `time` are listed under `RespawnTime` on [Reading spawners](reading-spawners.md). <span class="tech">(it sets `spawnSettings.spawnTime`, `useCustomSpawnTime`, `minRespawnTime` and `maxRespawnTime`.)</span>
- A new wait only applies from the next spawn, not to the countdown already running. The game picks the next countdown once and keeps it until that spawn happens. `SecondsToSpawn` is that countdown, and your mod can't change it. <span class="tech">(the countdown is drawn in `SetSpawnTime`, which returns early while a countdown is running. `SecondsToSpawn` is `timetospawn`, and the framework does not write it.)</span>
- Changes last for the scene session. The game does not [save](../../guides/getting-started/save-data.md) these settings, so loading the scene again brings back the values the map was built with. The game does save the countdown. <span class="tech">(`respawnEnabled`, `spawnChance`, `spawnTime`, `useCustomSpawnTime`, `minRespawnTime` and `maxRespawnTime` are marked `fsIgnore` in `SpawnSettings`, so the save does not store them. `timetospawn` is not `fsIgnore`, so the save keeps the timer.)</span>

## Limits

- `SetRespawn` returns `false` for an unknown id, or while a save is loading or being written.
- Turning respawn on may not start the timer. The game only starts the respawn timer when a spawner starts with respawn already on. A spawner that started with respawn off gets the new setting, but its timer does not run until its scene loads again.
- `SetRespawnTiming` returns `false` in these cases:
  - the id is unknown
  - `time` is not in `RespawnTime`
  - `minSeconds` is below 0
  - `maxSeconds` is below `minSeconds`
  - a save is loading or being written

**Common problems**

- A new wait has no effect yet: it only applies from the next spawn, not to the countdown already running.
- Your settings are gone after the scene loads again: the game does not save them, so the scene brings back the values the map was built with.
- You turned respawn on, but the spawner does not respawn: it started with respawn off, so its timer does not run until its scene loads again.
