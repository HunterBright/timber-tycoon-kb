---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [unity, humanoid, rig, mixamo, blender, bind-pose, retargeting, npc, tripo, auto-rig]
date: 2026-05-31
status: draft
severity: medium
---

# Crooked foot under Unity Humanoid = auto-rig copied the foot bind pose instead of mirroring it

## Symptom
Auto-rigged characters (Tripo mesh → Blender/auto-rig → Unity Humanoid) show a visibly
crooked/twisted LEFT foot during retargeted walk animation. Looks like a model defect; it isn't.

## Root cause
The auto-rigger wrote the **left foot's bind-pose rotation onto the right foot without mirroring it**
(sometimes the two feet quaternions are byte-for-byte identical). A correct biped needs the right
foot's bind rotation to be the sagittal mirror of the left (Y and Z components sign-flipped). Because
the feet are *copied* not *mirrored*, and both carry an off-axis tilt, Unity's Humanoid muscle
retargeting twists one foot.

Key distinction: the bone **positions** are symmetric (mesh is fine) — only the bone **rotations**
are wrong. So it is a RIG / bind-pose defect, never a mesh geometry problem.

## How to confirm cheaply (read-only, from the .meta — no Unity needed)
For a Humanoid FBX, `humanDescription.skeleton` in the `.fbx.meta` stores per-bone bind transforms.
1. Find the left/right foot bones (`Foot.L`/`Foot.R`, or `mixamorig:LeftFoot`/`RightFoot`).
2. Compare **positions** — they should be mirror-symmetric (proves the mesh/joints are fine).
3. Compare **rotations**: sagittal mirror of a quaternion `(x,y,z,w)` is `(x,-y,-z,w)`.
   Compute `max(|Lx-Rx|, |-Ly-Ry|, |-Lz-Rz|, |Lw-Rw|)`. A clean Mixamo rig is ~0.0004; a broken
   auto-rig is ~0.7. (Note: Generic rigs store an empty skeleton in the meta — you only get this
   data when animationType=Humanoid.)

## Fix that works
Re-rig via **Mixamo** (clean, properly mirrored skeleton) and play it as **Generic** (direct bone
playback, bypasses the Humanoid muscle layer entirely). Trying to fix it while staying Humanoid
(Enforce T-Pose, copy-from-other avatar, roll fixes) did NOT reliably work.

## Gotcha for batch conversion
You CANNOT just flip the broken rigs to Generic and reuse a Generic walk clip: Generic clips match
by bone PATH. A clip baked on `mixamorig:*` paths won't drive a Blender-named (`Foot.L`) skeleton.
Each character must actually get the Mixamo skeleton. Re-rigging also renames bones, which breaks
FBX-variant prefabs — bake a STANDALONE prefab instead (preserve its GUID so references survive).
Both Humanoid and Generic controllers can share the same `isWalking` parameter, so gameplay code
needs no change when swapping controllers.
