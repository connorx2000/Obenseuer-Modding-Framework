---
title: Models
parent: Registry
grand_parent: Items and crafting
nav_order: 5
---

# Models

This page covers `ModelRegistry` and the types it uses to load your own 3D models. You need it when an item should look different from the item it copies, or when you build meshes yourself.

This page uses `Vector3`, `GameObject`, `Mesh` from Unity. The [glossary](../../glossary.md#types-that-come-from-unity) says what each one is.
{: .unity-types}

Status: experimental.

A model is a 3D shape for an item. It is loaded from an OBJ file (a common 3D model file). An optional MTL file can sit next to it (the file that holds its colours and texture names). You load a model once, and then use the handle you get back for items. With a handle you can:

- give a new item a model;
- replace the model an item shows in the world;
- build meshes yourself.

<div class="api-type" markdown="1">
<p class="api-type-head"><code>ModelRegistry</code> <span>loads, finds and swaps models</span></p>

`LoadObj` loads a model, once. `Get` finds one you loaded earlier. `OverrideVanilla` swaps the model an item shows in the world.

<div class="api-pair" markdown="1">

```csharp
public static class ModelRegistry
{
    public static bool IsAvailable { get; }
    public static ModelHandle LoadObj(string modGuid, string key, string objPath, string texturePath, ModelOptions options);
    public static ModelHandle Get(string modGuid, string key);
    public static bool OverrideVanilla(int itemId, ModelHandle world, ModelHandle held);
}
```

| Method | Returns | What it does |
|---|---|---|
| `LoadObj(modGuid, key, objPath, texturePath, options)` | `ModelHandle` | Loads an OBJ model. |
| `Get(modGuid, key)` | `ModelHandle` | Finds a model you loaded earlier. |
| `OverrideVanilla(itemId, world, held)` | `bool` | Replaces the model an item shows in the world. |

| Property | Type | What it means |
|---|---|---|
| `IsAvailable` | `bool` | `false` when the model changes could not be set up. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>ModelHandle</code> <span>a model you have loaded</span></p>

`LoadObj` gives you one of these, and [`ItemDefinition.Model`](items.md) takes it. Check `IsReady` before you use it: a bad file gives a handle that is not ready.

<div class="api-pair" markdown="1">

```csharp
public sealed class ModelHandle
{
    public string Key { get; }
    public bool IsReady { get; }
    public string Error { get; }
    public Bounds Bounds { get; }
    public Mesh CreateMesh();
    public Material[] CreateMaterials(Material source);
}
```

| Method | Returns | What it does |
|---|---|---|
| `CreateMesh()` | `Mesh` | Builds a new mesh from the model. |
| `CreateMaterials(source)` | `Material[]` | Builds the materials for the model. |

| Property | Type | What it means |
|---|---|---|
| `Key` | `string` | The key you loaded the model with. |
| `IsReady` | `bool` | `true` when the model loaded without a problem. |
| `Error` | `string` | What went wrong. `null` when nothing did. |
| `Bounds` | `Bounds` | The size of the model. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>ModelOptions</code> <span>how to size and place the model</span></p>

Pass one of these to `LoadObj` to size, turn, place and collide the model. Every field has a default, so set only the ones you need.

<div class="api-pair" markdown="1">

```csharp
public sealed class ModelOptions
{
    public float Scale = 1f;
    public bool FitToTemplate = true;
    public ModelPivot Pivot = ModelPivot.Keep;
    public Vector3 EulerOffset;
    public Vector3 PositionOffset;
    public bool FlipX = true;
    public ModelCollider Collider = ModelCollider.Box;
}
```

| Property | Type | What it means |
|---|---|---|
| `Scale` | `float` | Makes the model bigger or smaller. It must be a real number above 0 and at most 1000. |
| `FitToTemplate` | `bool` | `true` resizes the model to match the size of the [template](../../glossary.md#template) (the item copied). |
| `Pivot` | `ModelPivot` | Where the centre point is. |
| `EulerOffset` | `Vector3` | Turns the model by these angles, in degrees. |
| `PositionOffset` | `Vector3` | Moves the model by this much. |
| `FlipX` | `bool` | `true` mirrors the model along the x axis, because OBJ files and the game use opposite hands. |
| `Collider` | `ModelCollider` | The touch shape. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>ModelPivot</code> <span>where the model's centre point is placed</span></p>

This is the value of `ModelOptions.Pivot`. It sets where the model's centre point is placed.

<div class="api-pair" markdown="1">

```csharp
public enum ModelPivot
{
    Keep,
    BottomCenter,
    Center
}
```

| Value | Meaning |
|---|---|
| `Keep` | Keeps the pivot of the OBJ file. |
| `BottomCenter` | The middle of the bottom of the model. |
| `Center` | The middle of the model. |

</div>
</div>

<div class="api-type" markdown="1">
<p class="api-type-head"><code>ModelCollider</code> <span>the shape the game uses to find out what you touch or pick up</span></p>

This is the value of `ModelOptions.Collider`. It chooses the touch shape of the item.

<div class="api-pair" markdown="1">

```csharp
public enum ModelCollider
{
    Box,
    ConvexMesh,
    Template
}
```

| Value | Meaning |
|---|---|
| `Box` | A simple box around the model. |
| `ConvexMesh` | A convex shape that follows the model. |
| `Template` | The template item's own shape. |

</div>
</div>

## Things to know

- `LoadObj` reads the file at once. It does not throw an error for a bad file. Instead you get a handle where `IsReady` is `false` and `Error` says what is wrong. The error is also written to the [log](../../guides/getting-started/logging.md). Only a bad `modGuid` or `key` throws an error. `key` is 1 to 64 characters of letters, digits, `.`, `_` or `-`.
- A handle is not ready when:
  - the path is not a full path;
  - `Scale` is not a real number above 0, or is above 1000;
  - an angle or offset is not a real number;
  - the file can't be read;
  - a file is over one of the limits.
- Loading the same [mod id](../../glossary.md#mod-id) and key again gives you back the first handle if it is ready. The key is compared letter for letter, so `Desk` and `desk` are two different models. A handle that failed is read again on the next call.
- `objPath` and `texturePath` must be full paths. The game reads the MTL file from the same folder as the OBJ file, and every texture it names must stay inside that folder. A `texturePath` you pass replaces the textures named in the MTL file on every part of the model. Pass `null` to use the MTL file's textures. <span class="tech">(The `mtllib` file is read from the OBJ folder, and every `map_Kd` must stay inside it. A `map_Kd` file must be `.png`, `.jpg` or `.jpeg`. No part of its path may be a Windows device name (CON, PRN, AUX, NUL, COM or LPT followed by a digit) before its first dot, ignoring trailing spaces, so `CON.x.png` and `con .png` are refused, and it must not contain `:`. A `map_Kd` line may start with the MTL options (`-s`, `-o`, `-clamp on` and so on, with their standard argument counts). An unknown option takes up to three following numbers (not `on` or `off`), so `-foo 3 x.png` names `x.png`. The file name is the rest of the line, so it may contain spaces.)</span>
- `ItemDefinition.Model` takes a handle from `LoadObj`. `null` keeps the model of the [template](../../glossary.md#template). Any other handle throws an `ArgumentException`. A handle that is not ready writes an error to the log, and the template's look is kept.
- `FitToTemplate` (on by default) resizes your model so its longest side matches the longest side of the template's model. A 27 cm model on a 10 cm template is shrunk to 10 cm. `Scale` then multiplies the result. Turn `FitToTemplate` off to keep the model's own size. A model whose scaled size is not a real number is refused. <span class="tech">(The fit is measured on the template's primary mesh through the transforms below its root. The root's own scale is not part of the fit. It stays on the item, so the model is drawn at the template's full size.)</span>
- The way the model faces comes from `EulerOffset` only. A template that lies on its side does not make your model lie down. The template's own turn is reset when the model is built. <span class="tech">(The template's root rotation is reset in `ModelBuild.Swap`.)</span>
- Colours from the MTL file are converted so they look right in the game. The colour numbers in an MTL file depend on the program that wrote it (Blender writes them in linear form). Check the colours of your own model in the game. <span class="tech">(`Kd` in the `.mtl` is converted from linear to sRGB for the material colour (`ModelBuild.Channel`) in every project. This assumes `Material` colours take sRGB values in both colour spaces.)</span>
- `OverrideVanilla(itemId, world, null)` replaces the model an item shows in the world: dropped, on a shelf or in storage. It works for any item, from the game or from a mod. A second call for the same item with a different handle writes a warning to the log and replaces the first. Nothing is saved, so removing the mod brings back the template's look.
- The model an item shows in the hand is set with `ItemRegistry.SetHeldModel`, see the [items page](../items.md).
- A dropped mod item is saved as its template's item, so a save never holds a broken model path. With your mod installed, your model is applied again when the save loads. Without it, the item shows as the template's prop holding the coin. <span class="tech">(A mod item's world drop saves the template's vanilla prefab path, so a save never holds `ERROR` or an unresolvable path. Without the mod, the code path gives the template prop holding the coin item (catalogue ID 0, "Open Sewer Coin").)</span>
- Other modded items that are dropped keep their own model path when that path loads. If it does not load, a save can hold `Prefabs/Items/ERROR` for the item. When the save loads, the framework gives such an item its model from the item list, so it does not turn into the coin. Without the framework, a save holding `ERROR` loads as the coin. <span class="tech">(The framework's spawn postfix writes the item's own `PrefabPathMany` (a stack over 1) or `PrefabPath` when that path loads, and leaves the path alone otherwise. On load, `FloorItems.Resolve` gives such an item its catalogue model.)</span>
- Icons are separate from models. See `ItemDefinition.IconPath` and `ItemIcons.OverrideVanilla` in [Item icons](item-icons.md#item-icons).
- `CreateMesh()` builds a new mesh from the OBJ file. It applies `Scale`, `EulerOffset`, `PositionOffset`, `Pivot` and `FlipX`, but not `FitToTemplate`. So the mesh has the model's own size times `Scale`. Each call builds a new mesh. You own it, so call `Object.Destroy` on it when you are done.
- `CreateMaterials(Material source)` builds one new material for each part of the model. Each is copied from `source`, with the colour and texture from the MTL file. You own the materials, so call `Object.Destroy` on each one when you are done. You do not own the textures. They are kept for the whole session and shared by every call and by item models, so do not destroy them. A texture refused by the limits keeps the material's plain colour.

This clones a prop and gives it the model's mesh.

```csharp
GameObject clone = UnityEngine.Object.Instantiate(prop);
MeshFilter filter = clone.GetComponent<MeshFilter>();
Mesh mesh = handle.CreateMesh();
if (mesh != null) filter.sharedMesh = mesh;
```

## Limits

- `CreateMesh()` returns `null` when:
  - the handle is not ready;
  - you did not call it from your mod's normal code;
  - the framework has not started yet. Declare `[BepInDependency("obenseuer.moddingframework")]` so the framework loads first.
- Your mod's normal code runs on the [main thread](../../glossary.md#main-thread) (the one thread the game uses). The first time a `CreateMesh()` call returns `null` because of the thread, error lines and a warning go to the log. That tells it apart from a handle that is not ready.
- `CreateMaterials(Material source)` returns `null` when:
  - the handle is not ready;
  - `source` is `null`;
  - you did not call it from your mod's normal code;
  - a material fails to build (the ones already built are destroyed).
- As with `CreateMesh`, the first thread-related `null` from `CreateMaterials` writes one warning.
- `ItemDefinition.Model` and `OverrideVanilla` need a handle that is ready. `OverrideVanilla` returns `false` when:
  - the model changes could not be set up;
  - the handle is not ready;
  - the game's item list is built and has no item with that number and a picture.
- `OverrideVanilla` throws an `ArgumentException` if you pass a held model.
- There are size limits, to keep a bad file from freezing the game:
  - The OBJ file may be at most 64 MB, with at most 2,000,000 points and 2,000,000 triangles.
  - It may have at most 8 parts that have their own material. Extra parts are merged into the last one.
  - An MTL file over 1 MB is ignored with a warning.
  - A texture over 2048 pixels on a side is refused, and the part keeps its plain colour.
  - A texture or icon file over 16 MB is not read.
  - A face with more than 4 corners is cut into triangles, with a warning. <span class="tech">(The limits are 2,000,000 vertices, 6,000,000 indices (2,000,000 triangles) and 8 sub-meshes (extra `usemtl` groups merge into the last). An image is refused from its header before it is decoded, when its first bytes are not a PNG or JPEG signature, when a PNG or JPEG is over the side limit, or when a JPEG has no frame header (SOF) before its end. The JPEG scan skips bytes between segments, as libjpeg does, so stray bytes cannot hide a large frame header.)</span>
- A convex touch shape over 255 triangles becomes a box, with a warning. A [template](../../glossary.md#template) whose root has no collectible item component is refused. <span class="tech">(`ModelCollider.ConvexMesh` above 255 triangles becomes `Box`. The component is `CollectibleItem`.)</span>

**Common problems**

- `IsReady` is `false`: one of the cases in the "A handle is not ready" bullet under Things to know applies. `Error` says what is wrong, and the error is also in the log. Fix the file or the value and call `LoadObj` again, because a handle that failed is read again on the next call.
- `CreateMesh()` returns `null`: the handle is not ready, the call did not come from your mod's normal code, or the framework has not started yet. Call it from your mod's normal code, and declare `[BepInDependency("obenseuer.moddingframework")]` so the framework loads first.
- `OverrideVanilla` throws an `ArgumentException`: you passed a held model. Pass `null` for `held`. The model in the hand is set with `ItemRegistry.SetHeldModel`.

## Implementation notes

- To replace the icon of an item from the game, use `ItemIcons.OverrideVanilla(int, string)`.
- The framework builds your model from a copy of the prop of the [template](../../glossary.md#template). It reads that copy straight from the game's files, not through the item's `Prefab` property. For an overridden item, that property would build the clone from itself. <span class="tech">(It uses `Resources.Load` on the vanilla path, not `Item.Prefab`, which runs `Loadprefab` again.)</span>
- A mod item keeps the template's original model path. <span class="tech">(It keeps the template's vanilla `PrefabPath`.)</span>
- Parts of the template that would get in the way are switched off or removed. The template's other mesh renderers are disabled, not their objects. The level-of-detail group, the skin component and the collectible item colours are removed on every model, not only on textured ones. <span class="tech">(`LODGroup`, `ItemSkin` and `CollectibleItemColors` are removed with `DestroyImmediate`.)</span>
- The touch shape is rebuilt on the root object, and any shape already on the root is removed. When you pick something up, the game reads only the hit shape's own object. So a shape on a child object could not pick the item up.
- `OverrideVanilla` does not wait while a save is loading or being written. It only records the model and clears two cached prefab fields, which the game loads again. The game's own steps that use the model run during a load or a save too. Skipping one would leave the template's look or save path. <span class="tech">(`OverrideVanilla` does not check `SaveData.Busy`. It only fills the `worlds` dictionary and clears two cached prefab fields, which the getter reloads. `Loadprefab`, `DestructibleList.SpawnItem`, `CollectibleItem.Start` and the catalogue step do not check it either.)</span>
- The first copy of each template writes one `template ...` line to the log. <span class="tech">(It comes from `ModelBuild.Diagnose`.)</span>
