---
title: Common tasks
parent: Lights
grand_parent: Objects and devices
nav_order: 1
---

# Common tasks

Short snippets for the usual jobs with [`Lights`](simple-lights.md). They switch simple lights on, step multi-state lights through their settings, and turn multi-state lights all off.

Switch on every simple light in the [scene](../../glossary.md#scene). `SetOn` returns `true` when the light now matches what you asked for.

```csharp
foreach (LightInfo light in Lights.All())
    if (!light.On) Lights.SetOn(light.Id, true);
```

Move every [multi-state light](multi-state-lights.md) to its next setting, wrapping back to the first. A light that is switched off cannot be changed, so `SetState` returns `false` for it.

```csharp
foreach (MultiStateLightInfo light in Lights.AllMultiState())
{
    int next = light.State + 1 >= light.StateCount ? 0 : light.State + 1;
    Lights.SetState(light.Id, next);
}
```

Turn a multi-state light all the way off (no setting lit), while the fixture itself stays switched on.

```csharp
foreach (MultiStateLightInfo light in Lights.AllMultiState())
    Lights.SetState(light.Id, -1);
```
