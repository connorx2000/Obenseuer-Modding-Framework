---
title: How a clip looks on the NPC
parent: NPC animation
grand_parent: People
nav_order: 4
---

# How a clip looks on the NPC

What decides how your clip looks on an NPC, and what ends a clip early. Read it when a clip does not show, moves the NPC, or stops by itself.

## Things to know

- Your clip has to match the NPC's skeleton. Use a humanoid clip on a humanoid rig, or a generic clip whose bone names exist on the NPC's model. A clip that does not match plays nothing on the bones it cannot find.
- The NPC's own animation keeps running under your clip. Your clip is mixed in over the NPC's current pose, by how strongly it is faded in. At half strength you see half of your clip and half of the NPC's pose, not a full replacement. <span class="tech">(The clip is a playable graph whose output weight is the fade. The graph has no copy of the NPC's controller, so the controller keeps running underneath.)</span>
- A clip with root motion can move the NPC. Root motion is movement that is recorded in the clip. It moves the NPC when the NPC's `Animator` has `applyRootMotion` on. Leave it off for a clip that should stay in place.
- Hiding, killing, or loading ends a clip. When the NPC's object is switched off or the NPC dies, its clip ends at once. It does not come back when the NPC returns, so play it again when you need it. A [scene](../../glossary.md#scene) (one loaded area of the game) load that replaces the whole scene ends every clip too. <span class="tech">(The graph is destroyed. This is a `Single` scene load; additive loads do not end clips.)</span>
- Clips are not saved. Play them again after a load, for example from [`Events.WhenReady`](../core-helpers/events-whenready.md). A `Play` called directly from an [`Events.SceneLoaded`](../../api-core/events.md) handler gives `false`, because the game still counts as loading during that whole event. <span class="tech">(`SaveGate.Busy` is set for the whole `SceneLoaded` event.)</span>

**Common problems**

- The clip plays nothing on some bones: the clip does not match the NPC's skeleton. Use a humanoid clip on a humanoid rig, or a generic clip whose bone names exist on the NPC's model.
- The clip moves the NPC: the clip has root motion, and the NPC's `Animator` has `applyRootMotion` on. Leave it off for a clip that should stay in place.
- The clip has ended by itself: the NPC's object was switched off, the NPC died, or a load ended it. Clips are not saved, and a scene load that replaces the whole scene ends every clip. Play the clip again when you need it.
- `Play` gives `false` from an `Events.SceneLoaded` handler: the game still counts as loading during that whole event. Play the clip from `Events.WhenReady` instead.
