---
title: Never binary-overwrite a skinned FBX under an existing .meta — it desyncs bindposes
type: anti-pattern
status: draft
confidence: low
verified: ''
date: '2026-05-30'
project: Kerf - Sawmill Tycoon
tags:
- unity
- fbx
- skinned-mesh
- bindposes
- mixamo
- generic-animation
- humanoid
- diagnosis
applies_to: []
source: ''
suggested-category: engine/anti-patterns
---

# Never binary-overwrite a skinned FBX under an existing .meta — it desyncs bindposes

## What we did wrong
To "keep the prefab GUID", we copied a new rigged Mixamo FBX's BYTES over an existing
`NPC_Male2.fbx` while keeping the old `.meta`/GUID, then reimported and further edited
that meta's `humanDescription` (Enforce T-Pose, symmetrize, etc.) across many passes.

## Symptom (what the user saw vs what metrics saw)
- The character's MESH deformed wrong — limbs to wrong places / "T-pose-ish", parts not
  following the body. In Generic playback the mesh sat at ~bind/T-pose.
- BUT every bone-space metric said the skeleton was fine: bones animated, rotations looped,
  left/right symmetric. We spent many iterations chasing a "Humanoid muscle-space foot
  retarget asymmetry" that was real in BONE space but NOT what the user was seeing.
- Lesson: **if the user reports wrong-looking deformation but your bone/rotation numbers
  look fine, you are measuring the wrong layer — suspect SKINNING (mesh↔bone bind), not
  the animation/retarget.**

## The decisive test (bones vs skin)
1. Same Animator controller + clips on TWO models: the suspect (overwritten) FBX and a
   PRISTINE With-Skin export of the same character. Identical setup.
2. Read `SkinnedMeshRenderer.bones[]`'s own bone rotations (e.g. LeftArm) AND look at the
   rendered mesh. If the SMR's bones are animating (arms down) but the mesh renders a
   different pose (arms out) → the mesh is NOT following its bones → SKINNING corruption.
3. Pristine deformed correctly; overwritten rendered T-pose mesh over moving bones →
   corruption is in the overwritten FBX's bind, introduced by the byte-overwrite.

Gotcha: `bones.Length == bindposes.Length == distinct`, no nulls, correct rootBone — ALL
look healthy. A count/integrity check does NOT catch bad bindpose VALUES. Only the
bones-animate-but-mesh-doesn't visual/behavioral test catches it.

Also: a Humanoid avatar can MASK the corruption (it rebuilds its own bind/retarget), so the
mesh appears to deform in Humanoid mode while Generic (raw bindposes) exposes the break.

## Correct pipeline
- Treat a skinned FBX as immutable content: to replace a model, import the NEW file as its
  OWN asset (fresh .meta) and rewire references — do NOT overwrite bytes under a foreign meta.
- For Mixamo NPCs: use a clean "With Skin" export per base, imported normally. If you need a
  stable prefab GUID, keep the PREFAB and repoint its model reference; never byte-swap the FBX.
- Animations that must avoid Unity Humanoid muscle-space quirks can be imported GENERIC and
  bound by transform path (mixamorig:* paths are identical across Mixamo bases) — but this
  ONLY looks right if the model's bindposes are intact, which a byte-overwritten FBX may not be.
