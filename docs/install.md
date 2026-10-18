---
title: Install
nav_order: 2
---

# Install

This page installs [BepInEx](glossary.md#bepinex) (the game's mod loader) and the [framework](glossary.md#framework) into the game folder. Players who run mods that need the framework, and people who make mods, both need these steps. Mod authors need one extra tool, named in What you need.

## What you need

1. Obenseuer 0.5.08 (Steam build 25969064).
2. BepInEx, the free tool that loads mods into the game. Either version works, and the framework zip works with both:
   - BepInEx 5: version 5.4.23 or later (Mono, x64). Download the zip from the [BepInEx releases page](https://github.com/BepInEx/BepInEx/releases). Its name starts with `BepInEx_win_x64_5.4`.
   - BepInEx 6: Unity.Mono, x64. The tested build is [be.788](https://builds.bepinex.dev/projects/bepinex_be/788/BepInEx-Unity.Mono-win-x64-6.0.0-be.788%2B5b766a3.zip). Download it from [builds.bepinex.dev](https://builds.bepinex.dev/projects/bepinex_be). Newer BepInEx 6 builds usually work. BepInEx 6 is a pre-release, so a new build can break mods. If something breaks, use be.788.

   Obenseuer is a 64-bit Windows game that runs on Unity's Mono. Pick "Mono" and "x64". Do not pick the IL2CPP or x86 files.
3. The framework zip from the [GitHub releases page](https://github.com/NomadsReach/Obenseuer-Modding-Framework/releases). It holds a BepInEx 6 build and a BepInEx 5 build.
4. Only if you make mods: the [.NET 10 SDK](https://dotnet.microsoft.com/download), the free tool that builds your code into a mod. Players do not need it.

You now have everything needed. Next, install for players, below.

## Install for players

1. Close the game.
2. Open the game folder, the folder that holds `Obenseuer.exe`. The default Steam path is `C:\Program Files (x86)\Steam\steamapps\common\Obenseuer`. In Steam, right-click the game, choose Manage, then Browse local files to open the folder. Extract the BepInEx zip there, so that the `BepInEx` folder and `winhttp.dll` sit next to `Obenseuer.exe`.
3. Start the game once, so BepInEx creates its folders. Then close it. The game folder now has a `BepInEx\plugins` folder and a `BepInEx\LogOutput.log` file. If it does not, see Common problems, below.
4. Extract the framework zip into the game folder. The zip contains a `BepInEx` folder, so its files land in `BepInEx\plugins\ObenseuerCore\`. The zip holds a BepInEx 6 build (`ObenseuerCore.dll`) and a BepInEx 5 build (`ObenseuerCore.BepInEx5.dll`). BepInEx loads the one that matches. `ObenseuerCore` is the framework's file and code name.
5. Start the game. The framework adds a "MODS" button to the main menu.

The framework is installed. The section Check it worked, below, confirms it from the log.

If you use ConfigurationManager or UnityExplorer, they need the builds for your BepInEx version.

### Common problems

- No `BepInEx\plugins` folder or `BepInEx\LogOutput.log` after the first start: check that `winhttp.dll` sits next to `Obenseuer.exe` and not one folder deeper. Extract the BepInEx zip again into the game folder itself.
- `BepInEx\logs\ObenseuerCore.log` is missing: BepInEx is not installed correctly. Start the game once after you extract BepInEx, and see [my mod does not load](faq.md#my-mod-does-not-load).
- The log reports an error about the other loader's copy: delete the file for the loader you do not use. The BepInEx 6 file is `ObenseuerCore.dll`, the BepInEx 5 file is `ObenseuerCore.BepInEx5.dll`. The same goes for `ObenseuerCore.WebUi.dll` and `ObenseuerCore.WebUi.BepInEx5.dll`. See [BepInEx 5 or 6?](faq.md#bepinex-5-or-6).
- Mods break after moving to a newer BepInEx 6 build: BepInEx 6 is a pre-release. Go back to be.788, the tested build.
- The Mods menu opens as the classic window, not with the web look: WebUi is not installed, or the WebView2 runtime is missing. The WebUi shim may also be missing or unable to start. WebUi works on Windows only. See Optional: WebUi, below, and [WebUi is unavailable](faq.md#webui-is-unavailable).
- The first line of `BepInEx\logs\ObenseuerCore.log` names a game build other than 0.5.08: the framework is built for 0.5.08. See [A feature stopped working after a game update](faq.md#a-feature-stopped-working-after-a-game-update).

### Updating

1. Close the game.
2. Delete `BepInEx\plugins\ObenseuerCore\`. Also delete `BepInEx\plugins\ObenseuerCore.WebUi\` if it exists. Extracting over the old folders does not remove files that a newer version dropped. For example, `ObxWebView.dll` and the `web` folder came with the framework zip in 0.9.0-beta, and moved to the WebUi zip in 0.10.0-beta.
3. Extract the new zips into the game folder.

Your settings in `BepInEx\config\` are kept. If the log then reports an error about the other loader's copy, see Common problems, above.

The update is done. The section Check it worked, below, shows which version loaded.

### Optional: WebUi

[WebUi](glossary.md#webui) lets mods show HTML pages. It also gives the framework's Mods menu its web look, styled like the game's own menu. Without it, the framework still runs and the Mods menu opens as the classic window. Install it if you want the web Mods menu, or if a mod asks for it.

Extract the WebUi zip from the [releases page](https://github.com/NomadsReach/Obenseuer-Modding-Framework/releases) into the game folder. Its files land in `BepInEx\plugins\ObenseuerCore.WebUi\`, with a BepInEx 6 build (`ObenseuerCore.WebUi.dll`) and a BepInEx 5 build (`ObenseuerCore.WebUi.BepInEx5.dll`). WebUi also needs the Microsoft Edge [WebView2](glossary.md#webview2) Runtime. Windows 10 and 11 have it already.

## Check it worked

Open `BepInEx\logs\ObenseuerCore.log` in the game folder. The first line names the framework version, the game build and the loader (BepInEx 5 or 6) it runs on. If the file is missing, see [my mod does not load](faq.md#my-mod-does-not-load).

## Make mods

Do the steps above first, including the framework zip. A mod project builds against the framework files in your game folder. Then go to [Your first mod](guides/getting-started/first-mod.md). It starts from the `Starter` folder in the repository. That folder builds on its own once the game, BepInEx and the framework are installed. The framework does not ship a `build.cmd`. Build your mod with `dotnet build`.

## Where to go next

Next: [Your first mod](guides/getting-started/first-mod.md).
