---
title: Setting up your project
parent: UI
grand_parent: Screen and UI
nav_order: 2
---

# Setting up your project

Before your mod can use the UI kit, your project needs two game file references and the `using` lines below. The [UI and HTML views guide](../../guides/ui.md#set-up-the-project) walks through the project set-up step by step.

This is the full list of what the API gives you, one section for each part.

```csharp
using BepInEx.Unity.Mono.Configuration;
using ObenseuerCore.Ui;
```

## Things to know

- Reference two game files in your project: `Unity.TextMeshPro.dll` and `UnityEngine.UI.dll`. Both are in `Obenseuer_Data\Managed` in the game folder. `ModUi.Title` and `ModUi.Text` give back a `TextMeshProUGUI`, and `ModUi.Button` gives back a `UnityEngine.UI.Button`. Without both references the compiler reports error CS0012. That error means a type is defined in a file your project does not reference.
- Add `using ObenseuerCore.Ui;` to each file that uses the kit.
- [`KeyBinding`](controls.md) takes a `KeyboardShortcut` from [BepInEx](../../glossary.md#bepinex) (the mod loader for Unity games). So a file that calls it also needs `using BepInEx.Unity.Mono.Configuration;` on BepInEx 6. On BepInEx 5, use `using BepInEx.Configuration;` instead.
- Only a BepInEx [plugin](../../glossary.md#plugin) can call `KeyBinding`. A framework mod has no BepInEx reference, so it cannot call it. See [framework mods](../framework-mods/loader-specific-members.md#loader-specific-members).

**Common problems**

- The compiler reports error CS0012: the project lacks one or both references, to `Unity.TextMeshPro.dll` and `UnityEngine.UI.dll`. Add both from `Obenseuer_Data\Managed` in the game folder.
