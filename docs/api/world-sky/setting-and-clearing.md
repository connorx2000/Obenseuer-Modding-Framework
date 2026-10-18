---
title: Setting and clearing
parent: World sky
grand_parent: Places and travel
nav_order: 2
---

# Setting and clearing

`WorldSky` is the static class that sets and clears your sky values and reads the sky as it looks now. This is the full list of what the API gives you. `modId` is your mod's id (see [Mod id](../../glossary.md#mod-id)).

<div class="api-type" markdown="1">
<p class="api-type-head"><code>WorldSky</code> <span>set and clear your sky values</span></p>
Use `Set` to change the sky to your values, and `Clear` to take your values back off. Use `Current` to read the sky as it looks now.

<div class="api-pair" markdown="1">

```csharp
public static class WorldSky
{
    public static bool IsAvailable { get; }
    public static SkyState Current { get; }
    public static bool Set(string modId, SkyOverride values);
    public static bool Clear(string modId);
}
```

| Method | Returns | What it does |
|---|---|---|
| `Set(modId, values)` | `bool` | Applies your values. `true` when they were applied. |
| `Clear(modId)` | `bool` | Removes your values. `true` when you had some. |

| Property | Type | What it means |
|---|---|---|
| `IsAvailable` | `bool` | `false` when the framework could not set up the sky changes. |
| `Current` | `SkyState` | What the sky looks like now, with any overrides included. `null` when it cannot be read. |

</div>
</div>

## Things to know

- `Set` applies your values on top of what the game does. If you call `Set` again, your new values replace your earlier ones.
- The values are copied. Changing your [`SkyOverride`](the-values.md) afterwards does nothing until you call `Set` again.
- `CloudDensity` is held between `0` and `1`. A larger or smaller number is clamped to that range.
- `Clear` removes only your values, and returns `true` when you had some.
- `Current` is `null` when the framework cannot read the sky. It never throws.

## Limits

- `Set` returns `false` when `IsAvailable` is `false`.
- `Set` throws an `ArgumentException` for a bad call: an invalid mod id, a `null` `values`, or a number that is not a real number (`NaN` or infinity). It also throws for a density, intensity or exposure below `0`.
- `Clear` throws an `ArgumentException` for an invalid mod id.

**Common problems**

- Your values are gone after a save loads or the level changes: they are not kept. Set them again from `Events.SceneLoaded` or `Events.Loaded` (see [How several mods share the sky](how-several-mods-share-the-sky.md#limits)).
