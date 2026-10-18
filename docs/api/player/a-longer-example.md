---
title: A longer example
parent: Player
grand_parent: Player life and skills
nav_order: 10
---

# A longer example

This page has a complete mod that checks several Player calls with one key press. When you press F9, it reads the player's position, health and a skill, and adds half a skill level. It also counts the active effects and the level's [spawn points](../../glossary.md#entrypoint) (named entrypoints).

This page uses `KeyCode` from Unity. The [glossary](../../glossary.md#types-that-come-from-unity) says what each one is.
{: .unity-types}

```csharp
using ObenseuerCore;
using UnityEngine;

[ModInfo("example.playersmoke", "Player Smoke", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public class PlayerSmoke : FrameworkMod
{
    private ModKeybind smokeKey;

    protected override void OnLoad()
    {
        smokeKey = Mod.Key("Smoke", "Run the player check", KeyCode.F9);
    }

    private void Update()
    {
        if (smokeKey == null || !smokeKey.Down()) return;
        Log.Info("pos " + PlayerState.Position + " alive " + PlayerState.IsAlive + " health " + PlayerState.Health);
        Log.Info("lockpicking " + Skills.Get(Skill.Lockpicking) + " level " + Skills.Level(Skill.Lockpicking));
        Log.Info("add lockpicking " + Skills.Add(Skill.Lockpicking, 0.5f));
        Log.Info("active effects " + Effects.Active().Count + ", entrypoints " + Travel.Entrypoints().Count);
    }
}
```

To run it:

1. Load a save.
2. Press F9.
3. Open the mod's [log](../../guides/getting-started/logging.md), `BepInEx\logs\example.playersmoke.log`.

The check has run. Next, compare the log with the list below.

What you should see:

- The position is not zero.
- `Add` returns `true`.
- `Level` goes up once `Get` crosses a whole number. To see gains from the game too, subscribe to [`Skills.SkillIncreased`](skills.md).
- The spawn point count matches the level's spawn points.

If something is wrong, look for errors in `BepInEx\logs\obenseuer.moddingframework.player.log`, `obenseuer.moddingframework.skills.log` and `obenseuer.moddingframework.effects.log`.
