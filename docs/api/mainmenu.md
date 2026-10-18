---
title: Main menu
parent: Framework basics
grand_parent: API reference
has_children: true
nav_order: 8
---

# Main menu

[`MainMenuButtons`](mainmenu/mainmenubuttons.md) puts your mod's own buttons on the game's main menu. Use it when your mod has a screen the player should open from there.

## What it is

The main menu is the first screen the player sees, with buttons such as New Game, Load Game and Quit. `MainMenuButtons` gives your mod two jobs:

- Add a button of its own to the menu.
- Switch the menu's own controls off and on while your mod shows something over the menu.

The framework's own [MODS button](mainmenu/the-mods-button.md) is added the same way.

## When to use it

- Add a button to the main menu that opens your mod's own screen.
- Stop the menu reacting to the keyboard or controller while your own window is open on top of it.
- Check whether the main menu is up before you try to show something over it.

## Quick example

This mod adds a button called HELLO to the main menu. Clicking it writes a line to the [log](../guides/getting-started/logging.md).

```csharp
using ObenseuerCore;

[ModInfo("example.menubutton", "Menu Button", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class MenuButton : FrameworkMod
{
    protected override void OnLoad()
    {
        MainMenuButtons.Add(Mod.Id, "Hello", "HELLO", () => Log.Info("The HELLO button was clicked."));
    }
}
```

[`Add`](mainmenu/add.md) takes four arguments, in this order:

- Your [mod id](../glossary.md#mod-id).
- A short name for the button. It must be different from every other button's name.
- The text on the button.
- Your code, which runs when the player clicks the button.

You can call `Add` from `OnLoad`, even though the menu may not exist yet. The framework builds the button when the menu appears.
