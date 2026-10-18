---
title: Narration
parent: Dialogue authoring
grand_parent: Story
nav_order: 6
---

# Narration

`Dialogue.Narrate(modId, text)` shows `text` in the game's dialogue box on the game's black screen. Use it for a short piece of narration, such as "Three days later". This part is experimental.

`Narrate` is built on [`DialogueAuthoring`](writing-a-conversation.md), with one conversation for each mod. The key is `narration`, so the title is your [mod id](../../glossary.md#mod-id), a dot, and `narration`. Each call replaces that conversation's text, then starts it through the game's black screen.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>Dialogue</code> <span>shows narration on a black screen</span></p>

Call `Narrate` with your mod id and the text, and check the `bool` it returns: `false` means the narration did not start.

<div class="api-pair" markdown="1">

```csharp
public static class Dialogue
{
    public static bool Narrate(string modId, string text);
}
```

| Method | Returns | What it does |
|---|---|---|
| `Narrate(modId, text)` | `bool` | Shows narration on a black screen. `true` when a conversation is running after the call. |

</div>
</div>

## Things to know

- `Narrate` returns `true` when a conversation is running after the call. That can be a different conversation, when a handler from `Dialogue.Intercept` (an [interceptor](../../glossary.md#interceptor)) redirects the start.
- The screen starts to fade to black at once, before the text shows.
- When the conversation ends, the framework fades the screen back. If the conversation does not start, the framework fades the screen back at once and `Narrate` returns `false`. If the start throws an error after the conversation has begun, `Narrate` returns `true`. The fade back then happens when the conversation ends. <span class="tech">(It uses the end-of-talk callback on the black screen's talk component (`BlackCanvas.instance.interactableTalk.onEndTalkCallback`). The framework keeps any callback already set there and runs it after the fade.)</span>
- The key `narration` belongs to `Narrate`. Each call first removes your own conversation with that key. `Add` and `AddBranch` refuse the key `narration` (`ReservedKey`), and the refusal is logged once.

## Limits

- `Narrate` returns `false` when the text is `null`, empty, only spaces, or over 1000 characters. It also returns `false` when the text has a control character other than a line break (`\n` or `\r`).
- `Narrate` also returns `false` in these cases:
  - The dialogue system is not available or not ready.
  - A save is loading or being written.
  - A conversation is already running.
  - The black screen is not there.
  - The player is waiting or asleep.
  - `DialogueAuthoring.Add` refuses the conversation, for example when the game has no player character.
- Do not call it from a wake-up callback while the sleep text is still waiting to show. Starting a conversation wakes the player and cuts that text off, so the call is refused. <span class="tech">(The check is that `WaitingController.eventDialogue` is empty.)</span>
- An invalid `modId` throws an `ArgumentException`.
