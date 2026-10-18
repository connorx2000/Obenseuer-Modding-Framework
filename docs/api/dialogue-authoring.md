---
title: Dialogue authoring
parent: Story
grand_parent: API reference
has_children: true
nav_order: 1
---

# Dialogue authoring

[`DialogueAuthoring`](dialogue-authoring/writing-a-conversation.md) lets your mod write its own conversations, add choices to the game's conversations and show narration on a black screen. Use it when a character needs something new to say.

Status: experimental.

## What it is

A conversation is a set of lines that a character speaks, and choices the player picks. This API does these jobs while the game runs:

- Write your own conversations and start them like any conversation from the game.
- Add a new answer to a conversation the game already has.
- Hang a whole menu of your own lines under a line of a game character.
- Change a line's text each time it shows.
- Show a plain block of narration text on a black screen.

An NPC line is spoken by the character. A player line is a choice the player picks. The framework does not write voice, camera cuts or other effects, so a conversation shows with the game's normal look. The API builds on the `Dialogue` part of the [Story API](../api-story/dialogue.md#dialogue), which starts and stops conversations.

## When to use it

- Give a character from your own mod something to say.
- Add a new player answer to a game character's conversation, for example a new shop question.
- Add a menu (such as "Membership") under a line of an existing character, with its own replies.
- Show a short piece of narration on a black screen, such as "Three days later".

## Quick example

This mod adds a small conversation as soon as the game is ready. It then [logs](../guides/getting-started/logging.md) the title the game knows the conversation by. You start the conversation with `Dialogue.Start`, as the next section shows.

```csharp
using ObenseuerCore;

[ModInfo("example.greeter", "Greeter", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class Greeter : FrameworkMod
{
    protected override void OnLoad()
    {
        Events.WhenReady(() =>
        {
            ConversationDefinition talk = new ConversationDefinition { Key = "greeting" };
            talk.Lines.Add(new ConversationLine { Key = "hello", Text = "Back again?", Next = new[] { "ask", "leave" } });
            talk.Lines.Add(new ConversationLine { Key = "ask", Player = true, Text = "Any work for me?", Next = new[] { "work" } });
            talk.Lines.Add(new ConversationLine { Key = "leave", Player = true, Text = "Not now." });
            talk.Lines.Add(new ConversationLine { Key = "work", Text = "Check the cellar.", Script = "Variable[\"mymod_cellar\"] = true" });

            ConversationRefusal refusal;
            string title = DialogueAuthoring.Add(Mod.Id, talk, out refusal);
            Log.Info(title == null ? "The conversation was refused: " + refusal : "Added " + title);
        });
    }
}
```

The first line, "Back again?", is spoken by the character. It offers two choices: "Any work for me?" and "Not now.". A line with no `Next` ends the conversation. The `Script` on the last line sets a conversation variable when the player reaches it. `Mod.Id` is the [mod id](../glossary.md#mod-id) you gave in `[ModInfo]`. [`Events.WhenReady`](core-helpers/events-whenready.md) runs your code once a save has loaded. See [Using events](../guides/getting-started/using-events.md) for more on `WhenReady`.

- `Add` returns the title, or `null` with the reason in `refusal`. `NoPlayerActor` means the game's database has no actor marked as the player. `Failed` means a game call threw or the database did not take the conversation. `ReservedKey` means the key is `narration`, which [`Dialogue.Narrate`](dialogue-authoring/narration.md) uses.
- `Remove` removes one of your conversations. It returns `false` for another mod's conversation, an unknown one, one that is running now, or a branch's conversation (use `RemoveBranch` for those).
- `Clear` removes all of your conversations that are not running, and the branches your mod added with `Dialogue.AddBranch`. It returns how many it removed.

## Example

This adds the same conversation under a fixed mod id, and starts it with [`Dialogue.Start`](../api-story/dialogue.md#starting-and-stopping) when `Add` returns a title.

```csharp
ConversationDefinition talk = new ConversationDefinition { Key = "greeting" };
talk.Lines.Add(new ConversationLine { Key = "hello", Text = "Back again?", Next = new[] { "ask", "leave" } });
talk.Lines.Add(new ConversationLine { Key = "ask", Player = true, Text = "Any work for me?", Next = new[] { "work" } });
talk.Lines.Add(new ConversationLine { Key = "leave", Player = true, Text = "Not now." });
talk.Lines.Add(new ConversationLine { Key = "work", Text = "Check the cellar.", Script = "Variable[\"mymod_cellar\"] = true" });

ConversationRefusal refusal;
string title = DialogueAuthoring.Add("example.mymod", talk, out refusal);
if (title != null) Dialogue.Start(title, playerTransform, npcTransform);
```

`playerTransform` and `npcTransform` are the [Transform](../glossary.md#transform) (where an object is in the world) of the player and of the character.

## Responses to a game conversation

[`Dialogue.AddResponse`](dialogue-authoring/responses-to-a-game-conversation.md) adds one player choice to a conversation the game already has, for example a second reply to an NPC line. Nothing is copied. The choice is written into the game's own conversation for this session. `Dialogue.RemoveResponse` or `Dialogue.RemoveAll` takes it out again. Experimental.

```csharp
public static class Dialogue
{
    public static string AddResponse(string modId, string conversationTitle, string anchorText, string playerText, string npcReply, string luaCondition, Action onPicked, int insertAt = -1, int priority = -1);
    public static string AddResponse(string modId, string conversationTitle, int anchorId, string playerText, string npcReply, string luaCondition, Action onPicked, int insertAt = -1, int priority = -1);
    public static string AddResponse(string modId, string conversationTitle, string anchorText, string playerText, string npcReply, string luaCondition, Action onPicked);
    public static string AddResponse(string modId, string conversationTitle, int anchorId, string playerText, string npcReply, string luaCondition, Action onPicked);
    public static bool RemoveResponse(string modId, string key);
}
```

- **Anchor.** The anchor is an NPC line of the conversation. `anchorText` is matched against the line's authored Dialogue Text, not the translated text the player sees. An exact match is tried first, then a match that ignores surrounding spaces and letter case. Zero matches, or more than one, is refused, so use the `anchorId` overload when the text is not unique. A player line cannot be the anchor. An anchor whose links lead only to NPC lines is refused, because the game then shows that NPC line and never the choice. An anchor with no links is accepted. If the anchor links to both NPC and player lines, the game plays an NPC line whose condition is true instead of showing the choices. It does the same for its own lines. So the new choice only shows when none of those NPC lines can play.
- **Shape.** The player line is offered with the anchor's other choices, at the same priority as they have, unless `priority` sets another. If `npcReply` is not null, the NPC answers the choice with `npcReply`, and the conversation ends after `npcReply`. If it is null, the conversation ends after the player line.
- **Position.** `insertAt` is the index in the anchor's outgoing links where the choice is inserted, counting from 0, so `0` puts it first. The default `-1` appends. That places the choice after links the game appends itself, such as the leave option. An index past the end appends too. The position is kept when the framework writes the choice again after a [scene](../glossary.md#scene) (one loaded area of the game) loads.
- **Priority.** `priority` is the game's link priority: `0` Low, `1` BelowNormal, `2` Normal, `3` AboveNormal, `4` High. The default `-1` copies the priority of the anchor's own player links, and `Normal` when it has none. The game considers links one priority tier at a time, from High down. So a higher priority offers the choice before the anchor's own choices, and a lower one after them. A value outside -1 to 4 throws `ArgumentException`.
- **Condition.** `luaCondition` is [Lua](../glossary.md#lua) (the game's dialogue scripting language), up to 500 characters, or null. The choice is offered only while it is true, as for a line's `Condition`.
- **Text.** A reply's text can change from one visit to the next through Dialogue System variable markup. Put `[var=Name]` in the text, and set `Name` with [`Dialogue.SetVariable`](../api-story/dialogue.md#variables).
- **onPicked.** Runs when the player picks the choice. It also runs when the choice is the only one and the game picks it for the player. An exception is logged once and does not reach the game.
- **Key.** `AddResponse` returns the key: `modId`, a dot and eight hex digits. The key comes from the conversation, the anchor and the texts, so the same arguments give the same key. Calling `AddResponse` again with the same arguments returns that key and replaces `onPicked`.
- **Re-apply.** The game can make new copies of its conversations on a scene load. So the framework adds the responses back on every scene load and after each load of a save. Your mod does not re-add them. The conversation or the anchor may not be in the database at that moment. Then the framework keeps trying once per frame for a bounded number of frames, and logs one failure per key when it gives up. So a response whose conversation loads late is applied when it appears. A later scene load or save load starts the attempts again.
- **Refusals.** `AddResponse` returns `null` in these cases:
  - The dialogue system is not installed or not ready (no game scene yet).
  - A save or load is running.
  - A text is empty or too long.
  - The conversation is not in the database.
  - The anchor is missing, ambiguous, not an NPC line, or links only to NPC lines.
  - The database has no player actor.

  An empty `conversationTitle` or `playerText` throws `ArgumentException`, and so do an `insertAt` below -1 and a `priority` outside -1 to 4. The original seven-argument form without `insertAt` and `priority` is kept as a thin wrapper, so a mod compiled against 0.11.0-beta still binds it. It is the same as passing -1 for both.
- **Remove.** `RemoveResponse` returns `false` for another mod's key, an unknown key, or while a save or load is running. `Dialogue.RemoveAll(modId)` also removes the mod's [interceptors](../glossary.md#interceptor) (handlers that run before the game acts) and Lua functions, whatever the save state. It keeps the mod's responses while a save or load is running, as `RemoveResponse` refuses then.
- **Saves.** The framework never saves responses. A save made while a response is in place loads without the mod: the read-state rows of the response lines are ignored.

This adds a rope question under the shop line "Need anything today?".

```csharp
string key = Dialogue.AddResponse("example.mymod", "Shop.Greeting", "Need anything today?", "Do you have any rope?", "Second shelf, by the door.", "not mymod_asked", () => Dialogue.SetVariable("mymod_asked", true));
if (key == null) return;
```

## Branches under a game NPC line

[`Dialogue.AddBranch`](dialogue-authoring/branches-under-a-game-npc-line.md) links a conversation of your own lines into a game conversation, for a menu tree. The branch is an authored conversation, so the rules above for ids, validation and re-adding apply to it. Experimental.

```csharp
public static class Dialogue
{
    public static string AddBranch(string modId, string conversationTitle, string anchorText, ConversationDefinition branch, int insertAt = -1, int priority = -1);
    public static string AddBranch(string modId, string conversationTitle, int anchorId, ConversationDefinition branch, int insertAt = -1, int priority = -1);
    public static string AddBranch(string modId, string conversationTitle, string anchorText, ConversationDefinition branch);
    public static string AddBranch(string modId, string conversationTitle, int anchorId, ConversationDefinition branch);
    public static bool RemoveBranch(string modId, string key);
}
```

- **Anchor.** The anchor is an NPC line of the game conversation, chosen as for `AddResponse`, and the same refusals apply. For a line with a `TextCallback`, the authored text is the callback's markup, so use the `anchorId` overload for that line.
- **Link.** The branch starts at its first line, the first entry of `Lines`. The anchor gets one link to that line. The link has the priority of the anchor's other player choices, unless `priority` sets another. It sits at the index `insertAt` in the anchor's links (`-1` appends, as it does for `AddResponse`). The first line must be a player line. `AddBranch` refuses a branch whose first line is an NPC line, because the player would get no choice.
- **Speaker.** The branch's NPC lines are spoken by the anchor's NPC, so its name and portrait show. `branch.Speaker` may be null or the name of that same actor. Any other name is refused.
- **Key.** `AddBranch` returns the key: `modId`, a dot and eight hex digits. It comes from the conversation title, the anchor and `branch.Key`. Calling `AddBranch` again with the same arguments is refused, so call `RemoveBranch` first. The branch's conversation title is the mod id, a dot and `branch.Key`. So a key already in the database is refused as `AlreadyExists`, also when it is used on another anchor.
- **Re-apply.** The framework writes the anchor link again after each scene load and each load of a save, as it does for responses. It retries the same bounded way when the conversation, the anchor or the branch conversation is not in the database yet. Your mod does not re-add it.
- **Remove.** `RemoveBranch` removes the link and the branch's conversation. It returns `false` for another mod's key, an unknown key, a save or load running, or the dialogue database not ready. It also returns `false` while any conversation is running, because the player can reach a branch through another conversation's link. `Dialogue.RemoveAll(modId)` removes the mod's branches too. It keeps any it cannot remove now, while a save, a load or a conversation is running. `DialogueAuthoring.Clear(modId)` does the same. A branch conversation that is still in place cannot be removed with `DialogueAuthoring.Remove`.
- **Refusals.** `AddBranch` returns `null` and logs the reason once in these cases:
  - The dialogue system is not ready.
  - A save or load is running.
  - The branch is not valid.
  - The branch key is `narration`.
  - The first line is not a player line.
  - The conversation or anchor is not found.
  - The branch speaker is not the anchor's NPC.

  An empty `conversationTitle` throws `ArgumentException`. A null `branch` throws `ArgumentNullException`. An invalid `modId` throws `ArgumentException`, and so do an `insertAt` below -1 and a `priority` outside -1 to 4. The original four-argument form without `insertAt` and `priority` is kept as a thin wrapper as well.
- **Saves.** Like responses, the framework never saves branches. A save made while a branch is in place loads without the mod.

This adds a "Membership" menu under the shop line "What can I do for you?".

```csharp
ConversationDefinition membership = new ConversationDefinition { Key = "membership" };
membership.Lines.Add(new ConversationLine { Key = "ask", Player = true, Text = "Membership", Next = new[] { "status" } });
membership.Lines.Add(new ConversationLine
{
    Key = "status",
    Text = "Your card is in order.",
    TextCallback = () => "You have " + MembershipDaysLeft() + " days left.",
    Next = new[] { "renew", "back" }
});
membership.Lines.Add(new ConversationLine { Key = "renew", Player = true, Text = "Renew for 30 days.", Script = "Variable[\"mymod_renew\"] = true" });
membership.Lines.Add(new ConversationLine { Key = "back", Player = true, Text = "Goodbye." });

string key = Dialogue.AddBranch("example.mymod", "Shop.Greeting", "What can I do for you?", membership);
```

The tree starts at the anchor "What can I do for you?". Then come the player choice "Membership" and the NPC reply with the days left. The two choices "Renew for 30 days." and "Goodbye." follow.

A line without `Next` ends the conversation. A branch cannot return to the game conversation, so "Goodbye." ends the talk and the game's menu is not shown again. Put any choice that should go back to the game's menu in the game conversation itself, not in a branch.

## Text callbacks

[`ConversationLine.TextCallback`](dialogue-authoring/text-callbacks.md) sets a line's text from a function. The game calls it when the line shows, so the text can change from one visit to the next. It works for a conversation from `DialogueAuthoring.Add` and for a branch.

- The callback returns the text. When it returns null, the line shows `Text` instead. When it throws, the line shows nothing and the exception is logged once.
- A returned text over 1000 characters, or with a control character, is replaced by `Text` and logged once.
- When `TextCallback` is set, `Text` may be null. Without a callback, `Text` is required as before.
- The callback can run several times for one showing of a line. While the game works out the links, it parses each NPC reply it considers. It parses a player choice once for the menu and again for the subtitle. The callback must have no side effects.
- The callback runs on the game's [main thread](../glossary.md#main-thread) (the one thread that runs the game) while the conversation is open. Keep it short. Do not start a conversation or write to the save from it.
- The returned text is parsed as Dialogue System markup, as authored text is. `[var=Name]`, emphasis tags such as `[em1]`, and `|` as a line break all work. The `|` line break is on by default in the game's subtitle settings.
- `Remove` drops the callbacks of that conversation, and `RemoveBranch` drops those of its branch. `DialogueAuthoring.Clear` drops those of each conversation it removes. `Dialogue.RemoveAll` drops those of the branches it removes. `Dialogue.RemoveAll` does not remove conversations made with `DialogueAuthoring.Add`, so use `Clear` for those.

## Narration

[`Dialogue.Narrate(modId, text)`](dialogue-authoring/narration.md) (experimental) shows `text` in the game's dialogue box on the game's black screen. It is built on this API, with one conversation per mod. The key is `narration`, so the title is `modId.narration`. Each call replaces that conversation's text, then starts it through the game's `BlackCanvas.StartDialogue`.

```csharp
public static class Dialogue
{
    public static bool Narrate(string modId, string text);
}
```

- It returns `true` when a conversation is running after the call. That can be a different conversation when a handler from `Dialogue.Intercept` redirects the start.
- It returns `false` in these cases:
  - The text is null, empty, only spaces, or over 1000 characters.
  - The text has a control character other than a line break (`\n` or `\r`).
  - `Dialogue.IsAvailable` is false, or the dialogue system is not ready.
  - A save or load is running, or a conversation is already running.
  - `BlackCanvas.instance` is null, or the player is waiting or asleep.
  - `DialogueAuthoring.Add` refuses the conversation, for example when the database has no player actor.
- A call from a wake-up callback is refused while the sleep event's text (`WaitingController.eventDialogue`) is still waiting to show. It is refused for the same reason: starting a conversation wakes the player and cuts that text off.
- The screen starts to fade to black at once, before the conversation text shows.
- When the conversation ends, the framework fades the screen back out. It uses the game's end-of-talk callback on the black screen's talk component. The framework keeps any callback already set there and runs it after the fade. If the conversation does not start, the framework fades the screen back at once and `Narrate` returns `false`. If the start throws after the conversation has begun, `Narrate` returns `true`, and the fade back happens when that conversation ends.
- An invalid `modId` throws `ArgumentException`, as for the other `Dialogue` calls.
- The key `narration` belongs to `Narrate`. Each call removes the mod's own conversation with that key first. `Add` and `AddBranch` refuse the key `narration` (`ReservedKey`), and the refusal is logged once.

## Self-check

Two settings in the [framework's config](../api-core/plugin.md#config-file-settings) turn on checks that log their result at start:

- `Debug.DialogueAuthoringSelfCheck` checks the definition rules, the line links and the speakers.
- `Debug.DialogueResponsesSelfCheck` checks the anchor matching and the id allocation for responses.
