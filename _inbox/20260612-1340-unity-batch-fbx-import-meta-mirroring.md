---
type: pattern
project: Timber_Tycoon
suggested-category: engine/patterns
tags: [unity, fbx, import, meta, prefab, batch, editor-scripting, mixamo]
date: 2026-06-12
status: draft
---

# Batch FBX import with pre-authored .meta files + prefab build in temp additive scene

## Problem
Importing N character/prop FBX files that must all share identical, non-default importer settings (rig type, avatar mode, scale), then building N identical prefabs from them — without touching the open scene and without clicking through the Inspector N times.

## Pattern (validated 2026-06-12, 11/11 NPC models, zero console warnings)

1. **Pre-author .meta files instead of configuring importers afterwards.** Take the .meta of an existing, known-good "benchmark" asset, replace only the `guid:` line with a fresh `[guid]::NewGuid().ToString('N')`, and write it next to the copied FBX *before* Unity sees the file. On the next `AssetDatabase.Refresh()` Unity imports with exactly the benchmark settings on the first pass — no reimport churn, no per-field scripting of ModelImporter. Write meta as UTF-8 **without BOM**.
2. **Hard gate before building anything**: load each imported FBX as GameObject and compare full bone *paths* (not just names) against the benchmark via `GetComponentsInChildren<Transform>`. Path comparison catches wrapper nodes (e.g. Blender's "Armature") that would silently break Generic clip binding even though all bone names exist. Also check: Avatar asset `isValid && !isHuman`, SkinnedMeshRenderer `bones.Length > 0`. Extra finger bones beyond the benchmark set are harmless; missing paths are fatal.
3. **Build prefabs in a temporary additive scene**, never in the open scene: `EditorSceneManager.NewScene(EmptyScene, Additive)` → `PrefabUtility.InstantiatePrefab(asset, tempScene)` → `UnpackPrefabInstance(Completely)` → add/configure components → `SaveAsPrefabAsset` → `DestroyImmediate` → `CloseScene(removeScene: true)` in a `finally`. The user's scene is never dirtied or saved.
4. Register results in ScriptableObjects via `SerializedObject`/`FindProperty` + `ApplyModifiedPropertiesWithoutUndo` + `AssetDatabase.SaveAssets()` (saves assets only, not scenes).

## Why
The .meta-first approach is deterministic and diff-able; scripting ModelImporter fields after import is the fragile alternative (API churn between Unity versions, double import). The temp-scene build is the only way to use PrefabUtility (which requires scene instances) while guaranteeing the live scene file stays byte-identical.

## Caveat
`AnimationClip.apparentSpeed`/`averageSpeed` are 0 for in-place clips — authored locomotion speed is NOT recoverable from clip metadata; plan an empirical measurement (foot-bone displacement) if you need to sync agent speed to animation.
