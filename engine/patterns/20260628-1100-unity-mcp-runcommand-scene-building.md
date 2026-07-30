---
title: Procedural Unity scene-building via unity-mcp RunCommand
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-06-28'
project: Kerf - Sawmill Tycoon
tags:
- unity
- unity-mcp
- runcommand
- procedural-placement
- editor-scripting
- physics-layers
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Procedural Unity scene-building via unity-mcp RunCommand

## When to use
Placing/building many objects in a Unity scene programmatically - especially when the live scene is BINARY (can't text-edit/grep) and you want a tight "place → screenshot → adjust" loop. Drive it through `Unity_RunCommand` (compiles + executes C# in the editor) + `Unity_Camera_Capture` (set `SceneView.lastActiveSceneView` pivot/rotation/size, then capture).

## The loop
Write a self-contained **idempotent** C# snippet: find+destroy a named root (`GameObject.Find` → `result.DestroyObject`), recreate it, place children (raycast the terrain collider for ground Y, assign a shared material, add a BoxCollider sized from combined renderer bounds, set rotation last so a box built at identity stays tight), then frame the scene view + capture. Re-run freely to iterate.

## Gotchas (Unity 6.x + this MCP) - these cost real time
- Class MUST be `internal class CommandScript : IRunCommand`. `public` → "inconsistent accessibility"; wrong name → NullRef.
- `System.Reflection` namespace is BLOCKED ("unauthorized namespace"). Don't reflect - call project statics directly.
- `HashSet<>` / `ISet<>` fail to compile (assembly not referenced). Use `List<T>` + `Contains`.
- `GetInstanceID()` / `GetEntityId()` = CS0619 hard error. Avoid; frame views by coordinates, not instance IDs.
- Qualify `UnityEngine.Mesh` (ambiguous otherwise).
- Calling a project static works directly: `BuildWaterSurface.Execute()`, `DecorFloraScatter.ScatterFull()`. But `Type.GetType("Name")` WITHOUT an assembly qualifier returns null (so a `present?` check via Type.GetType is misleading).
- MCP drops with "Connection revoked" after an asset refresh/reimport → user must re-approve in Project Settings → AI → Unity MCP. Plan for it.
- For C# **editor scripts** placing FBX: `PrefabUtility.InstantiatePrefab(AssetDatabase.LoadAssetAtPath<GameObject>(path))`.

## Selective physics barrier (block player, pass NPC vehicles)
Need an invisible wall the player can't cross but NPC cars drive through (they follow waypoints): put the walls on the `Boundary` layer (player Default collides with it) and `Physics.IgnoreLayerCollision(npcVehicleLayer, boundaryLayer, true)`. Persist it across editor restarts/builds with a tiny committed `[RuntimeInitializeOnLoadMethod(BeforeSceneLoad)]` setter - the edit-mode call alone may not survive a restart.

## Terrain extension without touching the tracked terrain FBX
To add ground (e.g. extend a shrunk map to meet backdrop mountains) without re-exporting/risking the tracked terrain mesh + its derived map-boundary + NavMesh: build a SEPARATE draped grass patch (grid mesh, raycast the existing edge height per column, the terrain's own material, grass vertex-color, MeshCollider). Seamless + reversible + isolated.
