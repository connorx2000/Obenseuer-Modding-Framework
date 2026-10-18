---
title: Saving
parent: Crafting stations
grand_parent: Items and crafting
nav_order: 6
---

# Saving

Speed and quality factors are not kept between scenes or in the save. This page says what is lost, what stays, and when to set the factors again.

## Things to know

- Speed and quality factors are not saved, and every full [scene](../../glossary.md#scene) load clears them. Set them again from an `Events.SceneLoaded` handler, as the quick example on [Crafting stations](../craftingstations.md#quick-example) does. Finished items keep their quality and a running craft keeps its time, because the game saves both. <span class="tech">(Every `LoadSceneMode.Single` scene load clears them.)</span>
