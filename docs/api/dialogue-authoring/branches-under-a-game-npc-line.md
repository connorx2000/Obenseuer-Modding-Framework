---
title: Branches under a game NPC line
parent: Dialogue authoring
grand_parent: Story
nav_order: 4
---

# Branches under a game NPC line

`Dialogue.AddBranch` links a conversation made of your own lines into a game conversation. Use it to build a menu tree under a game character's line. This part is experimental.

The branch is a conversation written with `DialogueAuthoring`. So the rules for keys, lines, and putting it back on the [Writing a conversation](writing-a-conversation.md) page apply to it. Both overloads link your branch under the "anchor" line. You give the anchor by its text or by its id.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>Dialogue</code> <span>links your branches into game conversations</span></p>

Call `AddBranch` and keep the key it gives back. You pass that key to `RemoveBranch` later.

<div class="api-pair" markdown="1">

```csharp
public static class Dialogue
{
    public static string AddBranch(string modId, string conversationTitle, string anchorText, ConversationDefinition branch);
    public static string AddBranch(string modId, string conversationTitle, int anchorId, ConversationDefinition branch);
    public static bool RemoveBranch(string modId, string key);
}
```

| Method | Returns | What it does |
|---|---|---|
| `AddBranch(modId, conversationTitle, anchorText, branch)` | `string` | Links your branch under the anchor line, found by its text. Returns the branch key. |
| `AddBranch(modId, conversationTitle, anchorId, branch)` | `string` | Links your branch under the anchor line, found by its id. Returns the branch key. |
| `RemoveBranch(modId, key)` | `bool` | Removes one of your branches. `false` for another mod's key or an unknown key. |

</div>
</div>

## Putting it together

This example adds a "Membership" menu under the line "What can I do for you?". Its reply shows the days left and changes each visit. `MembershipDaysLeft()` stands for your own method that returns a whole number.

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

string key = Dialogue.AddBranch(Mod.Id, "Shop.Greeting", "What can I do for you?", membership);
```

The tree starts at the game's line "What can I do for you?". Then come your player choice "Membership" and the character's reply with the days left. The two choices "Renew for 30 days." and "Goodbye." follow. The reply text comes from a [text callback](text-callbacks.md).

## Things to know

- The anchor is chosen the same way as for [`AddResponse`](responses-to-a-game-conversation.md), and the same refusals apply. For a line that has a `TextCallback`, the original written text is the callback's markup. Use the `anchorId` overload for that line.
- The branch starts at its first line, and that line must be a player line. The anchor gets one link to that line, at the priority of its other player choices.
- The branch's character lines are spoken by the anchor's character, so that character's name and picture show. `branch.Speaker` may be `null` or the name of that same character.
- `AddBranch` returns a key: your [mod id](../../glossary.md#mod-id), a dot and eight characters. It comes from the conversation title, the anchor and `branch.Key`.
- The branch's own conversation title is your mod id, a dot and `branch.Key`. So a key already in the game is refused with `AlreadyExists`, also when it is used under another anchor.
- The framework puts the link back by itself, as for responses. It does this each time a [scene](../../glossary.md#scene) (one loaded area of the game) loads, and after each load of a save.
- `RemoveBranch` removes the link and the branch's conversation.
- `Dialogue.RemoveAll(modId)` and `DialogueAuthoring.Clear(modId)` also remove your branches. They keep any they cannot remove right now. A branch's conversation cannot be removed with `DialogueAuthoring.Remove`.
- Branches are never saved. A save made while a branch is in place loads fine without your mod.

## Limits

- `AddBranch` refuses a branch whose first line is a character line, because the player would get no choice.
- `AddBranch` refuses a branch whose speaker is not the anchor's character. Any other name is refused.
- Adding the same branch again is refused, so call `RemoveBranch` first. A key already in the game is refused with `AlreadyExists`, also when it is used under another anchor.
- `RemoveBranch` returns `false` for another mod's key or an unknown key. It also returns `false` while a save is loading or being written, when the dialogue system is not ready, and while any conversation is running. The player can reach a branch through another conversation's link.
- `AddBranch` returns `null` when it is refused, and logs the reason once. That happens in these cases:
  - The dialogue system is not ready.
  - A save is loading or being written.
  - The branch is not valid.
  - The branch key is `narration`.
  - The first line is not a player line.
  - The conversation or the anchor is not found.
  - The branch speaker is not the anchor's character.
- An empty `conversationTitle` throws an `ArgumentException`. A `null` `branch` throws an `ArgumentNullException`, and an invalid `modId` throws an `ArgumentException`.
- A branch cannot send the player back into the game's conversation. A line without `Next` ends the conversation. So in the example above, "Goodbye." ends the talk and the game's menu is not shown again. Put any choice that should go back to the game's menu in the game conversation itself, not in a branch.

**Common problems**

- Adding the same branch again returns `null`: a branch that is already added is refused. Call `RemoveBranch` first.
- The game's menu does not show again after a branch choice: a branch cannot send the player back into the game's conversation. Put that choice in the game conversation itself.
