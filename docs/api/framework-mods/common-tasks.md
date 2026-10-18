---
title: Common tasks
parent: Framework mods
grand_parent: Framework basics
nav_order: 1
---

# Common tasks

Short code for settings, dependencies on other mods, and using a mod only when it is installed. The code goes in a mod like the one from [Your first mod](../../guides/getting-started/first-mod.md).

Add settings with `Config.Bind`. The second setting below is a slider in the [Mods menu](mods-menu.md), because it has a smallest and a largest value. `Changed` runs your code when the player moves it. The [Config](../../guides/getting-started/config.md) guide walks through settings in more detail.

```csharp
ModSetting<bool> loud = Config.Bind("General", "Loud", false, "Log more.");
ModSetting<int> count = Config.Bind("General", "Count", 3, "How many.", 1, 10);
loud.Changed += () => Log.Info("Loud is now " + loud.Value);
Log.Info("Count is " + count.Value + " (default " + count.Default + ")");
```

Depend on other mods with `[ModDependency]`. A mod with no version after its id only has to be there. The mods you name here load before yours.

```csharp
[ModInfo("example.tasks", "Tasks", "1.2.3")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
[ModDependency("example.otherlibrary")]
[ModDependency("example.anotherlibrary", "2.0")]
public sealed class Tasks : FrameworkMod
```

Use a mod only when it is there, without making it a must-have. [`Loader.Has`](dependencies.md) answers that, from inside [`Events.WhenReady`](../core-helpers/events-whenready.md).

```csharp
Events.WhenReady(() =>
{
    if (Loader.Has("example.optionalfriend")) Log.Info("The optional mod is running.");
});
```
