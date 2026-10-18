---
title: Loading
parent: Framework mods
grand_parent: Framework basics
nav_order: 4
---

# Loading

These notes cover when your mod starts, and what happens when it fails or stops. They explain when the `OnLoad` method from [Your first mod](../../guides/getting-started/first-mod.md) runs.

## Things to know

- The framework loads every framework mod in one go, after all [BepInEx](../../glossary.md#bepinex) (the game's mod loader) plugins have started. Do your starting work in `OnLoad`. <span class="tech">(BepInEx plugins have all run `Awake` by then.)</span>
- `OnLoad` runs once. [`Loader.Has(yourOwnId)`](dependencies.md) is `false` while your mod's own `Awake` and `OnLoad` are running. It becomes `true` when `OnLoad` finishes.
- Unity runs your class's own `Awake` before `OnLoad`. A framework mod stays loaded until the game closes. <span class="tech">(Mono cannot unload an assembly.)</span>

## Limits

- If `OnLoad` throws an error, your mod is switched off:
  - The error goes to your mod's log and to `ObenseuerCore.log`.
  - Your mod's component is destroyed straight away, so its `Update` cannot run on a half-started mod.
  - Every mod that depends on yours is skipped, but other mods and the framework keep running.
- Things your mod did before the error are not undone, such as event subscriptions, saved-data registrations and game patches. Register them last in `OnLoad`.
- An error in your class's `Awake` is logged by Unity. The framework does not stop it from spreading.

**Common problems**

- Your mod is switched off, and mods that depend on it are skipped: `OnLoad` threw an error. Read the error in your mod's [log](../../guides/getting-started/logging.md) or in `ObenseuerCore.log`.
- Event subscriptions or game patches stay active after `OnLoad` failed: they are not undone. Register them last in `OnLoad`.
