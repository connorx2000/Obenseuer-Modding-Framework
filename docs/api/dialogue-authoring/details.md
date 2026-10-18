---
title: Details
parent: Dialogue authoring
grand_parent: Story
nav_order: 8
---

# Details

The game and framework versions this API was tested against, the thread rule, and what happens when the dialogue setup fails. Read this when every dialogue call is refused for no clear reason.

Target: Obenseuer 0.5.08 (buildid 25969064), Unity 6000.0.77f1 Mono, BepInEx 6 Unity.Mono ([be.788](https://builds.bepinex.dev/projects/bepinex_be/788/BepInEx-Unity.Mono-win-x64-6.0.0-be.788%2B5b766a3.zip), the tested build) or BepInEx 5 (5.4.23 or later).
Namespace: `ObenseuerCore`. Source: `src/Story/DialogueAuthoring.cs`. Logs go to `BepInEx\logs\obenseuer.moddingframework.dialogueauthoring.log`.

Everything not listed in this reference is internal and may change without notice. Items marked experimental may change too.

This API builds on `Dialogue` (`src/Story/Dialogue.cs`), so it needs `Dialogue.IsAvailable` to be `true`. The framework changes some of the game's methods when it starts so that `Dialogue` works. This set of changes is a [feature group](../../glossary.md#feature-group) (the framework's changes for one feature). Its name is `obenseuer.moddingframework.dialogue`, the name you will see in the [log](../../guides/getting-started/logging.md) for responses, branches and narration. Writing a conversation itself changes no game method.

## When you can call it

- Call everything from your mod's normal code (`OnLoad`, event handlers, `Update`), not from a background thread you started. The game only runs safely on the main thread (see the [glossary](../../glossary.md#main-thread)).
- Writes are refused while a save is loading or being written. The calls return `null`, `false` or a refusal reason instead of changing anything.

## If something fails to set up

The framework changes some of the game's methods when it starts. If a game update breaks one of them, the framework turns off the [feature group](../../glossary.md#feature-group) that depends on it. It writes error lines to the group's log file in `BepInEx\logs`. The rest of the framework keeps working.

- If the dialogue system could not be set up, `Dialogue.IsAvailable` is `false`. [`Narrate`](narration.md), [`AddResponse`](responses-to-a-game-conversation.md) and [`AddBranch`](branches-under-a-game-npc-line.md) then refuse their calls, and [`DialogueAuthoring.Add`](writing-a-conversation.md) gives `NotReady`.
