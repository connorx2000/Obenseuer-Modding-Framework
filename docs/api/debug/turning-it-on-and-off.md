---
title: Turning it on and off
parent: Debug
grand_parent: Framework basics
nav_order: 2
---

# Turning it on and off

The ways to switch the trace on and off, and the filter that limits it to some types. Text in brackets at the end of a note is extra detail for experienced modders; skip it on a first read.

There are three ways to switch the trace, and they all change the same saved setting.

- The [Mods menu](../framework-mods/mods-menu.md) (the "MODS" button on the main menu) has a switch called `[Debug] Master`. It is off by default. A change applies at once, with no restart.
- The console command `core_debug` turns the trace on or off and shows its state. Type it in the [developer console](../hud/consolecommands.md) (opened with F1). Use `core_debug on`, `core_debug off` or `core_debug status`. Any other word prints a short usage line. <span class="tech">(`status` prints on or off, the number of game methods the trace is watching, and the 10 methods called most often.)</span>
- The setting is saved in the framework's [config](../../guides/getting-started/config.md) file, so the trace stays on after a restart. Turn it off when you are done, or every start of the game will be slow. <span class="tech">(The file is `BepInEx\config\obenseuer.moddingframework.cfg`, section `Debug`.)</span>
- `[Debug] TraceFilter` limits the trace to some types. It is a list of type name beginnings separated by commas, for example `PlayerState,GameTime`. Leave it empty to trace everything. If you change it while the trace is on, the trace restarts with the new list.

**Common problems**

- Every start of the game is slow: the trace setting is saved, so the trace stays on after a restart. Turn it off with `core_debug off` or the `[Debug] Master` switch.
- `core_debug` prints a short usage line: the word after it was not `on`, `off` or `status`. Type one of those three.
