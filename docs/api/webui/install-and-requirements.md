---
title: Install and requirements
parent: Web UI
grand_parent: Screen and UI
nav_order: 2
---

# Install and requirements

What a player must install before a [WebUi](../../glossary.md#webui) page can open, and what a mod that uses WebUi must declare. Read it before you ship a mod that opens views.

This is the full list of what the API gives you. Text in brackets at the end of a note is extra detail you can skip on a first read.

WebUi is a separate [BepInEx plugin](../../glossary.md#plugin), so a player has to install it. Without it, the WebUi types are missing from the game, and the Mods menu uses the classic window instead. It is needed only by mods that use WebUi and by the web Mods menu.

- Install the WebUi module from its own zip, named `ObenseuerModdingFramework-WebUi-<version>-beta.zip`. It goes to `BepInEx\plugins\ObenseuerCore.WebUi\`, with `ObxWebView.dll` and the `web\` folder. The framework itself must be installed too. <span class="tech">(The plugin is `ObenseuerCore.WebUi.dll`, with the GUID `obenseuer.moddingframework.webui` and version 0.11.1, and it depends on the framework. It works on Windows only.)</span>
- `ObxWebView.dll` is the small native helper that talks to [WebView2](../../glossary.md#webview2) (the Microsoft Edge drawing component). This page calls it the shim. The module ships it next to `ObenseuerCore.WebUi.dll`.
- A mod that uses WebUi must say so, and reference the module's DLL. Add `[BepInDependency("obenseuer.moddingframework.webui", ">=0.11.0")]` to your plugin, and reference `ObenseuerCore.WebUi.dll`.
- The WebUi types are not in `ObenseuerCore.dll`. On BepInEx 5 write `"0.11.0"` instead, because BepInEx 5 cannot read a version range. See [Supporting BepInEx 5 too](../../guides/getting-started/packaging.md#supporting-bepinex-5-too).
- The WebView2 Runtime has to be on the player's computer. It comes with Windows 10 and 11.

**Common problems**

- The Mods menu opens as the classic window, not with the web look: WebUi is not installed, or the WebView2 Runtime is missing. The shim may also be missing or unable to start, and WebUi works on Windows only. Install the WebUi zip as described in [Optional: WebUi](../../install.md#optional-webui), and check that the WebView2 Runtime is installed.
- A view does not open: read `BepInEx\logs\obenseuer.moddingframework.webui.log`. In your code, `WebUi.LastError` says why. The causes are listed under Limits on [The WebUi class](the-webui-class.md#limits). See also [WebUi is unavailable](../../faq.md#webui-is-unavailable).
