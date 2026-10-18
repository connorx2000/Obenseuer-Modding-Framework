---
title: Obenseuer Modding Framework world props API
parent: API reference
nav_order: 120
---

# Obenseuer Modding Framework world props API

`WorldProps` lets a mod put its own objects in a [scene](../glossary.md#scene), such as a model, a light or a copy of a furniture piece. The framework brings them back on every load. Use it when your mod needs objects of its own in the game world.

Status: experimental.
Target: Obenseuer 0.5.08 (buildid 25969064), Unity 6000.0.77f1 Mono, [BepInEx](../glossary.md#bepinex) 6 Unity.Mono (be.755 or newer, be.788 recommended) or BepInEx 5 (5.4.23 or later).
Namespace: `ObenseuerCore`. Log owner: `obenseuer.moddingframework.worldprops`.

Everything not listed here is internal. Call it on the [main thread](../glossary.md#main-thread) only. Reads return a neutral default (`null`, `false`, an empty list) when no save or scene is ready. `Place` returns the live object or `null`. `Move` and `Remove` return `bool`, and `Clear` returns a count. The only exception is `ArgumentException` for a malformed argument.

A world prop is an object the framework puts in a scene for a mod and brings back on every load. It can be a model, a primitive, a vanilla furniture prefab, an item's world model, a light, or an empty anchor.

The framework keeps every prop in its own file inside the save folder (`mods\obenseuer.moddingframework.worldprops.json`), and **the game's own save never holds a prop**. A prop is a root object of its scene. The game's save pass walks the root objects of the active scene, so that pass does reach the prop. What keeps the save clean is the strip. A furniture or item copy is built under the framework's inactive holder with every `MonoBehaviour` removed. A prop whose copy keeps one is refused.

An interaction adds `ModInteraction`, which is an `Interactable` and so a `SavableScript`. It overrides no save hook, so the pass runs it and it writes nothing.

```csharp
public static class WorldProps
{
    public static event Action<WorldPropSpawned> Spawned;

    public static GameObject Place(string modId, string propId, string scene, Vector3 position, Quaternion rotation, Vector3 scale, WorldPropSource source, WorldPropOptions options);
    public static bool Move(string modId, string propId, Vector3 position, Quaternion rotation);
    public static bool Remove(string modId, string propId);
    public static GameObject Get(string modId, string propId);                 // the live object, null when it is not in a loaded scene
    public static List<WorldPropInfo> List(string modId, string scene);
    public static int Clear(string modId);                                     // drops every prop of the mod, returns how many
    public static bool OnUse(string modId, string key, Action<WorldPropUse> handler);
}

public sealed class WorldPropSource
{
    public static WorldPropSource Model(string modGuid, string key);           // a ModelRegistry handle
    public static WorldPropSource Primitive(WorldPrimitive primitive);         // Cube, Sphere, Cylinder, Plane
    public static WorldPropSource Furniture(string furnitureId, int alt);      // a vanilla furniture prefab
    public static WorldPropSource Item(int itemId);                            // the item's world model
    public static WorldPropSource Light(LightType type, Color color, float range, float intensity, float spotAngle);
    public static WorldPropSource Anchor();                                    // an empty object with an optional box
}

public sealed class WorldPropOptions
{
    public bool Collider = true;      // the prop keeps or gets a collider
    public bool IsStatic;             // sets GameObject.isStatic; a static prop cannot be moved
    public int Layer;                 // 0 (Default) only in this version
    public string InteractKey;        // the OnUse key, at most 64 characters, empty for no prompt
    public string Data;               // your own text, at most 256 characters, handed back on use
}

public sealed class WorldPropInfo
{
    public string ModId { get; }
    public string PropId { get; }
    public string Scene { get; }
    public WorldPropKind Kind { get; }        // Model, Primitive, Furniture, Item, Light, Anchor
    public Vector3 Position { get; }
    public Quaternion Rotation { get; }
    public Vector3 Scale { get; }
    public bool Live { get; }                 // true when the object is in a loaded scene now
    public bool Collider { get; }
    public bool IsStatic { get; }
    public int Layer { get; }
    public string InteractKey { get; }
    public string Data { get; }
}

public sealed class WorldPropSpawned
{
    public GameObject Object { get; }         // active and in its final pose
    public WorldPropInfo Info { get; }
    public string ModId { get; }
    public string PropId { get; }
    public string Scene { get; }
}

public sealed class WorldPropUse
{
    public string ModId { get; }
    public string Key { get; }
    public string PropId { get; }
    public string Scene { get; }
    public string Data { get; }
    public GameObject Object { get; }
}
```

The types `Vector3`, `Quaternion`, `Color` and `GameObject` come from Unity. The [glossary](../glossary.md#types-that-come-from-unity) says what each one is.

## Placing and moving

- `Place` records the prop. It returns the live object when `scene` is loaded. Until a save is loaded, it is refused with one log line, so a call on the main menu records nothing.
- **A scene that is not loaded is not a refusal**: the framework keeps the record, returns `null` and logs nothing. It builds the object when that scene next loads. `Place` writes the record to the mod file at once.
- `propId` is unique per mod: 1 to 64 characters, any characters. Placing an id that is already used replaces that prop. The old object is destroyed, the record is overwritten, and `Spawned` is raised again.
- `scene` is the scene's `name`, at most 128 characters. It is exactly the name the game uses as a level name and as the `.tnmt` file name. Read it with `SceneManager.GetActiveScene().name`.
- `scale` must be finite and between 0.01 and 100 on each axis. `position` and `rotation` must be finite.
- `Move` writes the record and moves the live object when it is live. In a scene that is not loaded, it updates the record only and returns `true`. `Move` is refused on a static prop.
- `Remove` drops the record and destroys the live object. `Clear` does the same for every prop of the mod.
- `options` may be `null`. That means a collider, not static, layer Default, no interaction and no data.
- Every write is refused while a save loads or saves ([`SaveData.Busy`](../api-core/savedata.md)), and nothing is written.

## Sources

| Source | Built from | Collider |
|---|---|---|
| `Model(modGuid, key)` | The mesh and materials of a [`ModelRegistry`](registry/models.md) handle, on a new object. | A box from the mesh bounds. |
| `Primitive(kind)` | `GameObject.CreatePrimitive`, with the game's own default material. | The primitive's own. It is dropped when `Collider` is off. |
| `Furniture(furnitureId, alt)` | `Building.PrefabOf`: a copy of the furniture prefab. | The prefab's own. All of them are dropped when `Collider` is off. |
| `Item(itemId)` | `ItemRegistry.FindInDatabase` then `Item.Prefab`: a copy of the world prefab. | The prefab's own. All of them are dropped when `Collider` is off. |
| `Light(type, color, range, intensity, spotAngle)` | A new object with one `Light`. | None. |
| `Anchor()` | A new empty object. | A box of one metre when `Collider` is on. |

Furniture and item copies are made under the framework's inactive holder. While the copy is inactive, **every `MonoBehaviour` on it is destroyed and `Animator.fireEvents` is set to false**. The renderer, mesh filter, colliders, animator, particles and lights are kept.

The components are destroyed from the last one back to the first. The pass repeats while the survivor count keeps falling, because Unity refuses to destroy a component that another kept component requires with `[RequireComponent]`. No `Awake`, `OnEnable` or `Start` ever runs on the copy. So no `Collectible`, no `FurniturePlaceable`, no `DestructibleList` entry and no expiry timer is created.

The framework keeps the copy only when **no `MonoBehaviour` is left on it at all**. Otherwise every surviving component is logged and the prop is refused. That is why a crafting station, a vending machine, a trade stand or a battery piece, whose `Storage` is required by another component, is refused rather than placed. The log line names the component that blocked it.

Every prop is settled before it is activated, so a prop does not fall, roll, sound or emit on its own:

- A `Rigidbody` becomes kinematic with gravity off.
- An `AudioSource` or a `ParticleSystem` has `playOnAwake` off and is stopped.

Start a sound or an effect again from `Spawned` if you want it live. The `Model` source builds a fresh mesh and material array from the handle every time. It frees both when the object is destroyed, whatever the state of that object. Textures are cached and shared and are never freed. The `Light` source accepts `Point` and `Spot` only.

## Interactions

`WorldProps.OnUse(modId, key, handler)` registers one handler per mod and key for the session. A prop whose `InteractKey` matches gets an interaction through [`Interactions.Add`](interaction/interactions.md) on every spawn. The key text is what the prompt shows, so pick something readable, for example `Open hatch`. The handler receives a `WorldPropUse` with the prop id, scene and `Data`, and can call `Get`.

- Register with `OnUse` before the prop spawns, in `Awake` or `OnLoad`. A prop whose key has no handler at spawn time shows no prompt and logs one line.
- A second `OnUse` for the same mod and key is refused with one log line.
- A prop with an interaction key may be built with no collider on its root. The framework then adds a box collider so the prompt can appear. This needs `Collider` to be on: an interaction with `Collider` off is refused at `Place`.

**Common problems**

- A prop shows no prompt: its key had no handler at spawn time. Register with `OnUse` before the prop spawns, in `Awake` or `OnLoad`.

## Timing

- The framework subscribes to [`Events.SceneLoaded`](../api-core/events.md). A load queues one restore for that scene through [`Events.WhenReady`](core-helpers/events-whenready.md), so props wait for the save data to be read.
- A queued scene is restored a frame later, only while `scene.isLoaded` and while no save or load is running. A scene that is no longer loaded is skipped, and its records stay.
- A `Single` load destroys the objects of the old scene. So the framework marks every prop not live and clears its queue before it restores the new scene.
- An additive scene that is unloaded later drops its props from the live map through `SceneManager.sceneUnloaded`. **Records always survive**, and a scene that comes back brings its props back.
- `Spawned` is raised once per spawn, with the object active and in its final pose. It is never raised on `Move`, and never while a save or load is running.
- There is no per frame scan. Work happens on the scene events, and on a hidden driver that walks a short queue of scenes waiting for their props.

## Limits and refusals

Every refusal returns a value and logs once through the framework's log for that owner.

| Refusal | Result |
|---|---|
| no save is loaded | `null`, nothing is recorded |
| a scene name over 128 characters | `ArgumentException` |
| a save loads or saves (`SaveData.Busy`) | `null`, `false` or `0` |
| a scale outside 0.01 to 100 per axis, or a non-finite pose | `null` or `false` |
| `Layer` other than 0 | `null` |
| an interaction key with `Collider` off | `null` |
| a mod's 257th prop | `null` |
| more than 1024 props live at once | Not spawned. The record is kept. |
| `Move` on a static prop | `false` |
| `Move` or `Remove` of an unknown id | `false` |
| a second `OnUse` for the same key | `false` |
| a model that is not loaded or not ready, an unknown furniture id, an unknown item id | `null`, nothing is recorded |
| a prop of a mod that is not installed | Not spawned. One log line per mod per session. Records are kept. |
| a call off the Unity main thread | `null`, `false` or `0` |

Piece limits: 256 records per mod, and 1024 live objects in total. Records of a mod that is not installed are kept. Install the mod again and its props return.

**Common problems**

- A recorded prop is not spawned: its mod is not installed, or more than 1024 props are live at once. The record is kept in both cases. For a mod that is not installed, install the mod again and its props return.

## The saved file

One entry per prop holds:

- the mod id, the prop id, the scene and the kind
- the source key, and the alternative or item id
- the position, rotation and scale
- the collider, static and layer options
- the interaction key, your data, and the light fields.

`Clean` drops an entry in these cases:

- a bad mod id
- an empty or too long prop id
- an empty scene, or one over 128 characters
- an unknown kind
- a non-zero layer
- a value that does not belong to that kind
- a number that is not finite, or a scale outside the range
- anything past the 256 record cap of that mod.

It keeps the last entry of a repeated mod and prop id. All of that happens **silently**, with no log line. The next write makes the drop permanent, so a hand-edited file that breaks a rule loses that entry for good. A missing file loads empty. A file written by an older framework loads with the fields it has.

The framework writes the file at once, into the slot the game itself would write: `Saves\<character>\<slot>\mods\obenseuer.moddingframework.worldprops.json`. The path is built from the character and slot name, not from the folder the save was loaded from. So a Save As or an autosave into another slot writes there too. A write is skipped while no save is loaded.

Because that write is immediate, a prop you place stays in that slot even if you quit without saving. It stays even though the game's own save is reverted. To take it back, use `Remove` or `Clear`, or delete the file.

## In game checks

1. In `BepInEx\config\obenseuer.moddingframework.cfg` (the framework's [config](../guides/getting-started/config.md) file), set `Debug / WorldPropsSelfCheck` and `Debug / WorldPropsCommand` to `true`. Then start the game and load a throwaway save.
2. Read the [log](../guides/getting-started/logging.md): `world prop self-check ok` is the offline check. Open the console, load a save, and run `core_worldprops_check`. The command does the following, then logs `world prop live self-check ok in <scene>`:
   - It places a primitive, a point light and an anchor with an interaction in the scene you are in. It moves one, removes one, and checks that the prompt hands the prop to the handler.
   - It places a furniture prop and an item prop. It checks that the only `MonoBehaviour` left on each is a `ModInteraction` or none. It checks that `DestructibleList.instance.prefabs` and `Collectible.allCollectibles` did not grow. It checks that the framework's own file written afterwards is byte for byte the file written before.
   - It destroys one prop and restores it, and checks that a restore of a live prop raises no second `Spawned`.
   - It checks that a rigidbody, an audio source and a particle system are settled, and that a destroyed model prop frees its mesh and materials. Then it clears up.
3. Place a prop of each kind by hand, save, quit and reload. Everything returns in place, and the prompt works.
4. Remove one, save and reload. It is gone.
5. To prove the game's save is untouched, save with no props and copy `<save>\<LevelName>.tnmt` aside. Place props, save again and compare the two files: they are the same. The framework's own file `mods\obenseuer.moddingframework.worldprops.json` is the only one that changed.
6. Uninstall the test mod (or change its id) and reload. Nothing shows, there are no errors, and the record is still in the framework's file.

That completes the in-game checks.
