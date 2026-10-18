---
title: Skills
parent: Custom skills
grand_parent: Player life and skills
nav_order: 2
---

# Skills

The full list of what the `CustomSkills` API gives you: the calls, the event and the rules behind them. Text in brackets at the end of a note is extra detail for experienced modders; skip it on a first read.

A `modId` is your [mod id](../../glossary.md#mod-id) (see [Owner](../../glossary.md#owner) in the glossary). A `skillId` is a short name you pick for the skill, such as `baking`. Together they name one skill, so two mods can use the same `skillId` without clashing.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>CustomSkills</code> <span>register, read and change your skills</span></p>

Call `Register` for each skill. Then use `Get`, `Set` and `Add` to read and change its value.

<div class="api-pair" markdown="1">

```csharp
public static class CustomSkills
{
    public static bool IsAvailable { get; }
    public static bool Register(string modId, string skillId, string displayName, float maxValue);
    public static bool Clear(string modId);
    public static float Get(string modId, string skillId);
    public static int Level(string modId, string skillId);
    public static bool Set(string modId, string skillId, float value);
    public static bool Add(string modId, string skillId, float amount);
    public static bool QueueBanner(string text);
    public static event Action<CustomSkillArgs> LevelChanged;
}
```

| Property | Type | What it means |
|---|---|---|
| `IsAvailable` | `bool` | `false` when the character panel rows and banners could not be set up. |

| Method | Returns | What it does |
|---|---|---|
| `Register(modId, skillId, displayName, maxValue)` | `bool` | Declares a skill. `false` only when the mod already has 64 skills. |
| `Clear(modId)` | `bool` | Unregisters all of this mod's skills for this session. `true` when any were removed. |
| `Get(modId, skillId)` | `float` | The skill's value. `0` when the skill is not registered. |
| `Level(modId, skillId)` | `int` | The value rounded down. `0` when the skill is not registered. |
| `Set(modId, skillId, value)` | `bool` | Sets the value. `true` when it changed. |
| `Add(modId, skillId, amount)` | `bool` | Adds to the value. A negative amount lowers it. `true` when it changed. |
| `QueueBanner(text)` | `bool` | Puts a message in the queue for the skill banner. `true` when it was queued. |

| Event | Gives you | When it fires |
|---|---|---|
| `LevelChanged` | `CustomSkillArgs` | A skill's level went up or down. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>CustomSkillArgs</code> <span>what LevelChanged tells you</span></p>

`LevelChanged` gives you one of these when a skill's whole-number level goes up or down. `ModId` and `SkillId` say which skill it was.

<div class="api-pair" markdown="1">

```csharp
public sealed class CustomSkillArgs
{
    public string ModId { get; }
    public string SkillId { get; }
    public float Value { get; }
    public int Level { get; }
    public int PreviousLevel { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `ModId` | `string` | The mod that owns the skill. |
| `SkillId` | `string` | The skill's short name. |
| `Value` | `float` | The value after the change. |
| `Level` | `int` | The level after the change. |
| `PreviousLevel` | `int` | The level before the change. |

</div>
</div>

## Things to know

- `Register` declares a skill for this session only, so call it every time your mod starts. Call it from `OnLoad`. Calling it again for the same `skillId` replaces the name and the maximum. The skill's value is kept, because it is saved separately.
- The arguments to `Register` have limits:
  - `skillId` is 1 to 64 characters of letters, digits, `.`, `_` or `-`.
  - `displayName` is the name shown to the player. It cannot be empty.
  - `maxValue` is the highest value the skill can reach. It must be a number above 0 and at most 1,000,000. Use `10f` to match the game's own skills, which go up to 10. <span class="tech">(`10f` matches `PlayerStats.maxLevel`. The limit of 1,000,000 exists because the experience text is the value times 1000 and must fit in an `int`.)</span>
- A call with an invalid argument throws an `ArgumentException`. That covers a `modId` or `skillId` that breaks the rules above, an empty `displayName` and a `maxValue` outside the allowed range. It also covers a `value` or `amount` that is not a real number (for example `NaN` or infinity). Fix the call rather than catching the error.
- A skill that is not registered is not an error. That includes one removed by `Clear`. `Get` and `Level` return `0`, and `Set` and `Add` return `false`.
- The level is the value rounded down, so a value of `3.9` is level `3`. The value always stays between 0 and `maxValue`. A value set above `maxValue` or below 0 is held at the edge. The game does the same for its own skills. <span class="tech">(This mirrors `PlayerSkill.Level` and the clamp in `IncreaseSkill`.)</span>
- `Set` and `Add` return `true` when the stored value changed. They return `false` when the skill is not registered, or when the value did not change (for example when it was already at the maximum). They also return `false` while a save is loading or being written. Use a negative amount with `Add` to lower the skill. <span class="tech">(The save check is `SaveData.Busy`.)</span>
- `Clear(modId)` hides your skills for this session, and the player keeps their progress. The saved values stay, and registering the skill again brings them back. The rows disappear the next time the character panel refreshes.
- `LevelChanged` fires when `Set` or `Add` changes the whole-number level. A change that stays inside one level, such as `3.1` to `3.4`, does not fire it. It does not fire when a save loads.

## Limits

- A mod can have at most 64 skills. `Register` returns `false` when your mod already has 64 and the new `skillId` is not one of them. Replacing a skill you already registered does not count against the limit. In every other case it returns `true`. <span class="tech">(The limit is `CustomSkillsMath.MaxPerMod`.)</span>
- `IsAvailable` is `false` when the character panel rows and banners could not be set up. The calls (`Register`, `Get`, `Set`, `Add` and `Clear`) still work then, but no rows appear. `QueueBanner` returns `false`. See [If something fails to set up](details.md#if-something-fails-to-set-up). <span class="tech">(It is `false` when the panel and banner Harmony patches did not install, or when the feature group was turned off.)</span>

**Common problems**

- A call throws an `ArgumentException`: an argument breaks the rules under Things to know. Fix the call rather than catching the error.
- `Get` returns `0`, or `Set` and `Add` return `false`, for a skill you added earlier: the skill is not registered in this session, or `Clear` removed it. Call `Register` from `OnLoad` every time your mod starts; the saved value comes back.
- `Set` or `Add` returns `false` for a registered skill: the value did not change, or a save is loading or being written. In the save case, try again later.
