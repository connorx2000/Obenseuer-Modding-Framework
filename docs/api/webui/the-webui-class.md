---
title: The WebUi class
parent: Web UI
grand_parent: Screen and UI
nav_order: 3
---

# The WebUi class

`WebUi` opens views and reports on the web side as a whole: whether it works, the last error and the light or dark look. This page lists its calls and the rules behind them.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>WebUi</code> <span>opens views and tells you about the web side</span></p>
Use `Open` to show one of your HTML pages. Use `SuspendAll` and `ResumeAll` to hold and release every open view under a reason you name.

<div class="api-pair" markdown="1">

```csharp
public static class WebUi
{
    public static bool IsAvailable { get; }
    public static string LastError { get; }
    public static string Theme { get; }
    public static event Action<string> ThemeChanged;
    public static WebView Open(string modId, string folder, string page, bool modal);
    public static void SuspendAll(string reason);
    public static void ResumeAll(string reason);
}
```

| Method | Returns | What it does |
|---|---|---|
| `Open(modId, folder, page, modal)` | `WebView` | Opens a view and shows it. Gives `null` when it cannot. |
| `SuspendAll(reason)` | `void` | Hides the popup-window views for a reason. |
| `ResumeAll(reason)` | `void` | Lets go of that reason. |

| Property | Type | What it means |
|---|---|---|
| `IsAvailable` | `bool` | `true` when views can be opened. |
| `LastError` | `string` | The text of the last error. `null` when there is none. |
| `Theme` | `string` | `"light"` or `"dark"`, the look pages should use. |

| Event | Gives you | When it fires |
|---|---|---|
| `ThemeChanged` | `string` | The new look, `"light"` or `"dark"`. | The look has changed. You get the new `"light"` or `"dark"`. |

</div>
</div>

## Things to know

- `IsAvailable` is `true` on Windows when the shim (the native helper `ObxWebView.dll`) is next to the module and has not failed to start.
- It turns `false` for good when the shim fails to start, or when a serious error arrives that belongs to no view. In the default mode, the framework restarts the shim in another mode instead, so `IsAvailable` stays `true` unless that start fails too. `LastError` tells you why. <span class="tech">(The fatal-error case applies outside thread mode. In thread mode, such an error restarts the shim in composition mode for the rest of the session.)</span>
- `LastError` is `null` when there has been no error. It holds the text of the last problem the shim reported. It is set in these cases:
  - a call to the shim fails;
  - the shim reports an error;
  - the shim fails to start;
  - `Open` finds no game window yet (`no active game window`);
  - `Open` runs after the framework has shut down (`WebUi is shut down`);
  - `ObxWebView.dll` is missing.
- `Theme` comes from the WebUi module's setting `Appearance.WebTheme`, which can be `light`, `dark` or `system`. The default is `light`.
- `system` reads the Windows apps theme once at start, and again when the setting changes. It does not follow a later change of the Windows theme on its own.
- `ThemeChanged` fires with the new look after the setting changes. By then the framework has already told every open view (see [Theme](theme.md#theme)).
- `SuspendAll(reason)` hides the views that are popup windows, and `ResumeAll(reason)` lets go. Each reason is counted, so every `SuspendAll` needs one `ResumeAll` with the same reason.
- While any reason is held, the views are hidden. An empty reason is ignored. Both calls do nothing when they are not called from your mod's normal code (on the [main thread](../../glossary.md#main-thread)).
- `Open(modId, folder, page, modal)` creates a view and shows it. `modId` is your [mod id](../../glossary.md#mod-id). `folder` is the folder with your page, and `page` is the file in it to show first, such as `index.html`.
- A [modal view](../../glossary.md#modal-view) takes over the player's controls while it is shown. The player cannot move, and the mouse cursor is shown, like in a menu. A view that is not modal leaves the controls alone.
- The first `Open` starts the shim. It is not started earlier. The shim keeps its browser data in `BepInEx\cache\ObxWebView`.
- Your view gets its own web address, made from your mod id. It looks like `https://<host>/`. The framework serves your `folder` there, so `https://<host>/index.html` is `<folder>\index.html`.
- Take a mod id made of lowercase letters and digits with single dots between them. If it is at most 63 characters and not `obx.core`, each dot turns into `-` and the host ends with `.obx`. So `example.webuihello` gives `example-webuihello.obx`.
- Any other id gets `h--` and the first 16 hex digits of the SHA-256 hash of the id, then `.obx`. So `My_Mod` gives `h--faa957b3378b73e0.obx`. Different ids get different hosts. No id can map to `obx-core.obx`, the address of the [shared files](shared-assets.md). <span class="tech">(A hashed host is 64 bits of SHA-256, so a clash is not a practical risk. `https://obx-core.obx` is the host of the shared assets, written `obx-core.obx` in the code as `CoreHost`.)</span>
- You can have two open views with the same mod id. Each has its own number.

## How the views are hosted

- By default, each view is a small separate window placed over the game. Another mode can be chosen in the module's config, but the default is the one to build for. <span class="tech">(The setting is `Debug.WebUiMode`, which can be `thread`, `composition` or `windowed`. The default is `thread`. It is read once when the shim starts, so a change needs a game restart.)</span>
  - `thread`: the shim runs on its own thread. Each view is a [WebView2](../../glossary.md#webview2) control in a popup window owned by the game window. The popup is placed over the view's bounds in the game's client rectangle.
  - `composition`: the composition root runs on the game thread. This was the mode before version 0.9.
  - `windowed`: a child window of the game runs on the game thread.
  - `Debug.WebUiWindowed` (`true` or `false`, default `false`) is deprecated. Use `WebUiMode = windowed` instead. When it is `true`, the mode is `windowed` whatever `WebUiMode` says.
- Thread mode turns itself into composition mode for the rest of the session in three cases, and each is logged:
  - exclusive full screen is on at the first `Open`;
  - the thread does not start (composition is then tried at once);
  - a serious shim error arrives (the views close, and the next `Open` uses composition).

## Limits

- `Open` gives `null` when:
  - the shim cannot start. That means the computer is not Windows, the shim is missing, the game window does not exist yet, or the start failed;
  - `modal` is `true` and the game is busy. That means a menu, the pause menu, a shop, a conversation or the developer console is open. It also means no game camera is loaded, or the player's controls are off;
  - `folder` is not a folder that exists, or `page` is not a relative path (see [Navigation](navigation.md#navigation));
  - the view cannot be shown. It is closed again.
- A view that is not modal ignores the busy check.

**Common problems**

- `IsAvailable` is `false`: the computer is not Windows, or the shim is not next to the module. Or the shim failed to start, or a serious error arrived that belongs to no view. `LastError` tells you why. Then work through [WebUi is unavailable](../../faq.md#webui-is-unavailable) in the FAQ.
- `Open` gives `null`: one of the cases in the list above applies. Read `LastError`, and the log `BepInEx\logs\obenseuer.moddingframework.webui.log`.
