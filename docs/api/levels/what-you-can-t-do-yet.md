---
title: What you can't do (yet)
parent: Levels
grand_parent: Places and travel
nav_order: 3
---

# What you can't do (yet)

The level-change actions that [`Levels`](level-changers.md) does not offer, and where to look instead when another API covers them.

## Limits

- No lock picking, keys or keypads here. Those are on the [Interaction and locks](../interaction.md) page (`Locks`).
- No way to make a door relock itself after use. The game does that inside its own level change. <span class="tech">(This is `LockAfterUse`.)</span>
- No control over the door's animation or sound timing.
- No way to find out when a level has finished loading. Use [`Events.SceneLoaded`](../../api-core/events.md) for that.
