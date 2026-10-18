---
title: Common tasks
parent: Mailboxes
grand_parent: Phones, mail and media
nav_order: 1
---

# Common tasks

Short snippets for the usual jobs with [`Mailboxes`](reading-mailboxes.md): log the count each box shows, take the player's waiting mail, and look up one box by its id.

Log the mail count each box shows.

```csharp
foreach (MailboxInfo box in Mailboxes.All())
    Log.Info(box.Id + " (" + box.Mode + "): " + box.Count);
```

Take the player's waiting mail. This works on a receive box and gives the player everything in their inbox. `TakeMail` returns `true` when the inbox is empty afterwards.

```csharp
foreach (MailboxInfo box in Mailboxes.All())
{
    if (box.Mode != MailboxMode.Receive) continue;
    if (Mailboxes.TakeMail(box.Id)) Log.Info("The inbox is empty.");
    break;
}
```

Look up one mailbox by its id. `Get` gives back `null` when no mailbox has that id.

```csharp
MailboxInfo found = Mailboxes.Get("some-mailbox-id");
if (found != null) Log.Info("Count: " + found.Count);
```
