---
title: Add
parent: Main menu
grand_parent: Framework basics
nav_order: 3
---

# Add

`Add` puts a button on the [main menu](../mainmenu.md). It gives back `true` when the button was stored. Buttons are never removed once they are added.

## Things to know

- Button names (ids) are shared by every mod, and the first mod to use a name keeps it. If a second `Add` uses a taken name, it gives back `false`. One warning then goes to the [log](../../guides/getting-started/logging.md), naming both mods. If the same mod adds its own name again, it gets `false` with no warning.
- The version without `modId` cannot say who you are, so its repeats always warn and name "a mod without an id". Use the version with `modId`.
- It is safe to call `Add` early, such as from `OnLoad`. Nothing is built until the main menu exists.
- Your button is rebuilt each time the main menu loads, including when the player returns to the menu from a game. The framework copies one of the menu's own buttons and puts your label on it, so it looks like the rest of the menu. <span class="tech">(Each entry is cloned again from a main menu button: the one whose persistent call is `LoadGame`, otherwise the first one that calls the menu. The clone is named `Mod:<id>`, shows `label` in the template's text, and has its localization components removed.)</span>
- Your buttons sit in a row under the menu's "unknown" cell. They go from left to right, in the order they were added. A button that another mod put on the menu is never moved. Your buttons skip the spots those buttons take. They also move out of the way if such a button appears or moves later. <span class="tech">(The first row button sits under the "unknown" grid cell, in the left column of the menu's grid (3 columns by 2 rows). Before placing, every other `Mod:` button in the row (one this API did not create, such as co-op's `Mod:Multiplayer`) marks its grid column as taken. Each button takes the first column from "unknown" to the right that no taken column covers. When the number of children under the row parent changes, or the sum of the rounded x positions of the foreign `Mod:` buttons changes, the row is laid out again.)</span>
- The keyboard and controller can move across your buttons. Left and Right move along the row, past your buttons and other mods' buttons alike. Down from the "unknown" cell goes to the leftmost button in the row. Up from a button in the row goes to the closest of the "unknown", "credits" and "quit" cells, by horizontal position.
- If your click code throws an error, the game carries on. The error is written to the log and nothing else is affected. <span class="tech">(The callback runs inside a try/catch. A throw is logged to `BepInEx\logs\obenseuer.moddingframework.ui.log` as `main menu button <id> failed: <type>: <message>`.)</span>

## Limits

- `Add` gives back `false` when `id` or `label` is null or empty, when `onClick` is null, or when the `id` is already taken.
- It throws an `ArgumentException` when `modId` is not a valid [mod id](../../glossary.md#mod-id): 1 to 64 characters of letters, digits, `.`, `_` or `-`.

**Common problems**

- `Add` gives back `false`: `id` or `label` is null or empty, `onClick` is null, or the `id` is already taken. A name must be different from every other button's name, so pick one that no other mod uses.
