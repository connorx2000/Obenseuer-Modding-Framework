---
title: Common tasks
parent: Dialogue authoring
grand_parent: Story
nav_order: 1
---

# Common tasks

Short snippets for the jobs most dialogue mods need: start a conversation, remove one, add an answer to a game conversation, and show narration.

This page uses `Transform` from Unity. The [glossary](../../glossary.md#types-that-come-from-unity) says what each one is.
{: .unity-types}

Start a conversation you added with [`Dialogue.Start`](../../api-story/dialogue.md#starting-and-stopping), with the player and the character as the two sides. Call this from your own code, for example when the player presses a key near the character.

```csharp
void StartTalk(string title, Transform player, Transform npc)
{
    if (!Dialogue.Start(title, player, npc)) Log.Warn("The conversation did not start.");
}
```

Remove one conversation you added with [`DialogueAuthoring.Remove`](writing-a-conversation.md). `Clear` removes everything else your mod added to the dialogue system.

```csharp
DialogueAuthoring.Remove(Mod.Id, "greeting");
int removed = DialogueAuthoring.Clear(Mod.Id);
```

Add a new answer to a game character's conversation with [`Dialogue.AddResponse`](responses-to-a-game-conversation.md). The title and the line text must match what the game has.

```csharp
string key = Dialogue.AddResponse(Mod.Id, "Shop.Greeting", "Need anything today?", "Do you have any rope?", "Second shelf, by the door.", "not mymod_asked", () => Dialogue.SetVariable("mymod_asked", true));
if (key == null) Log.Warn("The answer was refused.");
```

Show narration on the black screen with [`Dialogue.Narrate`](narration.md).

```csharp
if (!Dialogue.Narrate(Mod.Id, "Three days later...")) Log.Warn("Narration could not start now.");
```
