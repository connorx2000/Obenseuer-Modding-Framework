---
title: Common tasks
parent: Quest authoring
grand_parent: Story
nav_order: 1
---

# Common tasks

Short examples for changing a task your mod made with [`QuestAuthoring`](changing-a-task.md). Each one uses the task id for the key `lost_key` of the mod `example.questy`.

Count progress on an objective, then finish the task. The counter of `search` reaches its target of 3, which finishes that objective:

```csharp
string id = QuestAuthoring.TaskId("example.questy", "lost_key");
QuestAuthoring.SetCounter("example.questy", id, "search", 3);
QuestAuthoring.Finish("example.questy", id);
```

Add another step to a task that is still running:

```csharp
string id = QuestAuthoring.TaskId("example.questy", "lost_key");
QuestObjectiveDefinition extra = new QuestObjectiveDefinition { Key = "lock", Title = "Lock the door again" };
bool added = QuestAuthoring.AddObjective("example.questy", id, extra);
```

Mark one objective as done, or end the task as failed:

```csharp
string id = QuestAuthoring.TaskId("example.questy", "lost_key");
QuestAuthoring.SetObjective("example.questy", id, "return", ObjectiveState.Finished);
QuestAuthoring.Fail("example.questy", id);
```
