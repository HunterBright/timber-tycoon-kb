---
type: lesson
tags: [unity, fbx, skinnedmesh, bindposes, meta, mixamo, rigging, asset-import]
severity: high
suggested-category: engine/lessons
project: Timber Tycoon
date: 2026-05-31
status: draft
---

# Binary-overwriting an FBX under its existing .meta corrupts skinned-mesh bindposes (T-pose collapse)

## Symptom
A rigged Mixamo character was "updated" by copying new FBX bytes onto an existing asset (`NPC_Male2.fbx`) while keeping its old `.meta`. At runtime the **skinned mesh collapsed to a T-pose / scrambled deformation** even though the **bones animated correctly**. Bone-reading metrics (bone count, transform positions, controller state) looked healthy — so automated checks passed while the character was visibly broken. The failure was only visible by rendering the actual skin, not by inspecting bones.

## Root cause
A model's `.meta` caches the rig/avatar import configuration **keyed to the original mesh's bindposes** (the bind-time inverse bone matrices that map mesh vertices into bone space). When you swap in different geometry under the same `.meta`, Unity reuses the cached/derived bindpose↔bone mapping against a mesh it no longer matches. The skinning math then deforms vertices with the wrong bind matrices → the mesh can't follow the (correctly animating) skeleton and collapses. Transferable insight: **bindposes are part of the imported mesh+rig identity, not a free-floating setting — they cannot survive a geometry swap that bypasses a fresh import.**

## Fix
Re-import the clean source as a **brand-new asset** (new filename → new GUID → fresh `.meta`), never overwriting the old FBX in place:
1. Copy source to a new path (e.g. `NPC_Base_Male1.fbx`).
2. `ModelImporter`: `animationType = Generic`, `avatarSetup = NoAvatar` (for NPCs driven by generic clips), then `SaveAndReimport()`.
3. Build a new prefab; assign material directly on the `SkinnedMeshRenderer` (FBX material remap is unreliable).
4. Re-point references by **name**, not by overwriting GUIDs.

Verification that the import is healthy: `mesh.bindposes.Length == smr.bones.Length`, `smr.rootBone != null`, and — decisively — **render the skin and watch a full stride**; do not trust bone-only metrics.

## Prevention
- **RULE: never binary-overwrite an existing FBX (or any imported asset) while keeping its old `.meta`.** To replace a rigged model, import as a new asset and re-link by name.
- When validating rigs/animation, **judge visually** (render the deforming skin). Bone metrics animate independently of skinning and will mask a bindpose corruption.
- Unity 6 note: `ModelImporter.importMaterials` is obsolete-as-error — using it makes a Coplay/Roslyn `execute_script` fail with an opaque "could not load CSharpResources" message rather than a clean compile error. Prefer `materialImportMode` / omit material flags when not needed.
- Coplay `execute_script` requires a `filePath` to a `.cs` file defining a `public static` entry method (default `Execute`); it does **not** accept inline code. An inline-code call returns a validation error and does nothing — never assume an inline call ran.
