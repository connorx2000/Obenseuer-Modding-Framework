---
title: Screen effects
parent: Screen and UI
grand_parent: API reference
has_children: true
nav_order: 3
---

# Screen effects

`ScreenFx` changes how the screen looks while the game runs: shakes, fades, zoom, tints, blur and the death message. Use it for hits, scene changes, a zoom key or your own menus.

This page uses `KeyCode` from Unity. The [glossary](../glossary.md#types-that-come-from-unity) says what each one is.
{: .unity-types}

## What it is

ScreenFx gives your mod these effects:

- [Shake](screen-fx/shake.md) the camera.
- [Fade](screen-fx/fading-to-black.md) the screen to black and back.
- Change the [message](screen-fx/death-message.md) shown when the player dies.
- Change the camera's [field of view](screen-fx/fov-override.md) (how wide the view is). This is how you make a zoom.
- Darken the edges of the screen with a coloured [vignette](screen-fx/vignette.md).
- [Blur](screen-fx/blur.md) the picture.

Effects that take an `owner` belong to your mod. Clear them with the matching `Clear` call when you are done. The `owner` is your mod's id (see the [glossary](../glossary.md#owner)).

## When to use it

- Add a zoom key by narrowing the field of view while a key is held.
- Shake the screen for an explosion or a hit, or fade to black for a scene change.
- Tint the screen edges red when the player is hurt, or blur the picture while your own menu is open.
- Change the text on the death screen to fit your mod.

## Quick example

This mod adds a zoom key. While the player holds Z, the view narrows to 30 degrees. When the key is let go, the player's normal view comes back.

```csharp
using ObenseuerCore;
using UnityEngine;

[ModInfo("example.zoom", "Zoom", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class ZoomMod : FrameworkMod
{
    private ModKeybind zoom;

    protected override void OnLoad()
    {
        zoom = Mod.Key("Zoom", "Zoom in", KeyCode.Z);
    }

    private void Update()
    {
        if (zoom == null) return;
        if (zoom.Down()) ScreenFx.SetFovOverride(Mod.Id, 30f);
        if (zoom.Up()) ScreenFx.ClearFovOverride(Mod.Id);
    }
}
```

- `Mod.Key` makes a key the player can see and change in the Mods menu.
- `Down()` is `true` on the frame the key is pressed, and `Up()` on the frame it is let go.
- `Update` runs every frame.
- The first argument of `SetFovOverride` is `Mod.Id`, your mod's id.

See [Keybinds](../guides/getting-started/keybinds.md) for more on keys.
