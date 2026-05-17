---
type: lesson
project: timber-tycoon
suggested-category: engine/lessons
tags: [unity, editor-script, runtime, assemblies, road-tool]
severity: high
time_lost: ""
date: 2026-05-17
status: draft
applies_to: ["unity-projects"]
---

# Runtime vs Editor Script Separation (Assembly Boundary)

## Problem
Editor scripts in the runtime folder cause build errors ("UnityEditor namespace not found"). Runtime scripts in an Editor folder silently fail to attach to GameObjects in standalone builds. "Builds work in editor but break in standalone" is the symptom.

## Root cause
Unity creates a separate `Editor` assembly from any folder named `Editor`. `using UnityEditor;` is only valid in this Editor assembly. Runtime scripts placed in an Editor folder are never included in the build assembly.

## Solution
Strict folder separation for every script:
- **Runtime script** (any MonoBehaviour, component, manager): `Assets/Project/Scripts/`
- **Editor extension** (CustomEditor, EditorWindow, MenuItem, PropertyDrawer): `Assets/Project/Scripts/Editor/`

Example for Road system:
- `Assets/Project/Scripts/Road.cs` — runtime, has waypoints, generates mesh
- `Assets/Project/Scripts/Editor/RoadTool.cs` — CustomEditor, Scene View handles

Runtime code needing optional editor features: wrap in `#if UNITY_EDITOR` directive.

Create the `Editor/` folder if it doesn't exist — Unity will auto-recognize it.

## What didn't work
Mixing scripts in one folder — builds break or scripts silently disappear.

## Transferability
Fundamental Unity project structure rule. Applies to every Unity project. Build CI should fail fast if this rule is violated — add assembly definition files to enforce boundaries.

## Related
- [Editor Scene View input capture](editor-scene-view-input-capture.md)
- [Custom Editor pattern for generators](../patterns/custom-editor-pattern-for-generators.md)
