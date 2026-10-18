---
title: Camera
parent: Player
grand_parent: Player life and skills
nav_order: 9
---

# Camera

[`CameraView`](../../api-core/cameraview.md) changes how the camera sees the player. This page covers the player's body in first person, and third person with your mod's id. For the field of view and zoom, see [Screen effects](../screen-fx.md).

<div class="api-type" markdown="1">
<p class="api-type-head"><code>CameraView</code> <span>body and third-person claims</span></p>

Call `SetFirstPersonBody` to keep the player's body visible in first person, and `SetThirdPerson` to ask for third person.

<div class="api-pair" markdown="1">

```csharp
public static class CameraView
{
    public static bool SetFirstPersonBody(string owner);
    public static bool ClearFirstPersonBody(string owner);
    public static bool SetThirdPerson(string owner, float distance);
    public static bool SetFirstPerson(string owner);
}
```

| Method | Returns | What it does |
|---|---|---|
| `SetFirstPersonBody(owner)` | `bool` | Keeps the player's body visible in first person, for selfies. |
| `ClearFirstPersonBody(owner)` | `bool` | Removes your claim to keep the body visible. |
| `SetThirdPerson(owner, distance)` | `bool` | Asks for third person at this distance. |
| `SetFirstPerson(owner)` | `bool` | Gives up your request for third person. |

</div>
</div>

## Example

<div class="api-example" markdown="1">

```csharp
CameraView.SetFirstPersonBody("com.example.selfie");
CameraView.ClearFirstPersonBody("com.example.selfie");
```

</div>

## Things to know

- `SetThirdPerson(owner, distance)` asks for third person as your mod's claim. `owner` is your mod's id, the [owner](../../glossary.md#owner) of the claim. The call adds your mod to the list of mods that want third person. While any mod is on the list, the game's first-person lock stays cleared. It returns `false` when third person is not allowed (see `ThirdPersonAllowed`).
- `SetFirstPerson(owner)` gives up your request. When no mod is left, the game's lock goes back on, the camera goes to first person, and it returns `true`. While another mod still wants third person, the camera stays where it is and it returns `false`.
- The versions without an owner share one claim of their own. That covers `SetThirdPerson(distance)`, `SetFirstPerson()` and `Toggle()`. The framework's own F6 toggle ([`Tweaks.Camera`](../../api-core/tweaks.md)) uses a separate claim, and turning `ThirdPersonToggle` off removes only that one. A new scene load drops every claim and puts the game's lock back.
- `SetFirstPersonBody(owner)` keeps the player's body visible in first person, for selfies. Normally the game hides the body in first person. While any mod holds this, the body stays shown. If the camera is already in first person, the body shows at once. It returns `false` when this part could not be set up, and throws an `ArgumentException` for a bad owner. <span class="tech">(The game hides the body in `ThirdPersonCameraCollision.FirstPerson` through `HidePlayerModel`. A prefix on `HidePlayerModel` skips that hide while any owner is set.)</span>
- `ClearFirstPersonBody(owner)` removes your claim. When the last claim goes and the camera is in first person, the body is hidden again, as the game does. It returns `false` when your mod had no claim.
- Several mods can hold the body claim at once. The body stays visible until all of them have cleared it.
- A new scene load clears every claim, the same as for the field of view override and hiding the HUD. Set yours again from your own [`Events.SceneLoaded`](../../api-core/events.md) handler.
- Third person is unchanged, because the body is already shown there. The Mods menu's model preview also hides the body itself. While a claim is held, this hide is skipped there too. <span class="tech">(The Mods menu preview calls `HidePlayerModel`, which the prefix skips.)</span>

## Limits

- The body is the game's own model, placed at the camera. The head or neck may block the view, and the body may clip the camera.

**Common problems**

- Your camera claim no longer applies after a scene loads: a new scene load clears every claim. Set yours again from your own `Events.SceneLoaded` handler.
