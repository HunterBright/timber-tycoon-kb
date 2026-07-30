---
title: Overwriting an FBX in place preserves prefab-variant references only if object NAMES are unchanged
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-06-06'
project: Kerf - Sawmill Tycoon
tags:
- unity
- fbx
- prefab-variant
- fileid
- reimport
- blender-export
- material-remap
applies_to:
- unity
- blender
source: ''
severity: high
suggested-category: engine/lessons
time_lost: ''
---

# Overwriting an FBX in place preserves prefab-variant references only if object NAMES are unchanged

## Problem
A prop FBX (Pelletizer) was re-exported from Blender with its hierarchy **flattened**
(all parts un-parented to the root, transforms applied to bake world position into
vertices) and re-imported by overwriting the existing `.fbx` bytes in place while keeping
the original `.meta`/GUID. The asset GUID stays valid, but the consuming asset was a
**prefab variant** of the FBX — it references the FBX's *internal sub-object fileIDs*
(`m_CorrespondingSourceObject`) for the root GameObject (which hosts gameplay scripts),
for individual Transforms wired into a controller (arms/rollers/spindle), and for two
sub-meshes used as MeshCollider geometry. If those internal fileIDs change, the prefab's
component wiring and colliders silently detach even though the GUID is intact.

## Root cause
With `fileIdsGeneration: 2` in the ModelImporter, Unity derives each imported sub-object's
fileID from a hash of its **name + type** (with hierarchy used only to disambiguate
duplicate names), NOT from its hierarchy index or parent. Consequence:
- Flattening the hierarchy and applying transforms does **not** change the fileID of an
  object whose **name is unchanged** and which had no name collision.
- The fileID **does** change if the object is renamed, or if flattening creates a new
  name collision that forces disambiguation.
The `100100000` "main model prefab" fileID is always stable, so a prefab variant never
fully detaches from its source — but its internal references can.

## Solution
1. **Checkpoint first** (`git commit`) — the FBX is LFS-tracked; the in-place overwrite is
   trivially reversible only if HEAD holds the old bytes.
2. Overwrite `.fbx` bytes only; never touch the `.meta` (preserves GUID + all asset-path refs).
3. **Verify before trusting**: after reimport, dump the new FBX's internal fileID→name
   table via `AssetDatabase.TryGetGUIDAndLocalFileIdentifier` on every GameObject /
   Transform / Mesh, and cross-check it against the exact list of fileIDs the prefab
   references (read them out of the `.prefab` YAML's `m_CorrespondingSourceObject` and
   `m_Mesh` entries beforehand). Report PRESENT / MISSING per id.
   In this case all 13 required fileIDs survived because the Blender export kept the part
   names (`Pelletizer_Body`, `Pelletizer_Arm_1`, …) identical.

## What didn't work
- **Trusting the GUID alone.** GUID preservation guarantees asset-*path* references resolve;
  it says nothing about internal sub-object fileIDs that prefab variants depend on.
- **`ModelImporter.AddRemap` to bind the single baked material.** Even with
  `materialImportMode = None` and `remapMaterialsIfMaterialImportModeIsNone = true`, the
  remap did not bind — all 29 renderer slots fell back to the default `Lit` material
  (model rendered flat grey). The reliable fix was assigning `sharedMaterials` directly on
  the prefab's MeshRenderers (`PrefabUtility.LoadPrefabContents` → set → `SaveAsPrefabAsset`);
  a prefab-connected scene instance then inherits it with no scene save needed.

## Transferability
Applies to any Unity project that wraps an imported model in a **prefab variant** (or a
prefab whose MeshColliders/script refs point at FBX sub-meshes/transforms) and later
re-exports that model from Blender/Maya/etc. The safe re-export rule: **keep object names
stable** across exports, and if you must flatten/apply-transforms, verify the internal
fileID table after import rather than assuming the GUID covers you. The fileID-dump +
required-id cross-check is a reusable pre-flight for any in-place model overwrite.

## Related
- feedback_fbx_importer_remap_unreliable (project memory) — AddRemap binding unreliability
- feedback_no_fbx_binary_overwrite (project memory) — the opposite case: SKINNED meshes /
  armatures must NOT be overwritten in place (collapses bindposes); this lesson is the
  STATIC-prop counterpart where in-place overwrite IS safe
- feedback_bake_space_transform_empties (project memory) — related Blender→Unity export quirk
