---
title: Responses to a game conversation
parent: Dialogue authoring
grand_parent: Story
nav_order: 3
---

# Responses to a game conversation

`Dialogue.AddResponse` adds one player choice to a conversation the game already has, for example a second reply to a character's line. Nothing is copied: the choice is written into the game's own conversation for this session. This part is experimental.

`Dialogue.RemoveResponse` or `Dialogue.RemoveAll` takes the choice out again.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>Dialogue</code> <span>adds and removes responses in game conversations</span></p>

Call `AddResponse` with the anchor and the player's text. Keep the key it returns, so that `RemoveResponse` can take the choice out later.

<div class="api-pair" markdown="1">

```csharp
public static class Dialogue
{
    public static string AddResponse(string modId, string conversationTitle, string anchorText, string playerText, string npcReply, string luaCondition, Action onPicked);
    public static string AddResponse(string modId, string conversationTitle, int anchorId, string playerText, string npcReply, string luaCondition, Action onPicked);
    public static bool RemoveResponse(string modId, string key);
}
```

| Method | Returns | What it does |
|---|---|---|
| `AddResponse(modId, conversationTitle, anchorText, playerText, npcReply, luaCondition, onPicked)` | `string` | Adds one player choice under the anchor line, found by its text. Returns the response key. |
| `AddResponse(modId, conversationTitle, anchorId, playerText, npcReply, luaCondition, onPicked)` | `string` | Adds one player choice under the anchor line, found by its id. Returns the response key. |
| `RemoveResponse(modId, key)` | `bool` | Removes one of your responses. `false` for another mod's key or an unknown key. |

</div>
</div>

## Things to know

- Both overloads add one choice under the anchor line. You give the anchor by its text or by its id. The anchor is the character's line that your new choice hangs under.
- `anchorText` is matched against the line's original written text, not the translated text the player sees. The framework tries an exact match first. Then it tries a match that ignores spaces around the text and capital letters. Use the `anchorId` overload when the text is not unique.
- An anchor with no links at all is accepted.
- Your choice sits with the anchor's other choices, at the same priority. If `npcReply` is not `null`, the character answers with `npcReply` and the conversation ends after it. If it is `null`, the conversation ends after your player line.
- `luaCondition` is [Lua](../../glossary.md#lua) (the game's dialogue scripting language), up to 500 characters, or `null`. The choice is offered only while it is true.
- The text of a choice can change from one visit to the next. Put `[var=Name]` in the text and set `Name` with [`Dialogue.SetVariable`](../../api-story/dialogue.md#variables).
- `onPicked` runs when the player picks your choice. It also runs when yours is the only choice and the game picks it for the player. If it throws an error, the framework logs it once and the game carries on.
- `AddResponse` returns a key: your [mod id](../../glossary.md#mod-id), a dot and eight characters. The key comes from the conversation, the anchor and the texts, so the same arguments give the same key. Calling `AddResponse` again with the same arguments returns that key and replaces `onPicked`.
- The framework puts your responses back by itself. The game can make new copies of its conversations when a [scene](../../glossary.md#scene) (one loaded area of the game) loads. So the framework adds the responses again on every scene load and after each load of a save. Your mod does not add them again.
- `RemoveResponse` returns `false` for another mod's key, an unknown key, or while a save is loading or being written. `Dialogue.RemoveAll(modId)` also removes your [interceptors](../../glossary.md#interceptor) (handlers that run before the game acts) and Lua functions at any time. While a save is loading or being written, it keeps your responses, as `RemoveResponse` refuses then.
- Responses are never saved. A save made while a response is in place loads fine without your mod. The "already shown" marks for the response lines are ignored.

## Limits

- If nothing matches, or more than one line matches, the call is refused.
- The anchor has to be a character's line that leads to choices. A player line cannot be the anchor. An anchor whose links lead only to character lines is refused, because the game then plays that line and never shows the choices.
- The anchor may lead to both character lines and player lines. Then your choice only shows when none of those character lines can play. The game plays a character line whose condition is true instead of showing the choices, as it does for its own lines.
- `AddResponse` returns `null` when it was refused. That happens in these cases:
  - The dialogue system is not installed or not ready (no game scene yet).
  - A save is loading or being written.
  - A text is empty or too long.
  - The conversation is not in the game.
  - The anchor is missing, matches more than one line, is not a character's line, or leads only to character lines.
  - The game has no player character.
- An empty `conversationTitle` or `playerText` throws an `ArgumentException`.

**Common problems**

- `AddResponse` returns `null` because the anchor text matches more than one line: use the `anchorId` overload.
