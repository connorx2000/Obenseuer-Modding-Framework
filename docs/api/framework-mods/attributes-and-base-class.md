---
title: Attributes and base class
parent: Framework mods
grand_parent: Framework basics
nav_order: 2
---

# Attributes and base class

An attribute is a label you put above your class in square brackets. `ModInfo` gives the mod its id, name and version, and `ModDependency` names a mod yours needs. Your class inherits from `FrameworkMod`, which gives you a log, settings and the other tools a mod needs.

The `MyFirstMod` class in [Your first mod](../../guides/getting-started/first-mod.md) uses all three. This page lists what each one holds.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>ModInfoAttribute</code> <span>goes on your class, once</span></p>

Put it on your class, once. The framework reads it to find your mod's id and name. `FrameworkMod.Info` gives it back to you.

<div class="api-pair" markdown="1">

```csharp
public sealed class ModInfoAttribute : Attribute
{
    public ModInfoAttribute(string id, string name, string version);
    public string Id { get; }
    public string Name { get; }
    public string Version { get; }
}
```

| Method | Returns | What it does |
|---|---|---|
| `ModInfoAttribute(id, name, version)` | `ModInfoAttribute` | Sets the id, name and version of your mod. |

| Property | Type | What it means |
|---|---|---|
| `Id` | `string` | The [mod id](../../glossary.md#mod-id), such as `example.myfirstmod`. |
| `Name` | `string` | The name shown in the [Mods menu](mods-menu.md). |
| `Version` | `string` | The mod's version, such as `0.1.0`. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>ModDependencyAttribute</code> <span>goes on your class, as many times as you need</span></p>

Put one on your class for each mod it needs. The framework reads them to plan the load order. It skips your mod when one is missing or too old. [Dependencies](dependencies.md) has the rules.

<div class="api-pair" markdown="1">

```csharp
public sealed class ModDependencyAttribute : Attribute
{
    public ModDependencyAttribute(string id);
    public ModDependencyAttribute(string id, string minVersion);
    public string Id { get; }
    public string MinVersion { get; }
}
```

| Method | Returns | What it does |
|---|---|---|
| `ModDependencyAttribute(id)` | `ModDependencyAttribute` | The mod must be there. |
| `ModDependencyAttribute(id, minVersion)` | `ModDependencyAttribute` | The mod must be there and at least this version. |

| Property | Type | What it means |
|---|---|---|
| `Id` | `string` | The id of the mod you need. |
| `MinVersion` | `string` | The oldest version you accept. `null` when any version will do. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>FrameworkMod</code> <span>the class your mod inherits from</span></p>

Your class inherits this to get the `Log`, `Config` and `Mod` properties. The framework calls `OnLoad` once.

<div class="api-pair" markdown="1">

```csharp
public abstract class FrameworkMod : MonoBehaviour
{
    public ModInfoAttribute Info { get; }
    public CoreMod Mod { get; }
    public CoreLog Log { get; }
    public ModConfig Config { get; }
    protected abstract void OnLoad();
}
```

| Method | Returns | What it does |
|---|---|---|
| `OnLoad()` | `void` | The framework calls this once. Start your work here. |

| Property | Type | What it means |
|---|---|---|
| `Info` | `ModInfoAttribute` | The `ModInfo` you wrote on your class. |
| `Mod` | `CoreMod` | Your mod's handle for save data, keybinds and game patches. |
| `Log` | `CoreLog` | Writes to `BepInEx\logs\<your id>.log`. See [Logging](../../guides/getting-started/logging.md). |
| `Config` | `ModConfig` | Your mod's settings, stored in `BepInEx\config\<your id>.cfg`. See [Settings](settings.md). |

</div>
</div>

## Things to know

- `Mod` is the handle for your mod's [save data](../../guides/getting-started/save-data.md), [keybinds](../../guides/getting-started/keybinds.md) and game [patches](../../guides/getting-started/patching.md). `Mod.Key(...)` gives a keybind, the same as for a BepInEx [plugin](../../glossary.md#plugin). A framework mod gets its handle from `Mod`. It does not use [`CoreMod.For`](../core-helpers/coremod.md), which needs a BepInEx plugin. <span class="tech">(`CoreMod.For(BaseUnityPlugin)` takes a BepInEx type, which a framework mod cannot reference.)</span>
