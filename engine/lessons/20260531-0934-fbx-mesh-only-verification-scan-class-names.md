---
title: 'Verifying an FBX is "mesh-only" before Mixamo: scan for the real CLASS names, not substrings - `AnimStack` matches the header property `ActiveAnimStackName`'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-05-31'
project: Kerf - Sawmill Tycoon
tags:
- fbx
- blender-export
- mixamo
- verification
- binary-scan
- false-positive
- rigging
applies_to:
- blender
- fbx
- mixamo
- unity
source: ''
severity: medium
time_lost: ~5 min (caught a false positive before reporting)
promoted: '2026-07-30'
---

# Verifying an FBX is "mesh-only" before Mixamo: scan for the real CLASS names, not substrings - `AnimStack` matches the header property `ActiveAnimStackName`

## Problem
To send a static mesh to Mixamo for a fresh auto-rig, the FBX must contain NO skeleton/skin/anim (otherwise Mixamo tries to "map existing skeleton" and errors). I verified the exported file by scanning its bytes for rig tokens. A naive scan for `b'AnimStack'` returned **1 hit** and falsely flagged the file as "not mesh-only" - even though it had zero bones and zero skin.

## Root cause
Binary FBX stores a GlobalSettings/header **property literally named `ActiveAnimStackName`** (an empty `KString`). A substring search for `AnimStack` matches inside `ActiveAnimStackName`. It is metadata, not an animation object. Blender also always writes an **empty `Takes` container** (1 hit) even with `bake_anim=False`.

## Solution
Scan for the actual FBX **class names** and treat these as the decisive set:
- Skeleton present? → `LimbNode`, `Skeleton`, `NodeAttribute` (skeleton subtype), `Bone`
- Skinning present? → `Deformer`, `Skin`, `Cluster`, `Weight`, `BindPose`, `Pose`
- Animation present? → `AnimationStack`, `AnimationLayer`, `CurveNode`, `AnimCurve`, `Take 001`

A genuinely mesh-only Blender export reads **all of the above = 0**, while `Geometry`/`Mesh` are present. The benign leftovers that are NOT rig data: `ActiveAnimStackName` (header property) and an empty `Takes` block. Do not match the bare substring `AnimStack` - match `AnimationStack`.

Blender export args that produce a clean static FBX for Mixamo: `object_types={'MESH'}`, `use_selection=True`, `bake_anim=False`, `axis_up='Y'`, `axis_forward='-Z'`, `apply_scale_options='FBX_SCALE_ALL'`, `mesh_smooth_type='OFF'`. Strip first: remove Armature modifier, `vertex_groups.clear()`, `parent_clear(type='CLEAR_KEEP_TRANSFORM')`, then `transform_apply`.

## What didn't work
- Substring scan for `AnimStack` → false positive on `ActiveAnimStackName`.
- Treating the empty `Takes` block as evidence of animation.

## Transferability
Any Blender→FBX→Mixamo (or →Unity) verification workflow. The class-name-vs-substring distinction generalises to inspecting any binary FBX without importing it (useful when re-importing would pollute / disconnect a live MCP scene).

## Related
- [[20260531-0934-tripo-polygon-soup-inverted-winding-fix]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[fbx-binary-overwrite-corrupts-bindposes|FBX binary-overwrite under a stale .meta corrupts skinned-mesh bindposes (mesh collapses to T-pose while bones animate)]] - wspolne: rigging, mixamo, fbx
- [[20260531-2000-blender-mesh-only-fbx-for-mixamo|Batch-extract clean mesh-only FBX from rigged .blend for Mixamo re-rig]] - wspolne: rigging, mixamo, fbx
- [[20260606-1515-fbx-inplace-overwrite-fileid-preservation|Overwriting an FBX in place preserves prefab-variant references only if object NAMES are unchanged]] - wspolne: blender-export, fbx
- [[20260717-0010-generated-rig-bone-axis-defect-skeleton-transplant|Rigi z generatorów AI (Hunyuan): osie kości rozjechane z frontem modelu = wykrzywiona stopa w retargecie; lek = przeszczep szkieletu w Blenderze]] - wspolne: rigging, mixamo
- [[20260713-1030-verify-in-target-engine-not-source-tool|Weryfikuj asset w silniku DOCELOWYM, nie w narzędziu źródłowym]] - wspolne: verification, fbx
- [[20260705-1745-mixamo-motion-only-vs-withskin-retarget|Mixamo "Without Skin" (motion-only) FBX psuje retarget Humanoid - uzyj "With Skin"]] - wspolne: mixamo, fbx
<!-- /POWIAZANE:auto -->
