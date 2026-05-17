---
type: lesson
project: timber-tycoon
suggested-category: engine/lessons
tags: [unity, tags, comparetag, prefabs, runtime-spawning]
severity: medium
time_lost: "~10 min"
date: 2026-05-17
status: draft
applies_to: ["unity-projects"]
---

# Tag Assignment: Code vs Inspector for Runtime-Spawned Objects

## Problem
Runtime-spawned objects don't get detected by `CompareTag` even though the script is correct. Pickup/trigger logic silently fails. "Why isn't my DirtClump being detected?" — it is, but the tag is "Untagged."

## Root cause
Tags assigned via Inspector on a prefab are baked into the prefab asset. When spawned via `Instantiate`, the instance inherits the prefab's tag. If the prefab's tag is "Untagged" (the default), all spawned instances are Untagged regardless of what `CompareTag` expects.

## Solution
Two options:
1. Set tag on the prefab in Inspector (Project view → prefab → Tag dropdown)
2. Assign tag in spawning code: `spawnedObject.tag = "DirtClump";`

Additional pitfall: the tag must exist in Project Settings → Tags before either method works. `CompareTag` with an undefined tag throws an exception; inspector dropdown won't show it.

Best practice: define a static constants class:
```csharp
public static class Tags {
    public const string DirtClump = "DirtClump";
    public const string Log = "Log";
}
```

## What didn't work
Expecting tag to auto-propagate from scene prefab references to spawned instances — it doesn't.

## Transferability
Applies to any Unity project using runtime Instantiate + tag-based detection (combat collision, pickup systems, trigger zones). Standard defensive pattern: after every Instantiate call, verify the tag on the spawned instance if tags are used for detection.

## Related
- [Script overrides prefab Inspector values](../anti-patterns/script-overrides-prefab-inspector-values.md)
