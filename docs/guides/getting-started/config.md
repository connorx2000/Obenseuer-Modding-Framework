---
title: Config
parent: Getting started
nav_order: 3
---

# Config

A config setting is a value that the player can change without touching your code, such as an on/off switch or a number. You create one with `Config.Bind`. The player changes it in the Mods menu (the "MODS" button on the main menu), and the [framework](../../glossary.md#framework) saves it for you.

## The example

This page continues the project from [Your first mod](first-mod.md). Replace the code in `Plugin.cs` with the code below. `Config.Bind` takes the section name, the setting name, the default value and a description. The two extra numbers on the middle line are the lowest and highest value allowed.

```csharp
using ObenseuerCore;

[ModInfo("example.hello", "Example Hello", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public class ExampleHello : FrameworkMod
{
    // The three settings. Keep them in fields so other methods can read them later.
    private ModSetting<bool> greetOn;
    private ModSetting<int> greetEvery;
    private ModSetting<string> greeting;

    protected override void OnLoad()
    {
        // An on/off switch, on by default.
        greetOn = Config.Bind("General", "Enabled", true, "Turn the greeting on or off.");
        // A number from 1 to 24, 6 by default.
        greetEvery = Config.Bind("General", "GreetEveryHours", 6, "Hours between greetings.", 1, 24);
        // A piece of text.
        greeting = Config.Bind("General", "Greeting", "Hello", "Text to show.");
        Log.Info("on " + greetOn.Value + ", every " + greetEvery.Value + " hours, text " + greeting.Value);
    }
}
```

Then build and check the result:

1. Close the game and run `dotnet build -c Release -p:DeployToGame=true` in your mod folder.
2. Start the game. The log line in `BepInEx\logs\example.hello.log` shows the three values.
3. Open the Mods menu, find Example Hello, and change a setting.

Your settings are live. The points below say where the value goes and how to read it.

### Common problems

- Example Hello is not in the Mods menu: a setting only appears once your mod has loaded and created it. First check [my mod does not load](../../faq.md#my-mod-does-not-load).
- The Mods menu opens as the classic window, not the web version: the classic window opens when the web version does not. See [My settings do not show in the Mods menu](../../faq.md#my-settings-do-not-show-in-the-mods-menu).

## Things to know

- **The value is saved for you.** It goes in `BepInEx\config\<mod id>.cfg` in the game folder, named after your [mod id](../../glossary.md#mod-id). The player's choice stays between game starts.
- **Read `.Value` each time you need it, not once at the start.** The player can change a setting in the Mods menu while the game runs. If you read it only once in `OnLoad`, your mod keeps using the old value.
- **You do not register anything for the Mods menu.** Every setting you bind shows up there on its own, under your mod's name.
- **The control depends on the type.** A `bool` is a toggle. An `int` or `float` bound with a lowest and highest value is a slider. A `string` is a text field. An enum (a fixed list of choices) is a stepper.
- **Do not name a setting field `enabled`.** Your mod class already has a built-in `enabled` switch from Unity, the game engine. Your field would hide it, so the compiler warns. This page uses `greetOn`.
- **To run code when a setting changes, use `Changed`.** For example `greetOn.Changed += () => Log.Info("changed");`.
- **The debug switches are listed on the [Debug page](../../api/debug.md).** That includes the framework's master trace switch.

## Changing a default in a new version

If you change a default value in a new version of your mod, players who already have the old file keep their old value. The saved value wins over the default in your code. To change it for them too, upgrade the saved value, but only when it still holds the old default. A value the player chose on purpose then stays.

The way to do it: bind a `ConfigVersion` setting, add one `if` line for each version, then write the current version. This example changes `GreetEveryHours` from 6 to 4 in version 2. Add the new field to the class above:

```csharp
private ModSetting<int> configVersion;
```

Then replace the `GreetEveryHours` line in `OnLoad` with the first line below, so a fresh install gets the new default. Add the rest after the three binds and before the `Log.Info` line:

```csharp
greetEvery = Config.Bind("General", "GreetEveryHours", 4, "Hours between greetings.", 1, 24);
configVersion = Config.Bind("General", "ConfigVersion", 0, "Internal. Do not edit.");
if (configVersion.Value < 2 && greetEvery.Value == 6) greetEvery.Value = 4;
configVersion.Value = 2;
```

The upgrade is in place. Build again to try it. Two more points about this pattern:

- **Each `if` checks the saved version, and the version is written only at the end.** A player who skips versions still gets every upgrade after the one they had saved. For version 3, add `if (configVersion.Value < 3 && ...)` on its own line, and change the written number to 3.
- **Players can see `ConfigVersion` in the Mods menu.** A [framework mod](../../api/framework-mods.md) cannot hide a setting. Hiding one needs a [BepInEx plugin](../../glossary.md#plugin), see [Packaging](packaging.md#bepinex-plugins-advanced).

## Where to go next

Next: [Keybinds](keybinds.md).
