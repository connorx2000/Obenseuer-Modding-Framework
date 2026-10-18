---
title: ModMenu
parent: Framework core classes
grand_parent: Framework basics
nav_order: 13
---

# ModMenu

`ModMenu` adds a page of your own to the Mods menu, the [framework's](../glossary.md#framework) menu for mods. Use it when your mod needs its own screen in that menu, beyond the Settings page every mod with settings already gets.

The Mods menu opens from the pause menu or the main menu. You build your page's contents with the `ModUi` building blocks on the [UI kit](../api/ui.md) page.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>ModMenu</code> <span>add a page to the Mods menu</span></p>

Call `AddPage` with your [mod id](../glossary.md#mod-id), a title and a build function to add your own page. `WidgetsFor` is an older helper, and new pages should use `ModUi` instead.

<div class="api-pair" markdown="1">

```csharp
public static class ModMenu
{
    public static void AddPage(string modId, string title, Action<RectTransform> build);
    public static ModMenuWidgets WidgetsFor(RectTransform content);
}
```

| Method | Returns | What it does |
|---|---|---|
| `AddPage(modId, title, build)` | `void` | Adds a page. `build` gets the empty area to fill, and runs each time the page is opened. |
| `WidgetsFor(content)` | `ModMenuWidgets` | An older helper for building rows on a page. `null` when the content is not a page's build area. |

</div>
</div>

### Putting it together

This plugin adds a page with one line of text.

<div class="api-example" markdown="1">

```csharp
using BepInEx;
using BepInEx.Unity.Mono;
using ObenseuerCore;
using ObenseuerCore.Ui;

public class HelloPage : BaseUnityPlugin
{
    private void Awake()
    {
        ModMenu.AddPage("example.hellopage", "Hello", host => ModUi.Text(host, "Hello from my page.", false));
    }
}
```

| Step | What it does |
|---|---|
| `ModMenu.AddPage("example.hellopage", "Hello", ...)` | Adds a page titled Hello. |
| `host => ModUi.Text(host, ...)` | Fills the page's empty area with one line of text. |

</div>

### The web Mods menu

- `ModMenuWidgets` is out of date, but still works. It is marked `[Obsolete("Use ObenseuerCore.Ui.ModUi")]`. New pages should use `ModUi`. A mod that still uses it gets compiler warning CS0618.
- The Mods menu opens as a web page by default. That is the `[UI] ModMenuStyle = web` config setting. Each [plugin](../glossary.md#plugin) with visible config entries gets one "Settings" page. A page you add with `AddPage` appears under your mod id. When no plugin has that mod id, it appears as a new entry named after your page's title.
- The older window opens when `[UI] ModMenuStyle` is `classic`, or when the web page cannot start. If the web page does not answer within 8 seconds twice in a row, the menu stays classic until the game restarts.
- The framework has a Settings page like any plugin, and it also carries actions. Its `Scene looks` section has a `Run scene looks checks` button. Clicking it runs the check for the loaded scene and shows the result on the menu status line. The classic window shows the same button under the framework's group, with the result next to it. The action needs no `[Debug]` flag.

### The classic window

- The window has a list of pages on the left and the selected page on the right. The list starts with "Mod Settings" (the framework's config editor), then every added page sorted by title.
- `AddPage` only adds a line to this list. It never adds a tab to the game's own menu. The list is read when the window opens, so a page you add while the window is open shows up the next time it opens. <span class="tech">(It is one `ModWindow` in the Full dock, used because the Center dock (640 px) cannot hold the sidebar and the settings rows. The `Side` column is the list and `Body` shows the selected page.)</span>
- In the game, the Mods tab is one entry in the pause menu. Its button opens the window, and arriving at the tab opens it too. Closing the window while the tab is selected returns the pause menu to the page it came from. Two mods never get the same tab id. <span class="tech">(The tab is one entry in `UIPageButtons.MenuPages`. Its id is the first unused `MenuPageNames` value above `MenuPageNames.Items`, checked against `MenuPages`.)</span>
- On the main menu, a "MODS" button sits under the main menu grid, in the row below "???". It opens the same window. Your own buttons can sit next to it, see [`MainMenuButtons`](mainmenubuttons.md).
- The tab is built once the game's menus exist, and built again when the game recreates them. If a menu's tab failed to build, it is not tried again for that menu. There is no way to remove a page while the game runs. <span class="tech">(It is built once `UIPageButtons` and `PauseMenu` exist.)</span>
- The pad shoulder buttons do not reach the Mods tab, but mouse and click do. The game cycles its menu tabs with a fixed list of six names (`_tabs`), and the framework does not change that list.
- Esc closes the window, and so does Joystick1Button1. The exception is when a text field has focus or a key row is waiting for a key. Then that field or row uses the press, and the window stays open. <span class="tech">(Esc is the game's "Pause Menu" key.)</span>

### Your pages

- Your `build` function gets an empty area and fills it each time the page opens. The area is cleared before each build, so rows are built once for every time the page is opened.
- If `build` throws, the page shows a notice and the window stays open. The notice says "This page failed to load. Details are in BepInEx\logs\obenseuer.moddingframework.ui.log", and the error is logged there. The other pages still work.
- If a control's callback throws, the window closes. Controls from `ModUi` run their callbacks under `ModWindow.Run`, and a callback that throws closes the window with the reason `Fault`. This is the rule for every window built with the kit, not only for pages.
- `ModUi.Text` returns a `TextMeshProUGUI`. Your project must therefore reference `Unity.TextMeshPro.dll` and `UnityEngine.UI.dll` from `Obenseuer_Data\Managed`, as the [UI kit](../api/ui.md) page explains.

**Common problems**

- Your page shows "This page failed to load": your `build` function threw. The error is in `BepInEx\logs\obenseuer.moddingframework.ui.log`.
- The window closes: a control's callback threw. A `ModUi` control's callback that throws closes the window with the reason `Fault`.

### The Mod Settings page

- It lists every setting of every loaded plugin. Settings are grouped by plugin (the first group starts open, the others closed) and then by section. Rows are sorted by section and key. <span class="tech">(A setting here is a BepInEx `ConfigEntry`. Entries that ConfigurationManager marks `Browsable = false` are not listed.)</span>
- A search box at the top filters the rows by plugin, section or key. While a search is active, every group is open.
- Each kind of setting gets a fitting control. A yes-or-no setting is a switch (`ModUi.Toggle`). A choice from a list is a stepper (`ModUi.Stepper`). A number with a range is a slider (`ModUi.Slider`), and text is a text box (`ModUi.TextField`). An enum or a list of acceptable values (`AcceptableValueList<T>`) is a stepper too.
- A keyboard shortcut is a key binding (`ModUi.KeyBinding`): click it, then press a key. Ctrl, Shift and Alt held at the press are kept. Esc cancels, and mouse buttons are ignored.
- Any other type that can be read from text is a text box, and a `[Flags]` enum is a text box too. Other types are shown but cannot be changed.
- Every changeable row has a Reset button, and its description shows as a tooltip on hover.
- Changes are saved to the [config file](../guides/getting-started/config.md) as you make them. Switches, steppers and key bindings save at once. A text box writes when you finish editing (Enter, or a click away), not on each key.
- Sliders write at once and save 0.25 seconds after the last change. Quitting to the desktop from the pause menu or the main menu saves a change that is still waiting.

### Typing and the inventory

- While you type in a box or set a key, the game's controls are locked. The game's keys and its controller update are skipped. The lock is released when the box loses focus or the capture ends.
- While your custom page is selected, the inventory is not reachable. Its tab switch and its per-frame input (Drop, Move Item, Use) do nothing.

### Dark theme

- The setting `[UI] ThemeConfigManager = true` gives the ConfigurationManager (F1) window a dark theme. It takes effect on the next start. Without ConfigurationManager, nothing is changed. <span class="tech">(The framework patches the private `OnGUI` of ConfigurationManager through `CorePatcher`. A prefix swaps `GUI.skin` for a dark skin built from the menu palette, and a finalizer puts back the previous skin.)</span>

### Still to be checked in the game

- The in-game Mods tab: the window opens on arrival, closing it returns to the previous page, and the pause menu ignores keys while the window is open.
- The main menu MODS button: its place under the grid, the click, left and right navigation to Multiplayer, and the navigation restore on close.
- Esc and Joystick1Button1 closing the window, and Esc inside a text box and inside a key capture.
- That the Pause Menu does not open on the same Esc press.
- Search, grouping, every kind of row, the 0.25 second save, and Reset.
- The page failure notice, and a throwing callback closing the window with the reason `Fault`.
