---
title: Add a Mods menu page
parent: Cookbook
nav_order: 4
---

# Add a Mods menu page

Add a page to the Mods menu (the "MODS" button on the main menu) with a title, a line of text and a button. Use it when your mod needs controls of its own that a plain setting cannot give.

## Add the references

The page uses [`ModUi`](../../api/ui/controls.md) helpers. `ModUi` draws with the game's text and button types, so your project needs two more references first.

Open your `.csproj` file in a text editor. Find the `<ItemGroup>` that holds the other `<Reference>` lines. Add these two references inside it, before its closing `</ItemGroup>`:

```xml
<Reference Include="Unity.TextMeshPro">
  <HintPath>$(ManagedDir)\Unity.TextMeshPro.dll</HintPath>
  <Private>false</Private>
</Reference>
<Reference Include="UnityEngine.UI">
  <HintPath>$(ManagedDir)\UnityEngine.UI.dll</HintPath>
  <Private>false</Private>
</Reference>
```

They point at game files in the `Managed` folder. `Private` set to `false` keeps the build from copying the game's files into your output.

The project has the references it needs. Next, add the page code.

## The example

[`ModMenu.AddPage`](../../api-core/modmenu.md) registers the page. The method you give it builds the page when the player opens it, using `ModUi` helpers.

Replace the code in `Plugin.cs` of your project from [Your first mod](../getting-started/first-mod.md) with this code.

```csharp
using ObenseuerCore;
using ObenseuerCore.Ui;
using UnityEngine;

[ModInfo("example.menupage", "Menu Page", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public class MenuPage : FrameworkMod
{
    protected override void OnLoad()
    {
        // Add a page: your mod id, the page title, and the method that builds the page.
        ModMenu.AddPage("example.menupage", "Example Page", BuildPage);
    }

    // Runs when the player opens the page. "parent" is the empty page to fill.
    private void BuildPage(RectTransform parent)
    {
        // A heading.
        ModUi.Title(parent, "Example Page");
        // A line of text. false means normal text, true would make it dim.
        ModUi.Text(parent, "This page belongs to the example mod.", false);
        // A button. Clicking it writes a line to this mod's log.
        ModUi.Button(parent, "Log a click", ButtonStyle.Primary, () => Log.Info("menu page button clicked"));
    }
}
```

Then build and open the page:

1. Close the game and run `dotnet build -c Release -p:DeployToGame=true` in your mod folder.
2. Start the game. Open the Mods menu and find Menu Page in the list.
3. Click Log a click, then check `BepInEx\logs\example.menupage.log` in the game folder.

Your page is in the Mods menu. The next section says where such a page can show.

### Common problems

- Menu Page is not in the list: check that the mod loaded, see [my mod does not load](../../faq.md#my-mod-does-not-load).
- A control cannot be reached with a controller: `ModUi` controls are reachable with a controller, but a raw `Selectable` that the page builds itself is not. Build the page from `ModUi` controls.

## Classic pages only

Pages added with `ModMenu.AddPage` are classic only. The build method runs in the classic uGUI window. That holds whether the player opens the page from the classic sidebar or from the [WebUi](../../glossary.md#webui) menu. The WebUi menu opens the classic pane for a page it does not render itself. A page never renders as HTML.

Keep a classic page to labels, buttons and the `ModUi` kit. A mod that needs a rich control, such as a searchable dropdown, a colour picker or multi-line text, builds that in the WebUi menu instead. See [the WebUi menu](../../api/webui.md).

## Things to know

- [`ModMenu.AddPage`](../../api-core/modmenu.md) takes the [mod id](../../glossary.md#mod-id), the page title and a method that builds the page. The method runs when the player opens the page.
- Other controls are in [UI and HTML views](../ui.md) and the [UI reference](../../api/ui.md).
- Config entries show up in the Mods menu on their own. A page is only needed for your own controls. See [Config](../getting-started/config.md).
- The classic window works with a controller as well as a mouse. The d-pad and the left stick move the focus. A or Enter activates the focused control, B or Esc goes back, and the bumpers switch sidebar pages. The footer shows the keys.

## Where to go next

Next: [Add an HTML panel](html-panel.md).
