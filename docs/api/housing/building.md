---
title: Building
parent: Housing
grand_parent: Places and travel
nav_order: 4
---

# Building

`Building` lists, places, moves and removes the furniture the player placed, and unlocks furniture blueprints. This page lists every call, property and event, with the rules behind them.

This page uses `Vector3`, `Quaternion`, `GameObject`, `Transform` from Unity. The [glossary](../../glossary.md#types-that-come-from-unity) says what each one is.
{: .unity-types}

A [building area](../../glossary.md#building-area) is one room, shop or other space where furniture can be placed. The functions that take an `areaId` use its id. `AllAreaIds()` gives all of them.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>Building</code> <span>furniture the player places</span></p>

This is the class you start from. It lists the building areas and the furniture placed in them. `Place`, `Remove` and `Move` change that furniture.

<div class="api-pair" markdown="1">

```csharp
public static class Building
{
    public static List<string> AllAreaIds();
    public static List<FurnitureItem> GetPlacedFurniture(string areaId);
    public static List<FurnitureManagerInfo> Managers(string sceneName);
    public static List<BlueprintInfo> AllBlueprints();
    public static bool IsBlueprintKnown(string name);
    public static bool UnlockBlueprint(string name);
    public static bool UnlockBlueprint(string name, bool silent);
    public static FurnitureItem Place(string managerGuid, string furnitureId, Vector3 position, Quaternion rotation, Quaternion childRotation, int skin, int altPrefab);
    public static FurnitureItem Place(string managerGuid, string furnitureId, Vector3 position, Quaternion rotation, Quaternion childRotation, int skin, int altPrefab, bool requireKnown);
    public static bool Remove(string guid);
    public static bool Remove(FurnitureItem item);
    public static FurnitureItem Find(string guid);
    public static bool Move(string guid, Vector3 position, Quaternion rotation);
    public static bool Move(FurnitureItem item, Vector3 position, Quaternion rotation);

    public static event Action<FurnitureItem> FurniturePlaced;
    public static event Action<FurnitureItem> FurnitureRemoved;
    public static event Action<FurnitureItem> FurnitureLoaded;
    public static bool Applying { get; }
    public static bool IsAvailable { get; }
}
```

| Method | Returns | What it does |
|---|---|---|
| `AllAreaIds()` | `List<string>` | The id of every building area. |
| `GetPlacedFurniture(areaId)` | `List<FurnitureItem>` | The furniture the player placed in one area. |
| `Managers(sceneName)` | `List<FurnitureManagerInfo>` | The building areas in one scene, or in all loaded scenes. |
| `AllBlueprints()` | `List<BlueprintInfo>` | Every furniture blueprint the game knows, whether the player has it or not. |
| `IsBlueprintKnown(name)` | `bool` | `true` when the player has this blueprint. |
| `UnlockBlueprint(name)` | `bool` | Gives the blueprint and shows the game's "new item" message. |
| `UnlockBlueprint(name, silent)` | `bool` | Gives the blueprint. `silent` set to `true` skips the message. |
| `Place(managerGuid, furnitureId, position, rotation, childRotation, skin, altPrefab)` | `FurnitureItem` | Places a piece. `null` when refused. |
| `Place(managerGuid, furnitureId, position, rotation, childRotation, skin, altPrefab, requireKnown)` | `FurnitureItem` | The same, but refuses unknown furniture when `requireKnown` is `true`. |
| `Remove(guid)` | `bool` | Takes a placed piece out of the world. `true` when removed. |
| `Remove(item)` | `bool` | The same, for a piece you hold. |
| `Find(guid)` | `FurnitureItem` | The placed piece with this `Guid`. `null` when there is none. |
| `Move(guid, position, rotation)` | `bool` | Moves a placed piece. `true` when moved. |
| `Move(item, position, rotation)` | `bool` | The same, for a piece you hold. |

| Property | Type | What it means |
|---|---|---|
| `Applying` | `bool` | `true` while `Place`, `Remove` or `Move` is running. |
| `IsAvailable` | `bool` | `false` when a furniture event could not be set up. |

| Event | Gives you | When it fires |
|---|---|---|
| `FurniturePlaced` | `FurnitureItem` | Right after a piece of furniture is placed. |
| `FurnitureRemoved` | `FurnitureItem` | Right after a piece of furniture is taken out. |
| `FurnitureLoaded` | `FurnitureItem` | One of your mod's pieces was found after a save loaded. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>FurnitureItem</code> <span>one placed piece of furniture</span></p>

`GetPlacedFurniture`, `Find`, `Place` and the furniture events give you this, one per placed piece. Its `Guid` is only good until the next save. Do not keep it as a permanent key.

<div class="api-pair" markdown="1">

```csharp
public sealed class FurnitureItem
{
    public string Guid { get; }
    public string Id { get; }
    public string Title { get; }
    public string AreaId { get; }
    public Vector3 Position { get; }
    public Quaternion Rotation { get; }
    public Quaternion ChildRotation { get; }
    public int Skin { get; }
    public int AltPrefab { get; }
    public string ManagerGuid { get; }
    public string ParentGuid { get; }
    public string AttachedGuid { get; }
    public GameObject ParentObject { get; }
    public GameObject AttachedObject { get; }
    public bool FromApi { get; }
    public GameObject Object { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `Guid` | `string` | The piece's save id. `null` until the game saves. |
| `Id` | `string` | The furniture's id. It starts with `obx:` for a mod's furniture. Otherwise it is the game's path. |
| `Title` | `string` | The name the game shows. |
| `AreaId` | `string` | The id of the building area the piece is in. |
| `Position` | `Vector3` | Where the piece is, in world coordinates. |
| `Rotation` | `Quaternion` | Its turn relative to the building area, as the game saves it. |
| `ChildRotation` | `Quaternion` | The turn of its first child object, relative to the piece. Identity when there is no child. |
| `Skin` | `int` | Which look it has. `0` is the normal look. |
| `AltPrefab` | `int` | Which alternative shape it has. `0` is the normal shape. |
| `ManagerGuid` | `string` | The id of the area's furniture manager. May be `null`. |
| `ParentGuid` | `string` | The `Guid` of the piece it sits on. `null` when none. |
| `AttachedGuid` | `string` | The `Guid` of the piece it is attached to. `null` when none. |
| `ParentObject` | `GameObject` | The object of the piece it sits on. `null` when none. |
| `AttachedObject` | `GameObject` | The object of the piece it is attached to. `null` when none. |
| `FromApi` | `bool` | `true` when the event was raised inside `Place` or `Remove`. Your mod can use it to skip its own echo. |
| `Object` | `GameObject` | The placed object itself, not a copy. Unity treats it as `null` once it is destroyed. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>FurnitureManagerInfo</code> <span>one building area</span></p>

`Managers(sceneName)` gives you one of these for each building area. Its `Guid` is the same value as an id from `AllAreaIds`.

<div class="api-pair" markdown="1">

```csharp
public sealed class FurnitureManagerInfo
{
    public string Guid { get; }
    public string SceneName { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `Guid` | `string` | The area's id, the same value as an id from `AllAreaIds`. |
| `SceneName` | `string` | The [scene](../../glossary.md#scene) the area is in. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>BlueprintInfo</code> <span>one furniture blueprint</span></p>

`AllBlueprints()` gives you one of these for each furniture blueprint the game knows. Pass its `Name` to `UnlockBlueprint` to give the player that blueprint.

<div class="api-pair" markdown="1">

```csharp
public sealed class BlueprintInfo
{
    public string Name { get; }
    public string Title { get; }
    public bool Known { get; }
}
```

| Property | Type | What it means |
|---|---|---|
| `Name` | `string` | The blueprint's name. The developer console accepts the same name. |
| `Title` | `string` | The name the game shows. |
| `Known` | `bool` | `true` when the player already has the blueprint. |

</div>
</div>

## Things to know

- `GetPlacedFurniture` lists the furniture the player placed in an area. Furniture that comes with the building is not included. <span class="tech">(It reads `FurnitureManager.placedFurniture`.)</span>
- `Managers(sceneName)` lists the building areas in one [scene](../../glossary.md#scene). If `sceneName` is `null` or empty, it lists every loaded area. Each entry has the area's id and the name of its scene. <span class="tech">(It reads `BuildingSystem.furnitureManagers` and matches `gameObject.scene.name`.)</span>
- `Managers` only lists areas that have started, so it is empty on the main menu. Several areas in different scenes can share an id. Use the scene name to tell them apart.
- There are three places a position can be measured from. `Position` is in the world. `Rotation` is relative to the building area, as the game saves it. `ChildRotation` is relative to the piece, for its first child object. <span class="tech">(`Rotation` is the root's `localRotation` under the manager transform (`gameObjectMainRotation` in the save). `ChildRotation` is the `localRotation` of child 0 (`gameObjectFurnitureRotation`).)</span>
- `Place` takes the same three values, so a value you read on one computer can be passed to another. That only works when both computers have the same scene and the same area positions, which is an assumption.
- `Place` puts a piece in an area the way build mode does. It builds the piece from the furniture's model and turns it. It puts it at `position`, adds it to the area, and applies `skin` and `altPrefab`. `furnitureId` is a mod furniture id (`obx:...`) or the game's own path for a normal piece. `Place` returns the new piece, or `null` when it is refused. <span class="tech">(It instantiates the furniture's prefab under the manager (`altPrefab`, 0 for the default prefab), sets the root's `localRotation` to `rotation` and child 0's to `childRotation`, sets `position` in world space, adds the piece to `manager.placedFurniture` through `FurniturePlaceable.SetFurnitureInfo`, and applies `skin` and the alternative prefab through `FurniturePlaceable.UpdateMeta`.)</span>
- `Place` raises `FurniturePlaced` once, after the piece is complete. The event has `FromApi` set to `true`.
- If `Place` fails half way, it cleans up. When a step fails after the object was built, the framework destroys the object, removes it from the area's list and returns `null`.
- The first form of `Place` does not check the furniture id against the list of known furniture (see [Catalogue](catalogue.md)). The second form can. If another player decides the id (co-op), always pass `true`.
- A piece placed in this play session has a `Guid` of `null` until the game saves it. After a save, the game builds the id from the area's id and the piece's place in the list. A piece's `Guid` changes when a save runs after another piece in the same area was removed. Do not use `Guid` as a permanent key. <span class="tech">(Build mode does not set it. `FurnitureManager.OnSavingGameTertiary` sets it to the manager's GUID plus an index. An empty GUID is `null` in `FurnitureItem` (`Guid`, `ManagerGuid`, `ParentGuid` and `AttachedGuid`), so a parent or attached piece placed this session has a `null` `ParentGuid` or `AttachedGuid` until the next save. Use `ParentObject` or `AttachedObject` to link such pieces.)</span>
- For co-op, do not send a `Guid`, a list position or `Object` to another player as the id of a piece. The game renumbers every piece on each save. The number depends on the order of the list, which a removal or a move can change.
- For co-op, the host should make its own id when a piece is placed. It keeps the list of ids in the co-op code, not in a save entry. Send that id, and call `Find` again after each save to find the new `Guid`.
- `Remove(guid)` takes a placed piece out of the world without giving the item to the player.
- `Remove` raises `FurnitureRemoved` before the piece is destroyed. The event has `FromApi` set to `true`. `Remove` does not give the item to the player, and it does not run the piece's own "taken" signal. <span class="tech">(It does not run the game's take step that adds the item to the player's stock, nor the piece's `onTaken` relay.)</span>
- `Remove(item)` does the same for a piece you hold. It works for a piece with no `Guid`, which the `Guid` version can't reach. <span class="tech">(It finds `item.Object` by object identity in a manager's `placedFurniture`.)</span>
- `Find(guid)` returns the placed piece with that `Guid`. It returns `null` when no piece matches or `guid` is empty. It looks in the same list as `Remove(guid)`, so furniture that came with the building is not found.
- A `Guid` is only good until the next save.
- `Move` puts a placed piece somewhere else. It sets the piece's world `position` and its turn relative to the area, the same values `Place` uses. The change is saved on the next save. <span class="tech">(It sets the root's `localRotation`. The save reads both from the transform.)</span>
- `Move` does not turn the piece's first child. As in `Place`, the new spot is not checked. `Move` raises no event. <span class="tech">(It does not run the game's take-and-place: the game's move starts from the player's aim (`BuildingSystem.Move`), then `BuildingSystem.Place` puts the piece down.)</span>
- `Applying` is `true` while `Place`, `Remove` or `Move` is running. A handler of `FurniturePlaced` or `FurnitureRemoved` can check it, or the `FromApi` value, to tell your mod's own change from a player's action. A player's move does not set it.
- There is no "moved" event. When the player moves a piece, you see `FurnitureRemoved` and then `FurniturePlaced`. <span class="tech">(The game has no place where the framework can notice a finished player move. `BuildingSystem.Move` only lifts the piece, and the new spot is known only in `BuildingSystem.Place`.)</span>
- `AllBlueprints()` lists every furniture blueprint the game knows, whether the player has it or not. `Known` says whether the player has it. <span class="tech">(It reads the game's furniture blueprint controller.)</span>
- `UnlockBlueprint(name)` gives the player the blueprint and shows the game's "New item" message. `UnlockBlueprint(name, true)` gives it without the message. The Items panel still marks the blueprint as new. <span class="tech">(The first form calls `TaskItem.GiveItemWithResults`. The silent form uses `TaskItemsManager.AddTaskItem`.)</span>
- `FurniturePlaced` fires once for each placed piece. It fires when build mode places a piece, and when a script adds a piece. It does not fire when a save loads, or for furniture that came with the building. <span class="tech">(For build mode it is raised from the finalizer on `BuildingSystem.Place`, after the game has set the rotation and the parent and attached links, so `Rotation`, `ChildRotation`, `ParentGuid` and `AttachedGuid` are final. For a script it is raised from `FurnitureManager.PlacedFurniture` (from `PlaceFurniture`). Both go through `FurniturePlaceable.SetFurnitureInfo`.)</span>
- `FurniturePlaced` is not raised when `Place` throws. The game has no undo, so a piece added before the throw stays placed, with no event.
- Some items raise a nearly empty `FurniturePlaced` when they are put down in build mode. Its `Id` is `null` and its `Title` is the object's name. Its `Guid` is `null`, its `Skin` and `AltPrefab` are 0, and its `ManagerGuid` is the area. Recognise these by `Id == null`.
- Only items whose model has a collectible-item part do this. An item without one (for example a coin) raises nothing from that placement. A script that places an item stack gets the same `null`-`Id` item once. <span class="tech">(Build mode places a piece with an `itemStack` through `BuildingSystem.Place`, which calls `ItemOperations.SetCollectibleItemValues`. That reaches `SetFurnitureInfo` only when the prefab has a `CollectibleItem`, and then the flag is set. The placeable's `furniture` field is still `null` at that point, so there is no `Furniture` to read. A scripted item-stack spawn goes through `PlaceFurniture` and `PlacedFurniture`.)</span>
- `FurnitureRemoved` fires when the game takes a piece out of an area. Moving a piece takes it out, so a move raises `FurnitureRemoved` and then, when it is put down, `FurniturePlaced`. The piece is passed to you before the game destroys it. <span class="tech">(It is raised from `FurniturePlaceable.Take`.)</span>
- `FurnitureLoaded` fires after each save load, for each of your mod's pieces in the loaded levels. That includes the load after a level change. Use it to re-attach your mod's behaviour to its pieces.
- Write the `FurnitureLoaded` handler so that running it twice on the same piece does no harm. For example, check `GetComponent` before `AddComponent`. Only pieces whose id starts with `obx:` raise it. The game's own pieces never do.
- A piece placed in this session does not raise `FurnitureLoaded`, so use `FurniturePlaced` for those. Check `item.Id != null && item.Id.StartsWith("obx:", StringComparison.Ordinal)`.
- `FurnitureLoaded` fires early in the load. It runs after [`FurnitureRegistry.Loaded`](../furniture/defining-furniture.md) and after your mod's [saved data](../../guides/getting-started/save-data.md) has loaded. It runs before the [`Events.Loaded`](../../api-core/events.md) handlers that your mod adds. `FurnitureRegistry.Get` already works in it. Writes such as `Building.Place` may still be refused at that point, so only read and attach. <span class="tech">(It runs during the framework's own `Events.Loaded` handling. `SaveData.Busy` may still be `true`.)</span>
- `FurnitureLoaded` only looks at levels that are loaded when the load finishes. Pieces in a scene that loads later, or in a level that is not loaded yet, raise nothing until a load includes their level. For those, use `Events.SceneLoaded` with `GetPlacedFurniture`.
- A mod piece whose build failed still raises `FurnitureLoaded`. It is a stand-in that keeps its `obx:` id. `FurnitureRegistry.Get(item.Id)` gives `null` for it.
- Pieces already in a save do not raise `FurniturePlaced`. After `Events.Loaded`, call `GetPlacedFurniture(areaId)` for each area in `AllAreaIds()` to find them. Your mod's own pieces in a save raise `FurnitureLoaded` for you.

## Limits

- `Place` returns `null` while a save is loading or saving, or when the area or the furniture is not known. It also returns `null` when the model is missing, or when `skin` or `altPrefab` is out of range.
- `Place` also returns `null` for an empty `managerGuid` or `furnitureId`. It does the same for a position or turn with a number that is not real (not a number, or infinite). In those two cases the problem is written to the [log](../../guides/getting-started/logging.md) once, and nothing is placed.
- With `requireKnown` set to `true`, `Place` returns `null`, logs once and places nothing when `IsKnownFurniture(furnitureId)` is `false`. A path that loads something that is not furniture is released and refused, with `null`.
- `Remove(guid)` returns `false`, and changes nothing, when the piece is not placed. A piece with no `Guid` is not found until a save.
- `Remove(guid)` also returns `false`, and changes nothing, when the game does not allow taking the piece, or a save is loading or saving. The same happens when another piece sits on it or is attached to it, or when it holds things. An empty or `null` `guid` returns `false` and logs once. <span class="tech">(The game flag is `cantTake`. A piece holds things when it has a storage with items, a liquid storage with liquid, a crafting job in progress, or fuel that is burning or in a slot. It also holds things when its trash box has items in it, because the game hands those items to the player when the piece is taken. A collectible item piece and a piece that needs an item to take are refused too. The game keeps such a piece's contents when it is moved, and `Remove` would destroy them, so the piece stays in the world.)</span>
- `Remove(item)` returns `false` when the piece was destroyed or is no longer placed. `item` must not be `null`, or it throws `ArgumentNullException`.
- `Move` returns `false`, and changes nothing, when the position or turn has a number that is not real, or a save is loading or saving. It also returns `false` when the game does not allow taking the piece. The same happens when the piece sits on or is attached to another piece, or other pieces sit on it or are attached to it. <span class="tech">(It makes the same `cantTake` and parent or attached checks that `Remove` makes.)</span>
- `Move` also returns `false` for an empty `guid`, and logs once. A `null` `item` throws `ArgumentNullException`.
- Both `UnlockBlueprint` forms return `false` without doing anything when the player already has the blueprint or the name is not found. An empty or `null` name returns `false` and logs once.
- `Building.IsAvailable` is `false` when a furniture event could not be set up (see [Details](details.md#if-something-fails-to-set-up)). A rent problem does not affect it.

**Common problems**

- `Place` returns `null`: one of the cases in the first three bullets above applies. Take the area id from `AllAreaIds()` and the furniture id from `Catalog()`.
- `Remove(guid)` returns `false` for a piece placed in this session: the piece has no `Guid` until the game saves. Use `Remove(item)` with the `FurnitureItem` you hold.
- `FurnitureLoaded` does not fire for a piece: the piece was placed in this session, its id does not start with `obx:`, or its level was not loaded when the load finished. Use `FurniturePlaced` for new pieces, and `Events.SceneLoaded` with `GetPlacedFurniture` for later levels.
- Pieces already in a save raise no `FurniturePlaced`: after `Events.Loaded`, call `GetPlacedFurniture(areaId)` for each area in `AllAreaIds()`.
