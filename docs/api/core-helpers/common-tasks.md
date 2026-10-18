---
title: Common tasks
parent: Framework helpers
grand_parent: Framework basics
nav_order: 1
---

# Common tasks

Short snippets for the usual jobs with the framework helpers: patch a game method through your [`CoreMod`](coremod.md) handle, run code once a save is ready, and get a handle in a plain plugin.

Run a method of yours right before a game method runs. The last two arguments say which method of yours to run: its type and its name.

```csharp
bool ok = Mod.Prefix(typeof(TimeOfDayAzure), "IncreaseDayCounter", typeof(Helpers), "AfterDay");
```

A game method can have several versions with the same name. For those, use the version of `Prefix`, `Postfix` or `Finalizer` that also takes an array of argument types, such as `new[] { typeof(int) }`. The array says which version you mean. The [CorePatcher short helpers](corepatcher-short-helpers.md) page lists every version.

Run something once a [save](../../guides/getting-started/save-data.md) is ready. If a save is already loaded, it runs straight away. See [Events.WhenReady](events-whenready.md).

```csharp
Events.WhenReady(() => Log.Info("save ready"));
```

If your mod is a [BepInEx plugin](../../glossary.md#plugin) (BepInEx is the game's mod loader) and not a [framework mod](../framework-mods.md), get the handle from the plugin.

```csharp
CoreMod mod = CoreMod.For(this);
mod.Log.Info("ready");
```
