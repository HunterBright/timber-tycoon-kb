---
title: FBX binary-overwrite under a stale .meta corrupts skinned-mesh bindposes (mesh collapses to T-pose while bones animate)
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-05-31'
project: Kerf - Sawmill Tycoon
tags:
- unity
- fbx
- skinned-mesh
- bindposes
- meta
- guid
- mixamo
- rigging
- asset-import
- generic-animation
applies_to: []
source: ''
severity: high
name: fbx-binary-overwrite-corrupts-bindposes
promoted: '2026-07-30'
---

# FBX binary-overwrite under a stale .meta corrupts skinned-mesh bindposes (mesh collapses to T-pose while bones animate)

## Context
We needed to swap a Blender-rigged NPC mesh for a new Mixamo-rigged one **without breaking the references that already pointed at it** - the prefab GUID and the `NPCSpawnConfig` ScriptableObject that listed the prefab. The tempting shortcut: copy the new FBX's BYTES over the existing `NPC_Male2.fbx` and keep the old `NPC_Male2.fbx.meta`, so the asset GUID survives and nothing has to be re-linked. We then reimported and further edited that same `.meta`'s `humanDescription` (Enforce T-Pose, symmetrize attempts) across many passes.

This silently corrupted the mesh's **bindposes** and cost most of a session chasing the wrong layer. This entry is the trap; the discriminator that mislabeled it lives in [Discriminating CLIP vs RIG vs SKIN](../patterns/discriminating-clip-vs-rig-vs-skin-humanoid-defect.md), and the methodology that would have caught it faster lives in [search-first, trust the render, check for upstream sabotage](debugging-search-first-trust-render-check-upstream.md).

## Root cause
A skinned mesh's **bindposes** are the bind-time inverse bone matrices that map mesh vertices into each bone's local space. They are part of the imported *mesh + rig identity*, computed at import from the geometry-vs-skeleton relationship - not a free-floating setting you can carry across a geometry swap.

When you overwrite the FBX bytes under a foreign `.meta`, Unity reuses cached/derived rig configuration (and, across our subsequent `humanDescription` edits, an avatar built for a different rest state) against geometry it no longer matches. The mesh↔skeleton bind matrices desync. The skinning math then deforms vertices with the wrong bind → the mesh cannot follow its (correctly animating) skeleton.

A Humanoid avatar can **mask** this: Humanoid retargeting rebuilds its own bind/pose at runtime, so the mesh appears to deform. Direct/**Generic** skinning uses the raw bindposes and exposes the break.

## Symptom - what the eyes saw vs what the metrics saw
- **Eyes:** the rendered MESH sat at ~bind/T-pose - limbs in the wrong places, parts not following the body - while the character "animated."
- **Metrics:** every bone-space number looked perfect. Bones animated, rotations looped, left/right was symmetric. `bones.Length == bindposes.Length == 28`, all distinct, no nulls, correct `rootBone`. We spent many iterations chasing a "Humanoid muscle-space foot retarget asymmetry" that was real in BONE space but had nothing to do with what was on screen.

**A count/integrity check cannot catch this.** The counts are clean; the corruption is in the bindpose *values*. Only a behavioral test - bones moving while the mesh doesn't follow - reveals it.

## The decisive A/B test (bones vs skin)
1. Put the **same** Animator controller + clips on TWO models: the suspect (overwritten) FBX, and a **pristine "With Skin" import** of the same character (own fresh `.meta`). Identical setup.
2. Read `SkinnedMeshRenderer.bones[]`'s own bone rotations (e.g. `LeftArm`) AND look at the rendered mesh. If the SMR's bones animate (arms down) but the mesh renders a different pose (arms out) → the mesh is **not following its bones** → skinning/bindpose corruption, not animation.
3. Result: the pristine import deformed correctly; the overwritten file rendered a near-T-pose mesh over moving bones, same controller. The fault was in the overwritten FBX's bind, introduced by the byte-overwrite.

## Fix
Treat a skinned FBX as immutable content. To replace a rigged model:
1. Import the NEW source as its **own asset** - new filename → new GUID → fresh `.meta` (e.g. `NPC_Base_Male1.fbx`). Never overwrite bytes under a foreign meta.
2. `ModelImporter`: for NPCs driven by generic clips, `animationType = Generic`, `avatarSetup = NoAvatar`, then `SaveAndReimport()`.
3. Build the prefab fresh; assign the material directly on the `SkinnedMeshRenderer.sharedMaterials` (FBX material remap is unreliable).
4. **Re-point references by NAME, not by preserving a GUID.** Rewire `NPCSpawnConfig` through the name-based Setup scripts instead of trying to keep the old asset GUID alive. The whole motivation for the byte-overwrite (keep references linked) is better served by a deterministic name-based re-link than by a GUID hack that corrupts the mesh.

Healthy-import check: `mesh.bindposes.Length == smr.bones.Length`, `smr.rootBone != null`, and - decisively - **render the skin and watch a full stride.** Do not trust bone-only metrics.

## Meta-lesson
**Bone-space metrics are not what renders. When they disagree, the render is ground truth and the metric is suspect.** Bones animate independently of skinning; a bindpose corruption sails past every bone/count check and only the deforming skin tells the truth. This single confusion drove the whole misdiagnosis (a "foot asymmetry" that was real in bone numbers but invisible to the actual defect).

## Adjacent tooling gotchas (Unity 6 / Coplay MCP)
- `ModelImporter.importMaterials` is obsolete-as-error in Unity 6 - using it makes a Coplay/Roslyn `execute_script` fail with an opaque "could not load CSharpResources" message rather than a clean compile error. Use `materialImportMode` / omit material flags when not needed.
- Coplay `execute_script` requires a `filePath` to a `.cs` file defining a `public static` entry method; it does **not** run inline code. An inline call returns a validation error and does nothing - never assume it ran. (Compiled `Assets/Editor` scripts are also the only route to reflect into internal Unity APIs; the MCP sandbox blocks `System.Reflection` - see [runtime vs editor script separation](runtime-vs-editor-script-separation.md).)

## Related
- [Discriminating CLIP vs RIG vs SKIN for a one-sided humanoid defect](../patterns/discriminating-clip-vs-rig-vs-skin-humanoid-defect.md)
- [Character pipeline: Tripo → Mixamo → Unity (clean recipe)](../../workflow/asset-pipeline/character-pipeline-tripo-mixamo-unity.md)
- [Debugging methodology: search-first, trust the render, check for upstream sabotage](debugging-search-first-trust-render-check-upstream.md)
- [FBX export standard settings (Blender → Unity)](fbx-export-standard-settings-blender-to-unity.md)

## Warianty tego samego wpisu

Ten sam problem zostal zapisany kilka razy. Kopie leza w `_archive/duplicates/` - zajrzyj tam, jesli szukasz obejscia opisanego innymi slowami:

- [[20260530-1800-fbx-binary-overwrite-corrupts-skin-bindposes]]
- [[20260531-0700-fbx-binary-overwrite-corrupts-bindposes]]
