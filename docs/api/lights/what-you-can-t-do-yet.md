---
title: What you can't do (yet)
parent: Lights
grand_parent: Objects and devices
nav_order: 4
---

# What you can't do (yet)

The light jobs that [`Lights`](simple-lights.md) does not offer, and what it can change instead.

## Limits

- No way to switch a [multi-state](multi-state-lights.md) fixture on or off as a whole. Only the lit setting can be changed. <span class="tech">(The game's `LightTurnOffOnMultiState.on` flag and `Toggle` method have no framework call.)</span>
- No way to address the `LightsON` script. It has no save id and no saved state. So no id can point at it, and a change would be lost the next time the scene starts.
- No way to wait for a fade to finish. The fades run over several frames. `Toggled` fires when the switch flips, not when the light has finished fading. <span class="tech">(`TurnLightOnFade` and `TurnLightOffFade`.)</span>
- No control over a light's brightness or materials. Only on or off, and the lit setting, can be changed.
