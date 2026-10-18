---
title: Route
parent: Drones
grand_parent: Objects and devices
nav_order: 5
---

# Route

`SetTarget` and `SetMaxSpeed` change how a drone moves: where it heads next and how fast it may fly. The node ids come from [`Drones.Nodes()`](air-nodes.md).

## Things to know

- `SetTarget(id, nodeId)` points a drone at an air node but does not start it. A drone that is off keeps the new target until you call [`SetActive(id, true)`](reading-and-controlling-drones.md). It returns `true` when the drone's target is now that node. It returns `false` for an unknown drone, an unknown node id, an empty id, or while a save is loading or being written. <span class="tech">(It finds the drone and the node, then calls `Drone.SetTarget`, which only assigns `targetNode`.)</span>
- `SetMaxSpeed(id, maxSpeed)` sets how fast the drone may fly, and the value must be above 0. A zero speed would break the drone's movement, so it is always refused. <span class="tech">(It calls `Drone.SetMaxSpeed`, which assigns the public `maxSpeed`. A zero would divide by zero in `Update`.)</span>
- `SetMaxSpeed` returns `false` in these cases:
  - The value is zero, negative, not a real number, or infinite.
  - The drone is unknown.
  - A save is loading or being written.
- The target is saved. The game saves a drone's target as the node's save id, and finds the node again when the save loads. A node with no save id is saved as nothing and loads with no target. <span class="tech">(The target is saved through `targetNodeRef`, which stores `Air_Node.GUID` and resolves it on load through `Air_Node.GetNode`.)</span>

## Limits

- You can't clear a drone's target. Passing a null node is refused. A drone with no target switches itself off on its next update. So clearing would stop the drone as a side effect, and the framework does not start that. <span class="tech">(With `deActivateOnPathEnd` set (the default), a null `targetNode` turns the drone off in `Update`.)</span>
- A target node can override your `SetMaxSpeed` value: if the node has its own top speed above 0, that one wins.
- The top speed should be saved too, but that has not been confirmed. <span class="tech">(`maxSpeed` is a public field without `[fsIgnore]`, so the save code is expected to keep it.)</span>

**Common problems**

- The drone does not fly at the speed you set: its target node has its own top speed above 0, and that one wins.
