---
title: Version check
parent: Multiplayer
grand_parent: Online play
nav_order: 5
---

# Version check

The framework checks that everyone in a [lobby](../../glossary.md#lobby) (a group of connected players) runs the same versions. That way a player on an old mod can't break the session. Each side sets the framework version, game version and Steam build id as information: the host on the lobby, and every member on itself.

## Things to know

- A connection request from a lobby member whose information has not arrived yet is held. It is accepted as soon as that member is checked. A request from anyone who is not in the lobby is ignored.

## Limits

- When you join, the join fails and [`Error`](lobby.md) is set if your versions do not match the host's. That is when the lobby's game tag is not yours, or its mod version or framework version differs from yours. It also fails when its game version differs from the game you run. And it fails when both sides know their Steam build and the two builds differ. <span class="tech">(The game version is `Application.version`, stored under `gv`. The Steam build id comes from the appmanifest, is stored under `gb`, and is empty when it can't be read.)</span> The messages are:
  - `Not a <gameTag> lobby.`
  - `Version mismatch: the lobby runs <hostVersion> (framework <hostCore>), you run <version> (framework <coreVersion>).`
  - `The host runs game version <hostVersion>, you run <gameVersion>.`
  - `The host runs game build <hostBuild>, you run <steamBuild>.`
- A member whose versions do not match is refused, and `Lobby.IsRefused` returns `true` for them. Their messages are thrown away and their connection requests are refused. They are missing from [`Net.Peers`](net.md), the list of your [peers](../../glossary.md#peer). They stay in `Members`, because Steam has no kick.
- The framework writes one line to the [log](../../guides/getting-started/logging.md) when the number changes: `<n> member(s) refused: mod version is not <version>, Core is not <coreVersion>, game version is not <gameVersion> or game build differs`.
- This is a compatibility check, not protection against a cheating player. The version information is set by the other side, so a changed game can claim any version.
