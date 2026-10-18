---
title: The log file
parent: Debug
grand_parent: Framework basics
nav_order: 5
---

# The log file

Where the trace writes its lines, and how long an old trace is kept. For the framework's other log files, see [Logging](../../guides/getting-started/logging.md).

- The log is `BepInEx\logs\ObenseuerCore.debug.log`. Each line has the frame number, the time and the level.
- The previous log is kept for one run. When the log is first opened in a game session, any old `ObenseuerCore.debug.log` is moved to `ObenseuerCore.debug.log.1`.
