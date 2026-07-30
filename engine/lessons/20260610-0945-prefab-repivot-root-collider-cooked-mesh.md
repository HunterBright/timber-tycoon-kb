---
title: 'Re-pivoting a prefab: a root MeshCollider with an embedded cooked mesh does NOT follow child-transform shifts'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-06-10'
project: Kerf - Sawmill Tycoon
tags:
- unity
- prefab
- pivot
- meshcollider
- embedded-mesh
- re-pivot
- asset-pipeline
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Re-pivoting a prefab: a root MeshCollider with an embedded cooked mesh does NOT follow child-transform shifts

## Problem
A prefab's visible mesh sat ~8 m from its pivot (FBX export defect). The standard re-pivot fix - add `SHIFT = -T` (T = trunk/anchor base center) to every immediate child's `localPosition` via `LoadPrefabContents` → `SaveAsPrefabAsset` - moves ALL visual geometry onto the pivot, **including geometry whose offset is baked into mesh vertices** (child at localPos 0 still moves because the child transform itself shifts).

BUT: the prefab also had a convex `MeshCollider` **on the prefab root**, whose `sharedMesh` was an **embedded cooked mesh sub-asset stored inside the .prefab file** (created earlier via `AddObjectToAsset`). Its vertices were baked at the same displaced position. The root is never shifted (only children are), so after the child-transform fix the visuals land on the pivot while the **collider stays at the old displaced location** - raycast interaction and physics silently break while everything *looks* fixed.

## Root cause (general rule)
Child-transform re-pivoting only moves things attached to children. Any geometry data referenced by **root-level components** (MeshCollider cooked meshes, particle shapes, custom bounds) keeps its own coordinate space and must be translated separately.

## Fix that worked (verified)
1. Shift all immediate children's `localPosition` by SHIFT; `SaveAsPrefabAsset`.
2. Re-fetch the embedded mesh: `AssetDatabase.LoadAllAssetsAtPath(prefabPath).OfType<Mesh>()`, then `vertices[i] += SHIFT`, `RecalculateBounds()`, `EditorUtility.SetDirty(mesh)`, **`AssetDatabase.SaveAssetIfDirty(mesh)`** (targeted - never `SaveAssets()`, which flushes ALL session-dirty assets and pollutes git scope).
3. `AssetDatabase.ImportAsset(prefabPath, ForceUpdate)` and verify **from the disk reload**: trunk-base T ≈ 0, collider bounds center ≈ 0, visual-center vs collider-center distance ≈ 0.
4. PhysX re-cooks the convex hull from the edited vertices at the next runtime instantiation - no manual re-cook needed for runtime-spawned prefabs.

## Hardening that review caught (apply-script checklist)
- Pre-check ALL targets read-only BEFORE mutating any (batch atomicity).
- Fail **closed** on NaN: `if (!(err <= tol)) abort;` - `if (err > tol)` is false for NaN and lets a broken prefab through.
- Snapshot collider `mesh.vertices`/`triangles` before mutation → self-healing fallback (rebuild + `CreateAsset` + re-point) if the sub-asset is lost.
- No `Undo.RecordObject` for asset edits when the transform half isn't undoable - a half-undoable operation re-splits collider from visuals on Ctrl+Z. Recovery = timestamped file backups.
- Verify external references first: if other assets reference only the prefab ROOT fileID, in-place editing preserves them; nothing re-keys.
