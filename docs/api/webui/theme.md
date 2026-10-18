---
title: Theme
parent: Web UI
grand_parent: Screen and UI
nav_order: 9
---

# Theme

A page that links [`obx-theme.css`](shared-assets.md) gets its colours from the `data-theme` setting on `<html>`. This page explains how the framework sets that attribute, and how your page hears about a change of look.

## Things to know

- The first `data-theme` is set before your page runs. The framework gives every view a small start-up script. That script sets the attribute on `<html>` as soon as the element exists, from [`WebUi.Theme`](the-webui-class.md).
- When the setting changes, the framework sends your page a message. It looks like `{"v":1,"type":"theme","mode":"light"}` (or `"dark"`), and it goes to every open view. Your page should set the attribute from that message.
- The framework does not change the attribute itself after the page has loaded.
- A page you load later also starts in the new look. The framework replaces each open view's start-up script with the one for the new look. A view keeps one such script.

**Common problems**

- The page keeps its old look after the setting changes: the framework does not change the attribute after the page has loaded. Set `data-theme` on `<html>` from the `theme` message.
