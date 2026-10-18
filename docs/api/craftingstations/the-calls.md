---
title: The calls
parent: Crafting stations
grand_parent: Items and crafting
nav_order: 2
---

# The calls

The full list of what the API gives you: the three classes, their methods and their properties. A `modId` is your [mod id](../../glossary.md#mod-id), the same one your mod uses for its settings. See [Owner](../../glossary.md#owner) in the glossary.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>CraftingSpeed</code> <span>how fast a kind of station works</span></p>

Call `SetSpeedFactor` with your mod id and a station kind, or `AllStations` for every kind. Call `Clear` to undo it. The kinds and the rules are on the [Speed](speed.md) page.

<div class="api-pair" markdown="1">

```csharp
public static class CraftingSpeed
{
    public const string AllStations = "*";
    public static bool SpeedFactorsAvailable { get; }
    public static bool SetSpeedFactor(string modId, string stationKind, float speed);
    public static bool Clear(string modId);
}
```

| Method | Returns | What it does |
|---|---|---|
| `SetSpeedFactor(modId, stationKind, speed)` | `bool` | Sets how fast one kind of station works for your mod. The speed is from 0.01 to 4. `true` when it was set. |
| `Clear(modId)` | `bool` | Removes all of this mod's speed changes. |

| Property | Type | What it means |
|---|---|---|
| `AllStations` | `string` | Pass this as the station kind to cover every kind of station. It is `"*"`. |
| `SpeedFactorsAvailable` | `bool` | `false` when the speed setup failed. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>CraftingQuality</code> <span>how good what stations make is</span></p>

Call `SetQualityFactor` with your mod id to change how good everything a station makes is. Call `Clear` to undo it. The rules are on the [Quality](quality.md) page.

<div class="api-pair" markdown="1">

```csharp
public static class CraftingQuality
{
    public static bool QualityFactorAvailable { get; }
    public static bool SetQualityFactor(string modId, float factor);
    public static bool Clear(string modId);
}
```

| Method | Returns | What it does |
|---|---|---|
| `SetQualityFactor(modId, factor)` | `bool` | Sets how good everything a station makes is, for your mod. The factor is from 0 to 4. `true` when it was set. |
| `Clear(modId)` | `bool` | Removes this mod's quality change. |

| Property | Type | What it means |
|---|---|---|
| `QualityFactorAvailable` | `bool` | `false` when the quality setup failed. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>CraftingStations</code> <span>start, stop, pause and resume a station</span></p>

Call `Start`, `Stop`, `Pause` and `Resume` with your mod id and the station's id to control a craft. The station ids come from [`Crafting.All()`](../recipes/crafting-stations.md). The rules are on the [Start and stop](start-and-stop.md) and [Pause and resume](pause-and-resume.md) pages.

<div class="api-pair" markdown="1">

```csharp
public static class CraftingStations
{
    public static bool Start(string modId, string stationId);
    public static bool Stop(string modId, string stationId);
    public static bool Pause(string modId, string stationId);
    public static bool Resume(string modId, string stationId);
}
```

| Method | Returns | What it does |
|---|---|---|
| `Start(modId, stationId)` | `bool` | Starts a craft. `true` when the station started. |
| `Stop(modId, stationId)` | `bool` | Ends a liquid batch early. `true` when it stopped. |
| `Pause(modId, stationId)` | `bool` | Freezes a running batch. `true` when it froze. |
| `Resume(modId, stationId)` | `bool` | Unfreezes a batch. `true` when it carried on. |

</div>
</div>
