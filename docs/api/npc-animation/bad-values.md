---
title: Bad values
parent: NPC animation
grand_parent: People
nav_order: 6
---

# Bad values

Which values make the [NPC animation](../npc-animation.md) calls throw an error, and what happens when the game refuses anything else.

- An invalid [mod id](../../glossary.md#mod-id) throws an `ArgumentException`. The mod id must be 1 to 64 characters of letters, digits, `.`, `_` or `-`.
- A `null` clip throws an `ArgumentNullException` named `clip`.
- A legacy clip throws an `ArgumentException` named `clip`. A legacy clip is one made for Unity's old animation system. <span class="tech">(It is a clip where `AnimationClip.legacy` is true.)</span>
- A fade that is below 0, not a number, or infinite throws an `ArgumentException` named `fadeSeconds`.
- Anything else the game refuses gives `false` and does not throw. The first failure of each call is written to the [log](../../guides/getting-started/logging.md) as an error.
