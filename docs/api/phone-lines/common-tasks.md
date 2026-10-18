---
title: Common tasks
parent: Phone lines
grand_parent: Phones, mail and media
nav_order: 1
---

# Common tasks

Short snippets for the usual jobs with [`PhoneLines`](phones-and-calls.md): list the phones, ring one, answer or decline, start a call, and check a phone's state. Later snippets reuse `phoneId` and `number` from the ring snippet.

List the phones in the scene, with the phone network each is on and whether it is busy.

```csharp
foreach (PhoneLineInfo line in PhoneLines.All())
    Log.Info(line.Id + " is on the " + line.Network + " network" + (line.Busy ? " and busy" : ""));
```

Make a phone ring. The number must be one the game knows, for example one from [`Phones.All()`](../phones.md). The last number you give is how many in-game seconds the phone rings before it gives up. Your code runs with `true` if the player picks up and `false` if nobody does.

```csharp
string phoneId = "some id from PhoneLines.All()";
string number = Phones.All()[0].Number;

bool rang = PhoneLines.Ring(phoneId, number, 60, answered =>
    Log.Info(answered ? "The player picked up." : "Nobody picked up."));
Log.Info(rang ? "The phone is ringing." : "The phone would not ring.");
```

Check whether a phone is ringing, and who is calling.

```csharp
if (PhoneLines.IsRinging(phoneId)) Log.Info("Ringing from " + PhoneLines.RingingNumber(phoneId));
```

Pick up a ringing phone for the player, or turn the call down.

```csharp
PhoneLines.Answer(phoneId);
PhoneLines.Decline(phoneId);
```

Start a call from a phone, as if the player had dialled the number.

```csharp
bool started = PhoneLines.StartCall(phoneId, number);
Log.Info(started ? "The call started." : "The call did not start.");
```

Check whether a phone is busy.

```csharp
Log.Info("Busy: " + PhoneLines.IsBusy(phoneId));
```

Check whether the phone events work. If `IsAvailable` is `false`, see [Details](details.md#if-something-fails-to-set-up).

```csharp
if (!PhoneLines.IsAvailable) Log.Warn("Phone line events are off.");
```
