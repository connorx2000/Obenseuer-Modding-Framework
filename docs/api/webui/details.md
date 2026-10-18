---
title: Details
parent: Web UI
grand_parent: Screen and UI
nav_order: 11
---

# Details

The build [WebUi](../../glossary.md#webui) targets and where its log goes. Also what can fail, when it is safe to call it, and what happens with bad values.

Target: Obenseuer 0.5.08 (buildid 25969064), Unity 6000.0.77f1 Mono, [BepInEx](../../glossary.md#bepinex) 6 Unity.Mono ([be.788](https://builds.bepinex.dev/projects/bepinex_be/788/BepInEx-Unity.Mono-win-x64-6.0.0-be.788%2B5b766a3.zip), the tested build) or BepInEx 5 (5.4.23 or later).
Namespace: `ObenseuerCore`. Logs go to `BepInEx\logs\obenseuer.moddingframework.webui.log`.

Everything not listed in this reference is internal and may change without notice.

## If something fails to set up

The framework does not change the game's own methods for this part of the API. What can fail is the shim (the [WebView2](../../glossary.md#webview2) helper `ObxWebView.dll`). If it is missing, or it cannot start, [`WebUi.IsAvailable`](the-webui-class.md) is `false`, `Open` gives `null`, and `LastError` tells you why. Its name in the log is `obenseuer.moddingframework.webui`.

**Common problems**

- `WebUi.IsAvailable` is `false` and `Open` gives `null`: the shim is missing, or it cannot start. Read `LastError` for the reason, then work through [WebUi is unavailable](../../faq.md#webui-is-unavailable) in the FAQ.

## When you can call it

- Call everything from your mod's normal code (OnLoad, event handlers, Update), not from a background thread you started. The game and Unity only run safely on the main thread (see the [glossary](../../glossary.md#main-thread)). From another thread, `Open` gives `null`, the calls that return a `bool` give `false`, and `Close` does nothing.

## Bad values

- A mod id that is not valid throws an error. `Open` throws an `ArgumentException` when `modId` is not 1 to 64 characters of letters, digits, `.`, `_` or `-`, as the other framework calls do. See [Mod id](../../glossary.md#mod-id).
- A folder or page that is not valid gives `null`. `Open` gives `null` when `folder` is not a folder that exists, or when `page` is not a relative path (see [Navigation](navigation.md#navigation)).
- Nothing else throws an error. A call into the shim that fails gives `false` or `null`, and `LastError` then holds the shim's error text.
