---
title: Common tasks
parent: Multiplayer
grand_parent: Online play
nav_order: 1
---

# Common tasks

Short snippets for the usual multiplayer jobs: share the player's position, send larger data to one player, and find friends to invite.

This page uses `Vector3` from Unity. The [glossary](../../glossary.md#types-that-come-from-unity) says what each one is.
{: .unity-types}

Share the local player's position every frame, and count the other players within five metres. [`Publish`](presence.md) keeps sending for as long as you keep calling it, so call it from `Update`.

```csharp
private void Update()
{
    Presence.Publish("Bar", PlayerState.Position, 0f, false);
    int nearby = 0;
    foreach (PresenceRecord remote in Presence.Remotes)
        if (Vector3.Distance(remote.Position, PlayerState.Position) < 5f) nearby++;
}
```

Send a larger piece of data to one player with [`NetBlob`](blob.md), and receive it on the other side. `Send` returns -1 when it could not start. Check who sent the data before you trust it. The code below checks that the sender is in `Net.Peers`, the list of your [peers](../../glossary.md#peer) (the other players you can send to).

```csharp
NetBlob.Received += (peer, modId, name, data) =>
{
    if (modId == Mod.Id && name == "note" && Net.Peers.Contains(peer))
        Log.Info("Got a note: " + Encoding.UTF8.GetString(data));
};
NetBlob.Finished += (id, delivered) => Log.Info("Transfer " + id + (delivered ? " arrived" : " failed"));

int id = NetBlob.Send(peer, Mod.Id, "note", Encoding.UTF8.GetBytes("Meet me at the bar."));
if (id < 0) Log.Info("The note could not be sent.");
```

Find friends who are playing Obenseuer with [`Steam`](steam.md), open the invite window, and leave the [lobby](lobby.md) when your mod is done. `951240` is the number to use for Obenseuer.

```csharp
foreach (ulong friend in Steam.FriendsPlaying(951240))
    Log.Info(Steam.Name(friend) + " is playing Obenseuer");

if (Lobby.IsHost) Lobby.Invite();
Lobby.Leave("pingpong");
```
