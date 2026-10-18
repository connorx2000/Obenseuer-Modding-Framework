---
title: Mailboxes
parent: Phones, mail and media
grand_parent: API reference
has_children: true
nav_order: 5
---

# Mailboxes

[`Mailboxes`](mailboxes/reading-mailboxes.md) lets your mod read the mailboxes in the [scene](../glossary.md#scene) and react when the player uses one. Use it when your mod should do something around the player's mail.

## What it is

A mailbox is the box in the world where the player collects and posts mail. There are two kinds:

- A receive box hands over the letters that have arrived for the player.
- A send box takes the mail the player wants to post.

This API gives your mod these jobs:

- List the mailboxes in the [scene](../glossary.md#scene).
- Read the mail count each one shows.
- Take the player's waiting mail for them.
- Hear when the player uses a box.

A mailbox holds no mail of its own. Both kinds read from the player's own mail, not from the box. The player's inbox and the way mail is sent are on the [economy page](economy/mail.md#mail).

## When to use it

- Show a message or play a sound when the player uses a mailbox.
- Check whether the player has waiting mail, from the count on a receive box.
- Collect the player's waiting mail from a script, for example as part of an event.

## Quick example

This mod writes a [log](../guides/getting-started/logging.md) line each time the player uses a mailbox. When a save is loaded, it also logs how many mailboxes the scene has.

```csharp
using ObenseuerCore;

[ModInfo("example.mailwatch", "Mail Watch", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class MailWatch : FrameworkMod
{
    protected override void OnLoad()
    {
        Mailboxes.Used += args => Log.Info("Mailbox " + args.Id + " used in " + args.Mode + " mode.");
        Events.WhenReady(() => Log.Info(Mailboxes.All().Count + " mailboxes in this scene."));
    }
}
```

`Used` is an event: the framework calls your code each time it happens. `MailboxMode` is either `Receive` (the player takes mail) or `Send` (the player posts mail). [`Events.WhenReady`](core-helpers/events-whenready.md) waits until a save is loaded before it runs your code. See [Using events](../guides/getting-started/using-events.md).
