---
title: Discriminating CLIP vs RIG vs SKIN for a one-sided humanoid animation defect
type: pattern
status: draft
confidence: low
verified: ''
date: '2026-05-30'
project: Kerf - Sawmill Tycoon
tags:
- unity
- mixamo
- humanoid-retargeting
- animation
- avatar-calibration
- SampleAnimation
- diagnosis
applies_to: []
source: ''
suggested-category: engine/patterns
---

# Discriminating CLIP vs RIG vs SKIN for a one-sided humanoid animation defect

## Symptom
A Mixamo-rigged NPC played the shared Walk clip with its LEFT foot raised/twisted
differently from the right. Need to know whether the fault is the shared clip
(central, batch-safe fix), the per-character avatar/rig (pipeline fix before
batching N more bases), or Mixamo skin weights (per-base mesh risk).

## The three-way test (all read-only, no scene save)
1. **CLIP vs RIG** — apply the SAME Humanoid clip to two different rigs and read
   BONE local rotations (not the mesh). The cleanest reference is Mixamo's own
   **X Bot**: even when `X Bot@Walking.fbx` is animation-only (0 renderers, no
   mesh), the FBX still imports a valid Humanoid **avatar + skeleton GameObjects**.
   Instantiate that meshless skeleton and sample onto it too.
   - Use `AnimationClip.SampleAnimation(go, t * clip.length)` at normalized
     t = 0/0.25/0.5/0.75. Works in EDIT mode (no Play needed). Set
     `animator.runtimeAnimatorController = null` first so a controller can't
     overwrite the sampled pose. Read `animator.GetBoneTransform(HumanBodyBones.LeftFoot)`
     etc. `.localRotation.eulerAngles`.
   - Healthy gait: `LeftFoot(t) ≈ RightFoot(t+0.5)` with X equal, Y&Z negated
     (mirror + half-cycle phase). If the reference rig is symmetric but the
     suspect rig shows a *constant* L/R offset at every phase → the clip is
     innocent; the suspect rig is the culprit.
2. **AVATAR BIND** — dump the suspect's `ModelImporter.humanDescription.skeleton`
   foot/toe/lowerleg bind rotations. Mirror-symmetric L vs R rules out a gross
   bind error. NOTE: symmetric bind does NOT clear the avatar — the T-pose/muscle
   calibration can still be off in a way the parent-relative bind euler doesn't show.
3. **BONE vs SKIN** — because step 1 reads the BONE transform, an asymmetric bone
   rotation means it is NOT a skin-weight artifact (skin weights would show a bad
   mesh over a correctly-moving bone).

## What the numbers looked like (real case)
- X Bot (reference): both feet in the same pitch range, crossing over through the
  cycle → symmetric.
- NPC_Male2: left-foot pitch ~25–30° greater than the phase-matched right foot at
  EVERY sample → constant one-sided offset. Bind eulers were mirror-symmetric.
- Verdict: **per-character Mixamo avatar/rig calibration** (left-foot muscle/T-pose
  reference), not the clip, not skin weights. Per-base risk for a batch.

## Gotchas
- Euler readouts near ±90° pitch (a planted/lifted foot) make Y/Z explode from
  gimbal — trust the pitch (X) delta, not the absolute Y/Z magnitudes.
- The local-rotation NUMBERS are not comparable BETWEEN two rigs (different bind
  frames). Compare L-vs-R symmetry WITHIN each rig, then compare the two rigs'
  symmetry verdicts.
- A symmetric `humanDescription.skeleton` bind can still retarget asymmetrically if
  the rest/T-pose calibration is off. Fix path: Configure Avatar → Enforce T-Pose,
  or fix the foot orientation in the Blender rest pose before Mixamo.

## Resolution (2026-05-30): it was avatar muscle-AXES, fixed only in Blender
The one-sided foot turned out NOT to be fixable at the Unity avatar layer:
- bone→human mapping, bind rotations, bind-basis matrices, bone positions, and
  muscle limits were ALL mirror-symmetric (~1-2°);
- Unity's **genuine Enforce T-Pose** (invoked via reflection into
  `UnityEditor.AvatarSetupTool` — `GetModelBones`/`GetHumanBones`/`MakePoseValid`/
  `TransferPoseToDescription(SerializedProperty, Transform)` then reimport) ran
  successfully but changed the asymmetry by <0.5°.
- The defect was in the Avatar's INTERNAL per-bone muscle axes, read via reflection
  on `UnityEngine.Avatar` internal instance methods `GetPreRotation(int)` /
  `GetPostRotation(int)` / `GetLimitSign(int)` (arg = `(int)HumanBodyBones`). The
  left/right limit-sign mirror flip was INCONSISTENT down the chain (LowerLeg flips X,
  Foot flips Y, **Toes flip nothing — L and R identical**). Those axes/signs derive
  from each bone's **local ROLL** in the Mixamo auto-rig, which roll does not surface
  in bind position/rotation symmetry.
- Enforce T-Pose fixes the POSE, never the bone ROLL. Muscle limits were all default.
  So there is NO Unity-import lever — the fix is per-base in Blender: Edit Mode →
  select leg/foot/toe bones → Armature ▸ Symmetrize (or recalc roll so L = mirror of R)
  → re-export → re-import → re-run the world-space discriminator (target <~10°).

Key reflection note: the unity-mcp RunCommand sandbox BLOCKS `System.Reflection`, but a
normal COMPILED Editor script in `Assets/Editor` is not sandboxed and can reflect into
internal `AvatarSetupTool` / `Avatar` APIs. That compiled-script route is how to invoke
the real Enforce T-Pose and read muscle axes when an MCP scripting bridge forbids reflection.

## Why transferable
Any Unity project retargeting shared Mixamo/Humanoid clips across multiple
auto-rigged characters can hit one-sided limb defects. This test isolates the
layer (clip / avatar / skin) in minutes, read-only, using the always-present X Bot
avatar as the rig-independent control — before committing a whole batch.
