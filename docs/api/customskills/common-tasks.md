---
title: Common tasks
parent: Custom skills
grand_parent: Player life and skills
nav_order: 1
---

# Common tasks

Short snippets for the usual jobs: read a skill, change it, show a banner and hide your skills. The calls are listed in [Skills](skills.md), and `Mod.Id` is your [mod id](../../glossary.md#mod-id).

Read a skill's value and level. The value can have a fraction, such as `3.4`. The level is that value rounded down, so `3`.

```csharp
float value = CustomSkills.Get(Mod.Id, "baking");
int level = CustomSkills.Level(Mod.Id, "baking");
Log.Info("Baking: " + value + " (level " + level + ")");
```

Set a skill to an exact value, or take some away.

```csharp
CustomSkills.Set(Mod.Id, "baking", 5f);
CustomSkills.Add(Mod.Id, "baking", -1f);
```

Show a message in the skill banner. The rules are in [Banners](banners.md).

```csharp
if (!CustomSkills.QueueBanner("The oven is warm."))
    Log.Warn("The banner was not queued.");
```

Hide your skills for the rest of this session. The saved values stay.

```csharp
CustomSkills.Clear(Mod.Id);
```
