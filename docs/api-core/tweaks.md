---
title: Tweaks
parent: Framework core classes
grand_parent: Framework basics
nav_order: 11
---

# Tweaks

The framework ships a set of gameplay options, such as slower hunger, bigger stacks or faster crafting. They have no C# API. This page says how each tweak behaves, in case your mod runs next to them.

Players turn a tweak on in the framework's [config file](../guides/getting-started/config.md), `BepInEx\config\obenseuer.moddingframework.cfg`, one section for each tweak. They can also use the in-game config window ([BepInEx](../glossary.md#bepinex) ConfigurationManager, F1).

### Settings

| Setting | Default | What it does | Stays in the save? |
|---|---|---|---|
| `Tweaks.Needs`: `HungerDrainMultiplier`, `ThirstDrainMultiplier`, `HygieneDrainMultiplier`, `AlcoholNeedDrainMultiplier`, `MushroomNeedDrainMultiplier`, `SmokingNeedDrainMultiplier`, `GamblingNeedDrainMultiplier` | `1` | <details>{::nomarkdown}<summary>{:/nomarkdown}Scales how fast that need changes.{::nomarkdown}</summary>{:/nomarkdown}It uses [`Survival.Decay`](survival.md), set or removed live, so the game's methods are not changed. For the four cravings it scales both the rise and the fall.</details> | No. |
| `Tweaks.Stacks`: `StackMultiplier` | `1` | Scales the stack size of items that stack (above 1). The amount of liquid a slot holds scales too. | Stacks that are over the limit stay over it when the value is lowered or set back to `1`, until that slot is filled again. |
| `Tweaks.Spoilage`: `SpoilageMultiplier` | `1` | Scales how fast items spoil. | Spoilage already applied is saved for good. |
| `Tweaks.CraftingSpeed`: `CraftingSpeed` | `1` | <details>{::nomarkdown}<summary>{:/nomarkdown}Scales the craft time of manufacturing and of blueprints. `2` is twice as fast.{::nomarkdown}</summary>{:/nomarkdown}Below `1`, the speed-up for waiting on a blueprint is slower than its target. The craft ends at the scaled length, but the speed-up uses the unscaled time.</details> | <details>{::nomarkdown}<summary>{:/nomarkdown}A craft that is already started keeps the scaled length it began with.{::nomarkdown}</summary>{:/nomarkdown}Manufacturing crafts are checked against the current length, so raising the value mid-craft can finish it at once. Lowering it can make its progress jump, and changing it between sessions does the same.</details> |
| `Tweaks.DishwasherSpeed`: `DishwasherSpeed` | `1` | Scales how long a dishwasher recipe takes, for recipes made from now on. `2` is twice as fast. It stacks with `CraftingSpeed`. | No. |
| `Tweaks.NoMinimumSkill`: `NoMinimumSkill` | `false` | <details>{::nomarkdown}<summary>{:/nomarkdown}Manufacturing only. Removes the minimum skill needed to craft, and the extra craft time for low skill.{::nomarkdown}</summary>{:/nomarkdown}That includes `HardMinimumSkill`. Recipes whose craft roll always fails stay disabled. Skill still affects the chance of failing and the quality. Blueprints are not covered.</details> | No. |
| `Tweaks.Renovations`: `CostMultiplier` | `1` | Scales the [OC](../glossary.md#oc) first payment and the [RM](../glossary.md#rm) final payment of a renovation. The result is rounded to whole units. | No. Orders already placed keep their saved progress. |
| `Tweaks.Renovations`: `MoneyOnly` | `false` | <details>{::nomarkdown}<summary>{:/nomarkdown}Renovations need only money. The item requirement is skipped for the upgrade check and when items are taken.{::nomarkdown}</summary>{:/nomarkdown}A mod that takes renovation items itself does not read this flag. Its prefix (runs before the game's method) on `TenementController.RemoveRequiredItems` still runs, because HarmonyX runs every prefix, and it may still take items. See [`TweakState`](tweakstate.md).</details> | No. |
| `Tweaks.Renovations`: `BuildTimeMultiplier` | `1` | Scales the build time when an order is placed, and the total shown. The total is rounded up to whole hours. | No. Changing it while an order is placed can leave that order's progress wrong. |
| `Tweaks.Renovations`: `InstantBuild` | `false` | Places renovation orders with no build time. Orders started by a script with `TenementUpgradeStart` keep the time the script gave them. | No. |
| `Tweaks.Camera`: `ThirdPersonToggle` | `false` | <details>{::nomarkdown}<summary>{:/nomarkdown}Turns on the `ToggleKey` switch between first and third person.{::nomarkdown}</summary>{:/nomarkdown}Turning it off clears the toggle's lock and goes back to first person, if the game's first-person lock was on.</details> | The framework saves nothing. The game saves `firstperson` and the cleared flags. See [`CameraView`](cameraview.md). |
| `Tweaks.Camera`: `ToggleKey` | `F6` | <details>{::nomarkdown}<summary>{:/nomarkdown}The key that switches the view while `ThirdPersonToggle` is on.{::nomarkdown}</summary>{:/nomarkdown}It is ignored while a game menu, shop, pause, dialogue, the console or game controls are active. `V` is already bound to "Rotation Mode", so it is not the default.</details> | No. |
| `Tweaks.Camera`: `AllowScrollZoom` | `false` | <details>{::nomarkdown}<summary>{:/nomarkdown}Clears the first-person lock so the scroll wheel moves between first and third person.{::nomarkdown}</summary>{:/nomarkdown}The game's values are restored when this and `ThirdPersonToggle` are both off. It is not applied while a camera zone forces first person.</details> | The framework saves nothing. A save made while it is on stores the cleared flags. |
| `Tweaks.Time`: `TimeScale` | `1` | <details>{::nomarkdown}<summary>{:/nomarkdown}Makes the game clock run faster. `2` is twice as fast.{::nomarkdown}</summary>{:/nomarkdown}Need drain, spoilage, craft timers and renovation timers run on game time, so they speed up too.</details> | No. The saved time of day is unchanged. |

### Things to know

- Every tweak is off by default. Multipliers start at `1`, and yes-or-no options start at `false`.
- Changes apply at once from the in-game config window. A tweak changes the game's methods the first time its value becomes different from the default. It keeps those changes for the rest of the run. At the default value the change does nothing and gives the game value back untouched.
- Multipliers are kept between `0.01` and `100`. A value that is not a number becomes `1`. Every time a value is kept inside the limits, it is [logged](../guides/getting-started/logging.md) to `BepInEx\logs\obenseuer.moddingframework.tweaks.log`.
- Each change is logged once as `tweak <Section>.<Key>: <old> -> <new>`. The line is written after the value has stayed still for half a second, so dragging a slider logs one line. When a tweak changes the game's methods it logs `tweak <Section> patched`, or `tweak <Section> patch not applied` if that failed.
- A tweak that fails logs the problem and does nothing else. Its [feature group](../glossary.md#feature-group) is `obenseuer.moddingframework.tweaks`, the name you will see in the log.
- Turning `NoMinimumSkill` off while a station is open takes effect only when the recipe list refreshes, because the craft itself has no skill check.

### Limits

- Some timers ignore `TimeScale`. The NPC route timing and the blueprint wait speed-up still use the unscaled day length.
- If the game's runtime inlines `AzureTimeController.SetNewDayLength`, `TimeScale` does nothing. Check that the clock really speeds up after you set it.
