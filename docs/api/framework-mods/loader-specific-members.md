---
title: Loader-specific members
parent: Framework mods
grand_parent: Framework basics
nav_order: 8
---

# Loader-specific members

Some parts of the framework use types from [BepInEx](../../glossary.md#bepinex) (the game's mod loader). A framework mod has no reference to BepInEx, so it can't call these, and the compiler stops with error CS0012. Only BepInEx plugins can call them.

The table lists each such member. The right column says what a framework mod uses instead. If you need one of the members marked "nothing", write a [BepInEx plugin](../../guides/getting-started/packaging.md#bepinex-plugins-advanced) instead.

| Member | BepInEx type it needs | A framework mod uses |
|---|---|---|
| `CoreMod.For(BaseUnityPlugin)` | `BaseUnityPlugin`, the base class of a BepInEx plugin | `FrameworkMod.Mod` |
| `Loader.Find(string)` | `PluginInfo`, BepInEx's record of a plugin | `Loader.Has(string)` |
| `Loader.All()` | `PluginInfo` | nothing |
| `ModUi.KeyBinding(...)` | `KeyboardShortcut`, BepInEx's key and modifier combination | nothing |
| `ModKeyBinding.Set(KeyboardShortcut)` | `KeyboardShortcut` | nothing |
| `Core` base type | `BaseUnityPlugin` | its static members work |
| `WebUiPlugin` | `BaseUnityPlugin` | nothing, mods never use it |

**Common problems**

- The compiler stops with error CS0012: your framework mod calls one of the members above. Use what the right column names instead. Where it says nothing, a framework mod has no replacement.
