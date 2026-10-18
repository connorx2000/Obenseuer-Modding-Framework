---
title: Checking the trace itself
parent: Debug
grand_parent: Framework basics
nav_order: 6
---

# Checking the trace itself

Two ways to test that the trace code works: a self-check inside the game and an offline harness that runs without it. Most modders never need either.

- `[Debug] DebugTraceSelfCheck` makes the framework test its own trace code at start. It runs the line formatter, the rate limiter and the filter once. It writes whether they passed to the [log](../../guides/getting-started/logging.md).
- The offline harness runs the same engine without the game. It prints one PASS or FAIL line for each check. It exits with code 1 if any check fails.
- The harness reads `0Harmony.dll` (the [Harmony](../../glossary.md#harmony) library) from `BepInExCore`, which defaults to `GameDir\BepInEx\core`. Pass `-p:BepInExCore=<folder>` to use another folder.

To run the harness, use this command from the framework's source folder:

```
dotnet run --project tools/DebugTraceHarness -c Release
```
