---
title: Common tasks
parent: Web UI
grand_parent: Screen and UI
nav_order: 1
---

# Common tasks

Short snippets for the usual jobs with an open view: send it data, hide it on Escape, size it, pin it to a monitor and let it show a website. In these snippets, `view` is the [`WebView`](the-webview-class.md) that `WebUi.Open` gave you.

Tell the page something when it loads, and keep sending it fresh numbers. The page receives each one in its `message` listener.

```csharp
view.Post("{\"type\":\"state\",\"balance\":" + Wallet.Balance + "}");
```

Hide the view when the player presses Escape, and run your code when that happens.

```csharp
view.EscapeHides = true;
view.Dismissed += () => Logger.LogInfo("The player closed the page.");
```

Cover only part of the screen instead of the whole window.

```csharp
view.SetBounds(100, 100, 600, 400);
```

Show the page on a computer monitor while the player uses the computer. See [Pinning](pinning.md#pinning).

```csharp
ComputerApps.ScreenChanged += s => { if (s != null) { view.PinTo(s); view.Show(); } else view.Hide(); };
```

Let a frame (a window inside your page) show one live website. Do this before the page loads. See [Frame hosts](frame-hosts.md#frame-hosts).

```csharp
if (!view.AllowFrameHost("stalburg.net")) Logger.LogWarning("The host was refused: " + WebUi.LastError);
```
