---
title: Rules for taking an NPC
parent: NPC control
grand_parent: People
nav_order: 4
---

# Rules for taking an NPC

These rules cover when a hold on an [NPC](../npcs.md) (a character in the game) ends, what a release puts back, and which NPCs [`NpcControl.Take`](taking-and-releasing-an-npc.md) refuses.

## Things to know

- Your [claim](../../glossary.md#claim) (your mod's hold on an NPC) ends by itself when the NPC dies. The framework notices on its next call, so a later `Release` returns `false` and `IsTaken` is `false`. Your mod may call `Take` again with the same NPC id after the NPC moved to a new body in the game. Then the old claim is dropped and the new body is taken. That drop puts back the same things `Release` does, which for a dead NPC is the fade distance only.
- Things are put back only when the game has not changed them since.
  - The fade distance is put back only while it still holds the value the framework set.
  - `Movement` is the one flag that sets a value to hold the NPC, so releasing it writes the walking switch back. It does not turn movement on while the NPC is sitting in a chair or asleep. The game turns movement on again when the NPC stands up or wakes.
  - `Sight`, `Talk` and `Barks` change no value. The framework checks their flags whenever the game asks, so releasing them takes effect at once.
- `Release` and `ReleaseAll` always work. The game's save does not write anything the framework changes, so the framework does nothing special at save time.
- This API raises no events.

## Limits

- Only NPCs that are really in the [scene](../../glossary.md#scene) can be taken. `Take` needs a loaded NPC that is alive.
  - An NPC that is switched off only because the game hid it for being far away can be taken.
  - With the `Fade` flag, an NPC that is switched off only because the camera hid it can be taken too.
- An NPC switched off for any other reason is refused, and so is a camera-hidden NPC taken without `Fade`. An unknown, dead or off-scene NPC is refused.
- `Take` is refused while a save loads or is being written.
- A bad `modId` (your [mod id](../../glossary.md#mod-id)) throws an `ArgumentException`. So does a flags value that is not a mix of the [`NpcControlFlags`](taking-and-releasing-an-npc.md) members. A game failure is logged once and the call returns `false`. Nothing else throws. <span class="tech">(The `modId` is checked by the framework's mod id rules. `flags` outside `NpcControlFlags.All` are refused.)</span>

**Common problems**

- `Take` returns `false`: one of the refusals listed under Limits applies. A camera-hidden NPC taken without `Fade` is refused; take it with the `Fade` flag.
