---
title: Determine a humanoid's up/forward axis from ARMATURE bone landmarks, not from bounding-box max-spread — a T-pose arm span can beat true height
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-05-31'
project: Kerf - Sawmill Tycoon
tags:
- blender
- orientation
- armature
- mixamo
- humanoid
- t-pose
- bounding-box
- diagnostics
applies_to:
- blender
- rigging
- character-pipeline
source: ''
severity: low
time_lost: minimal — avoided a wrong camera aim / wrong region mapping
suggested-category: engine/lessons
---

# Determine a humanoid's up/forward axis from ARMATURE bone landmarks, not from bounding-box max-spread — a T-pose arm span can beat true height

## Problem
Needed to locate a character's "right lower leg / knee" and map mesh defects to body regions. A heuristic that picks the vertical axis as the axis of largest extent flagged **X** as vertical — but X was the **T-pose arm span (1.757 m)**, which narrowly beat the true **height (Z spread 1.741 m)**. Also, `object.dimensions` reported a thin Z while the mesh actually stood upright along world Z — because the object was parented to a rotated armature, so its *local* frame differed from *world*.

## Root cause
1. In a T-pose / A-pose, arm span ≈ height, so "largest extent = vertical" is unreliable.
2. `object.dimensions` is the LOCAL bounding box × scale. If the object inherits rotation from a parent (e.g. an FBX-imported armature converting Y-up→Z-up), local axes ≠ world axes. World-space landmarks are what matter.

## Solution
Read the **armature bones in world space** (`armature.matrix_world @ bone.head_local`). For a standard rig (e.g. Mixamo `mixamorig:` names) the landmarks are unambiguous:
- Vertical = axis from `Hips`/`Spine` up to `Head`/`HeadTop_End`, down to `Foot`/`Toe` (here: Z, feet≈0 → head≈1.5).
- Character's right vs left = sign of X on `Right*`/`Left*` bones (here right leg = −X).
- Knee = `RightUpLeg.tail` == `RightLeg.head`; shin = `RightLeg` head→tail; ankle = `RightFoot.head`.
Use bone segment positions to aim diagnostic cameras and to label defect locations by nearest bone (point-to-segment distance) — far more robust than guessing from bbox.

## What didn't work
- Max-extent-axis heuristic for "vertical" on a T-pose mesh.
- Reading `object.dimensions` as if it were world-aligned when the object had inherited/parent rotation.

## Transferability
Any character/rig diagnostic in Blender across projects. Whenever an armature exists, prefer bone landmarks for anatomy/orientation; reserve bbox heuristics for prop meshes with no rig. The parent-rotation vs local-dimensions gotcha applies to all FBX-imported hierarchies.

## Related
- [[20260531-0934-tripo-polygon-soup-inverted-winding-fix]]
