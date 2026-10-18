---
title: Navigation
parent: Web UI
grand_parent: Screen and UI
nav_order: 7
---

# Navigation

[`Navigate`](the-webview-class.md) and [`Open`](the-webui-class.md) take a relative path, such as `index.html` or `pages/shop.html`. This page lists the paths they refuse, and what else the shim blocks.

A path is refused when it:

- is empty, or starts with `/` or `\`;
- contains `:`, `\`, `%` or `..`.

A refused path gives `null` from `Open` and `false` from `Navigate`.

The shim (the [WebView2](../../glossary.md#webview2) helper `ObxWebView.dll`) also blocks these:

- navigation outside the addresses it mapped for your view (frames can also load the hosts you named with `AllowFrameHost`, see [Frame hosts](frame-hosts.md#frame-hosts));
- new windows;
- downloads;
- permission requests;
- links to other programs.
