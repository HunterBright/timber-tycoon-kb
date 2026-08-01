---
title: Crooked foot under Unity Humanoid = auto-rig copied the foot bind pose instead of mirroring it
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-05-31'
project: Kerf - Sawmill Tycoon
tags:
- unity
- humanoid
- rig
- mixamo
- blender
- bind-pose
- retargeting
- npc
- tripo
- auto-rig
applies_to: []
source: ''
severity: medium
promoted: '2026-07-30'
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

Key distinction: the bone **positions** are symmetric (mesh is fine) - only the bone **rotations**
are wrong. So it is a RIG / bind-pose defect, never a mesh geometry problem.

## How to confirm cheaply (read-only, from the .meta - no Unity needed)
For a Humanoid FBX, `humanDescription.skeleton` in the `.fbx.meta` stores per-bone bind transforms.
1. Find the left/right foot bones (`Foot.L`/`Foot.R`, or `mixamorig:LeftFoot`/`RightFoot`).
2. Compare **positions** - they should be mirror-symmetric (proves the mesh/joints are fine).
3. Compare **rotations**: sagittal mirror of a quaternion `(x,y,z,w)` is `(x,-y,-z,w)`.
   Compute `max(|Lx-Rx|, |-Ly-Ry|, |-Lz-Rz|, |Lw-Rw|)`. A clean Mixamo rig is ~0.0004; a broken
   auto-rig is ~0.7. (Note: Generic rigs store an empty skeleton in the meta - you only get this
   data when animationType=Humanoid.)

## Fix that works
Re-rig via **Mixamo** (clean, properly mirrored skeleton) and play it as **Generic** (direct bone
playback, bypasses the Humanoid muscle layer entirely). Trying to fix it while staying Humanoid
(Enforce T-Pose, copy-from-other avatar, roll fixes) did NOT reliably work.

## Gotcha for batch conversion
You CANNOT just flip the broken rigs to Generic and reuse a Generic walk clip: Generic clips match
by bone PATH. A clip baked on `mixamorig:*` paths won't drive a Blender-named (`Foot.L`) skeleton.
Each character must actually get the Mixamo skeleton. Re-rigging also renames bones, which breaks
FBX-variant prefabs - bake a STANDALONE prefab instead (preserve its GUID so references survive).
Both Humanoid and Generic controllers can share the same `isWalking` parameter, so gameplay code
needs no change when swapping controllers.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260725-1015-ai-autorig-proportions-crush-humanoid|Szkielet z auto-rigu AI ma inne proporcje niz siatka: postac w grze skladasie w harmonijke]] - wspolne: retargeting, rig, humanoid
- [[20260531-2000-blender-mesh-only-fbx-for-mixamo|Batch-extract clean mesh-only FBX from rigged .blend for Mixamo re-rig]] - wspolne: auto-rig, tripo, mixamo
- [[20260705-1745-mixamo-motion-only-vs-withskin-retarget|Mixamo "Without Skin" (motion-only) FBX psuje retarget Humanoid - uzyj "With Skin"]] - wspolne: retargeting, humanoid, mixamo
- [[20260726-1420-humanoid-sloty-opcjonalne-vs-wymagane|Humanoid: sloty OPCJONALNE zwracaja null na poprawnym awatarze - fallback po nazwach nie moze byc pod jednym `!isHuman`]] - wspolne: rig, humanoid, mixamo
- [[character-pipeline-tripo-mixamo-unity|Character pipeline: Tripo mesh → Mixamo rig → Unity (clean, working recipe)]] - wspolne: tripo, mixamo, npc
- [[20260531-0934-humanoid-orientation-from-armature-not-bbox|Determine a humanoid's up/forward axis from ARMATURE bone landmarks, not from bounding-box max-spread - a T-pose arm span can beat true height]] - wspolne: humanoid, mixamo, blender
<!-- /POWIAZANE:auto -->
