---
title: Details
parent: Items
grand_parent: Items and crafting
nav_order: 6
---

# Details

This page gives the target game version, the log file and the rules for calling `Items`. Check it when a lookup gives an empty result or you need to know when a call is safe.

Target: Obenseuer 0.5.08 (buildid 25969064), Unity 6000.0.77f1 Mono, [BepInEx](../../glossary.md#bepinex) (the game's mod loader) 6 Unity.Mono ([be.788](https://builds.bepinex.dev/projects/bepinex_be/788/BepInEx-Unity.Mono-win-x64-6.0.0-be.788%2B5b766a3.zip), the tested build) or BepInEx 5 (5.4.23 or later).
Namespace: `ObenseuerCore`. Logs go to `BepInEx\logs\obenseuer.moddingframework.items.log`.

Everything not listed in this reference is internal and may change without notice. Items marked experimental may change too.

## When you can call it

- Call everything from your mod's normal code (`OnLoad`, event handlers, `Update`), not from a background thread you started. The game only runs safely on the main thread (see the [glossary](../../glossary.md#main-thread)).

## What to check for and what comes back

- The catalogue is filled only after the game's item database has started. Before that, or on the main menu when no scene has loaded it, every read gives its plain default: an empty list or `null`. <span class="tech">(The catalogue is `ItemDatabase.database`, filled from `StreamingAssets/Items.json` when the game's `ItemDatabase` component starts.)</span>
- The reads only look. They add no items, raise no events and change no game methods. The framework installs nothing for this part, so there is no [feature group](../../glossary.md#feature-group) (a named set of the framework's changes) that can turn off.
- Each call builds new `ItemDef` objects. `Price` and `MaxStack` show the difficulty setting at the time of the call.
- If the game fails while the framework reads one item, that item is left out of the results. The first such failure of any read goes to the [log](../../guides/getting-started/logging.md). Later ones do not. <span class="tech">(All members share one log key, `Items.Build`.)</span>

**Common problems**

- Every read gives an empty list or `null`: the game's item database has not started yet, or you are on the main menu and no scene has loaded it. Run your lookups after a save has loaded, for example inside [`Events.WhenReady`](../core-helpers/events-whenready.md), as the Quick example on the [Items](../items.md) page does.
