---
title: Pay the player
parent: Cookbook
nav_order: 2
---

# Pay the player

Pay the player 50 [OC](../../glossary.md#oc) (Open Sewer Coins, the game's main currency) once per game day. Use this when your mod rewards the player over time. The paid day is saved with the game, so loading a save does not pay twice.

## The example

`PayState` holds the number of the last day paid. [`SaveData.Bind`](../../api/savedata-game/bound-slots-savedata-bind.md) keeps it with each save. Every game hour the mod checks whether today was paid yet. If not, it calls [`Cash.Give`](../../api/economy/money-cash.md).

Start from the project you made in [Your first mod](../getting-started/first-mod.md), and replace the code in `Plugin.cs` with the code below.

```csharp
using System;
using ObenseuerCore;

[Serializable]
public class PayState
{
    // The game day that was paid last, or -1 if never.
    public int lastPaidDay = -1;
}

[ModInfo("example.payplayer", "Pay Player", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public class PayPlayer : FrameworkMod
{
    // How much to pay each day.
    private const int DailyOc = 50;
    // The saved data. Its Value is never null.
    private SaveSlot<PayState> slot;

    protected override void OnLoad()
    {
        // Keep PayState with each save, under the key "pay".
        slot = SaveData.Bind<PayState>("example.payplayer", "pay");
        // Check once every game hour.
        Events.HourPassed += OnHour;
    }

    private void OnHour(int hour)
    {
        // Skip while a save or load runs, or when today was paid already.
        if (SaveData.Busy || slot.Value.lastPaidDay == GameTime.Day) return;
        // Pay, and remember today only if the payment worked.
        if (Cash.Give(MoneyCurrency.OC, DailyOc)) slot.Value.lastPaidDay = GameTime.Day;
    }
}
```

Build it and check the payment:

1. Close the game and run `dotnet build -c Release -p:DeployToGame=true` in your mod folder.
2. Start the game and load a save.
3. Wait until the next full game hour. You receive 50 OC, and the next hour does not pay again.

The daily payment works. Things to know, below, explains the saved slot and the hourly check.

### Common problems

- No OC arrives at the next full hour: the code skips the hour while a save or load runs, and when today was paid already. A day that is missed during a save is paid in a later hour. A day counts as paid only if `Cash.Give` worked.
- Nothing is paid on any day: check that the mod loaded, see [my mod does not load](../../faq.md#my-mod-does-not-load).

## Things to know

- [`SaveData.Bind`](../../api/savedata-game/bound-slots-savedata-bind.md) returns a slot whose `Value` is never `null`, so you do not need a null fallback.
- The check runs every hour, not once a day.
- [`Cash.Give`](../../api/economy/money-cash.md) takes whole OC amounts. For money in a bank account, use `Banking.Refund` instead. See [Gameplay and economy](../gameplay.md#money).
- Save data is covered in the [Save data](../getting-started/save-data.md) guide.

## Where to go next

Next: [React to a new day](react-to-a-new-day.md).
