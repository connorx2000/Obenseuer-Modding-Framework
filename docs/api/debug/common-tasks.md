---
title: Common tasks
parent: Debug
grand_parent: Framework basics
nav_order: 1
---

# Common tasks

Short recipes for the usual trace jobs: check its state, narrow it to some types, and keep it on across restarts. All of them work from the console or the Mods menu, with no code.

Check whether the trace is on, and which framework methods are called the most. Type this in the [developer console](../hud/consolecommands.md) (opened with F1):

```
core_debug status
```

Trace only the parts you care about. Open the [Mods menu](../framework-mods/mods-menu.md) (the "MODS" button on the main menu), find `[Debug] TraceFilter` and enter type names separated by commas, for example `PlayerState,GameTime`. Only types whose names start with one of those words are traced.

Keep the trace on across restarts. Open the Mods menu and switch `[Debug] Master` on. The setting is saved, so the trace stays on the next time you start the game. Switch it off again when you are done.
