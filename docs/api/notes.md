---
title: Notes
parent: Phones, mail and media
grand_parent: API reference
nav_order: 6
---

# Notes

`Notes` lets your mod list, check and give the single notes the player collects, and tells you when the player gets one. Use it when a quest step or a reward depends on a note.

Status: experimental.

## What it is

In the game, the player collects notes and emails. These are single sheets with a title and some text, which the player can read again in their notes window. This API lets a mod do these jobs:

- List all the single notes the game has.
- List the ones the player holds.
- Check whether the player holds one.
- Give one to the player.
- Get told when the player gets one.

Book pages are not covered here. Use `Books` (see [Books](books.md)) to read books, and `Hud.ShowNote` (see [HUD](hud.md)) to open a note or a page on screen. Notebooks that the player writes in are not covered either.

## When to use it

- Give the player a note when a quest step is done.
- Check that the player has read or collected a note before something happens.
- Log or react when the player collects a note, however they got it.

## Quick example

This mod writes a [log](../guides/getting-started/logging.md) line with the title of every note the player collects. Once a save is loaded, it also logs how many single notes the game has and how many the player holds.

```csharp
using ObenseuerCore;

[ModInfo("example.notewatch", "Note Watch", "0.1.0")]
[ModDependency("obenseuer.moddingframework", "0.11.0")]
public sealed class NoteWatch : FrameworkMod
{
    protected override void OnLoad()
    {
        Notes.Collected += args => Log.Info("Collected note: " + args.Title);
        Events.WhenReady(() => Log.Info("The game has " + Notes.All().Count + " notes, the player holds " + Notes.Held().Count));
    }
}
```

`Collected` is an event: the framework calls your code each time it happens. [`Events.WhenReady`](core-helpers/events-whenready.md) runs your code once a save is loaded (see [Using events](../guides/getting-started/using-events.md)).

## Common tasks

Give the player a note by its id. `Give` returns `true` only when this call added the note.

```csharp
NoteEntryInfo first = Notes.All()[0];
if (Notes.Give(first.Id)) Log.Info("Gave the note " + first.Title);
```

Check whether the player already has a note.

```csharp
bool has = Notes.Has("some-note-id");
```

List the notes the player holds.

```csharp
foreach (NoteEntryInfo note in Notes.Held())
    Log.Info(note.Id + ": " + note.Title);
```

## Reference

This is the full list of what the API gives you. Text in brackets at the end of a note is extra detail for experienced modders, and you can skip it on a first read.

### Notes

`Notes` reads and gives the game's single notes. Each note is described by a `NoteEntryInfo`.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>Notes</code> <span>list, check and give single notes</span></p>

Call `Give(id)` to add a note the game already has. Call `Has(id)` or `Held()` to check what the player holds. `Collected` fires each time a new note is collected.

<div class="api-pair" markdown="1">

```csharp
public static class Notes
{
    public static bool IsAvailable { get; }
    public static List<NoteEntryInfo> All();
    public static List<NoteEntryInfo> Held();
    public static bool Has(string id);
    public static bool Give(string id);
    public static event Action<NoteCollectedArgs> Collected;
}
```

| Method | Returns | What it does |
|---|---|---|
| `All()` | `List<NoteEntryInfo>` | Every single note the game has. |
| `Held()` | `List<NoteEntryInfo>` | The single notes the player holds, in the order they were collected. |
| `Has(id)` | `bool` | `true` when the player holds this note. |
| `Give(id)` | `bool` | Gives the player the note. `true` only when this call added it. |

| Property | Type | What it means |
|---|---|---|
| `IsAvailable` | `bool` | `false` when the collected event could not be set up. |

| Event | Gives you | When it fires |
|---|---|---|
| `Collected` | `NoteCollectedArgs` | Right after the player collects a note. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>NoteEntryInfo</code> <span>one note</span></p>

`All()` and `Held()` give you one of these. Use its `Id` with `Has` and `Give`.

<div class="api-pair" markdown="1">

```csharp
public sealed class NoteEntryInfo
{
    public string Id { get; }
    public string Title { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `Id` | `string` | The note's id. Use it with `Has` and `Give`. |
| `Title` | `string` | The note's title. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>NoteCollectedArgs</code> <span>what Collected gives you</span></p>

`Collected` gives you one of these when the player gets a note they did not have. `Id` and `Title` say which note it was.

<div class="api-pair" markdown="1">

```csharp
public sealed class NoteCollectedArgs
{
    public string Id { get; }
    public string Title { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `Id` | `string` | The note's id. |
| `Title` | `string` | The note's title. |

</div>
</div>

#### Things to know

- `All()` lists every single note the game has, and leaves out book pages. It is the same list the game's developer console uses. <span class="tech">(It lists the `NoteInfo` assets in the game's `Resources/Notes` folder and skips any with `NoteInfo.book` set.)</span>
- `Held()` lists the single notes the player has collected, in the order they were collected. Book pages are left out. <span class="tech">(It reads `NoteController.notes` and skips entries that have a book.)</span>
- A note's id is the note's own id in the game files. If it has none, the id is where the note is stored. If it has neither, the id is its title. <span class="tech">(`Id` is `NoteInfo.Id`: the `id` field, else `addressableAssetPath`, else `title`.)</span>
- `Has(id)` and `Give(id)` find the note by its id, and do nothing if two notes share an id. When two notes in the game have the same id, `Has` and `Give` can't tell them apart, so both return `false` for that id.
- `Give(id)` adds the note the way the game's own mail, tasks and bundles do. The notes window and the [save](../guides/getting-started/save-data.md) both update as they would in the game. It does not show the "New note" pop-up, the same as the game's mail and tasks. <span class="tech">(It calls the game's `NoteController.AddNote`.)</span>
- `Collected` fires once for each single note that goes from not held to held, however it was added. That includes `Give`, mail, tasks and the console. It does not fire again for a note the player already holds, and it does not fire when a save loads.
- `IsAvailable` is `false` when the collected event could not be set up. The reads and `Give` keep working when it is `false`. See [If something fails to set up](#if-something-fails-to-set-up).

#### Limits

- `Give` returns `false` when it did not add the note. That happens when the id is not a note the game has, or the player already holds the note. It also happens when a save is loading or being written. The same goes when the game's note controller is not running (on the main menu, or before a save is loaded). An id of `null` also returns `false`, and writes one line to the log.

### What you can't do (yet)

- You can't add a new note of your own. Every note is a data file that ships with the game, and the save stores a note by the path of that file. A mod can't add a note without shipping such a file, so `Give` only works for notes the game already has. <span class="tech">(A note is a `NoteInfo` asset, stored in the save by its addressable path.)</span>

## Details

Target: Obenseuer 0.5.08 (buildid 25969064), Unity 6000.0.77f1 Mono, [BepInEx](../glossary.md#bepinex) 6 Unity.Mono ([be.788](https://builds.bepinex.dev/projects/bepinex_be/788/BepInEx-Unity.Mono-win-x64-6.0.0-be.788%2B5b766a3.zip), the tested build) or BepInEx 5 (5.4.23 or later).
Namespace: `ObenseuerCore`. Logs go to `BepInEx\logs\obenseuer.moddingframework.notes.log`.

Everything not listed in this reference is internal and may change without notice. Items marked experimental may change too.

### When you can call it

- Call everything from your mod's normal code, not from a background thread you started. The game and Unity only work safely on one thread, the main thread (see the [glossary](../glossary.md#main-thread)).
- It is safe to call any of it at any time, even on the main menu or while a save is loading. If no save is loaded, or the game's note controller is missing, the reads give an empty list or `false`.
- Your `Collected` handler runs inside the game's own call, one at a time. If one of your handlers throws an error, the framework logs it and still runs the other handlers.
- `Collected` also fires while a save is loading, if a note is added then. To ignore those, check [`SaveData.Busy`](../api-core/savedata.md). It is `true` while a save is loading or being written.

### What to check for and what comes back

- Nothing here throws. A `null` id gives `false` and one line in the log. Everything else gives its plain default (`false` or an empty list).
- `IsAvailable` only tells you about the `Collected` event. In code it is "set up, and the [feature group](../glossary.md#feature-group) is not turned off".

### If something fails to set up

The framework changes some of the game's methods when it starts, so it can tell your mod when a note is collected. If a game update breaks one of them, only that feature turns off. The framework writes error lines and a warning to the log. The rest of the API keeps working.

- The `Collected` event depends on the feature group `obenseuer.moddingframework.notes`. This is the name you will see in the log.
- If the framework can't change the game's method that adds a note, it turns the group off and `Collected` stops. It writes error lines and a warning to `BepInEx\logs\obenseuer.moddingframework.notes.log`. `All`, `Held`, `Has` and `Give` keep working. <span class="tech">(The game method is `NoteController.AddNote(NoteInfo, bool)`.)</span>

**Common problems**

- `Collected` never fires and `IsAvailable` is `false`: the event could not be set up, or the feature group is turned off. Look for the error lines and the warning in `BepInEx\logs\obenseuer.moddingframework.notes.log`. `All`, `Held`, `Has` and `Give` still work.
- `Collected` fires while a save is loading: a note was added during the load. To ignore those, check `SaveData.Busy`.
