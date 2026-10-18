---
title: Common tasks
parent: Drones
grand_parent: Objects and devices
nav_order: 1
---

# Common tasks

Short snippets for the most common drone jobs: start one, send it to a node, set its top speed and read where it is.

Switch a drone on with [`SetActive`](reading-and-controlling-drones.md). It returns true when the drone is in the state you asked for.

```csharp
if (Drones.SetActive("some-drone-id", true)) Log.Info("the drone is running");
```

Send a drone to an air node and then start it. A drone that is off keeps its new target until you start it. [Route](route.md) has the rules for `SetTarget`.

```csharp
if (Drones.SetTarget("some-drone-id", "some-node-id")) Drones.SetActive("some-drone-id", true);
```

Set a drone's top speed. The value must be above 0.

```csharp
Drones.SetMaxSpeed("some-drone-id", 5f);
```

Read where a drone is and where it is heading.

```csharp
DroneInfo drone = Drones.Get("some-drone-id");
if (drone != null) Log.Info(drone.Position + " heading for " + drone.TargetNodeId + ", then " + drone.NextNodeId);
```

List the [air nodes](air-nodes.md) and the nodes each one links to.

```csharp
foreach (AirNodeInfo node in Drones.Nodes())
    Log.Info(node.Id + " links to " + node.LinkedNodeIds.Count + " nodes");
```
