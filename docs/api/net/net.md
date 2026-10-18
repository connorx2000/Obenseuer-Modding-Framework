---
title: Net
parent: Multiplayer
grand_parent: Online play
nav_order: 3
---

# Net

`Net` sends short text messages to other players in the [lobby](../../glossary.md#lobby) (a group of connected players). Each message goes on a channel, a name you choose, so different mods and features do not mix. Your mod registers a handler (your code that runs when a message arrives) for a channel, and the framework runs it for each message there.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>Net</code> <span>short text messages in the lobby</span></p>

Call `Register` with your [mod id](../../glossary.md#mod-id) and a channel name to receive messages. Call `Send` to send one to a player in `Peers`.

<div class="api-pair" markdown="1">

```csharp
public static class Net
{
    public const int MaxPacketBytes = 1024;
    public const int MaxIdentifierLength = 32;
    public static ulong Me { get; }
    public static List<ulong> Peers { get; }
    public static long Dropped { get; }
    public static bool Send(ulong peer, string channel, string payload, bool reliable);
    public static bool Register(string modId, string channel, Action<ulong, string> handler);
}
```

| Method | Returns | What it does |
|---|---|---|
| `Send(peer, channel, payload, reliable)` | `bool` | Sends text to one player on a channel. With `reliable` set to `true`, Steam resends it until it arrives. `true` when the message was sent. |
| `Register(modId, channel, handler)` | `bool` | Sets the code to run for messages on this channel. `handler` gets the sender's [SteamID64](../../glossary.md#steamid64) (a Steam account number) and the text. `true` when registered. |

| Property | Type | What it means |
|---|---|---|
| `MaxPacketBytes` | `int` | The largest a message can be, in UTF-8 bytes. It counts the channel, a pipe character and the text. |
| `MaxIdentifierLength` | `int` | The longest a channel, game tag or version can be. |
| `Me` | `ulong` | The local player's SteamID64. The same as `Steam.LocalId`. |
| `Peers` | `List<ulong>` | The players you can send to, your [peers](../../glossary.md#peer): checked lobby members, not including you. |
| `Dropped` | `long` | How many messages were thrown away because a sender's queue was full, since the game started. |

</div>
</div>

## Things to know

- Your handler runs on the [main thread](../../glossary.md#main-thread), once the framework has read the message. It gets the sender's SteamID64 and the text, without the channel name. <span class="tech">(Handlers run during the `Update` of the framework's driver. Packets are read for up to 2 ms per frame, and at most 4 per sender are handed to handlers each frame.)</span>
- If your handler throws an error, the framework logs it with the handler's name. The other handlers still run.
- If a player sends faster than your handler keeps up, the messages wait in a queue and are handed out in order. Up to 1024 messages wait for each sender.
- If the same `modId` registers the same channel again, the new handler replaces the old one.
- The message is plain text. The framework never turns it into game objects. Send structured data as your own text format, and check it when it arrives.
- All mods share one Steam channel. The framework sorts the messages by channel name. <span class="tech">(It is Steam P2P channel 7.)</span>

## Limits

- `Send` returns `false` when it could not send: the player is not one of `Net.Peers`, the message is over `MaxPacketBytes`, or Steam refuses it.
- `Send` throws an error when `payload` is `null` or `channel` is not a valid name. The naming rules are on [Details](details.md#when-you-can-call-it).
- `Register` returns `false` when another mod already uses the channel. The framework then writes one warning that names both mods.
- The channels `presence` and `blob` belong to the framework. The framework's own id `obenseuer.moddingframework` can't register at all. `Register` returns `false` for both.
- When a sender's queue is full, the oldest 1024 messages are kept and the newest are dropped. `Net.Dropped` counts every message dropped this way. The [log](../../guides/getting-started/logging.md) gets a line at most once a minute for each sender. The queue is cleared when the framework leaves its lobby. <span class="tech">(A packet over the per-sender budget is handed out first on the next frame, and a newer packet from that sender waits behind it. The log line reads `net: dropped N packets from <SteamID64>`, where N is that sender's drops since its last line. A queue's packets are also dropped when the sender is no longer a peer.)</span>
- A message over `MaxPacketBytes` is dropped. So is a message on a channel that nobody registered. <span class="tech">(The 1024 limit is the framework's own limit. It counts the whole packet in UTF-8 bytes, so a long channel name leaves less room for text.)</span>
