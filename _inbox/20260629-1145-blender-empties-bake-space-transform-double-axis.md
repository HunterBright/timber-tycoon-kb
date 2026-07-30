---
title: FBX with parent EMPTIES imports tipped 90° when exported with bake_space_transform=True
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-06-29'
project: Kerf - Sawmill Tycoon
tags:
- blender
- fbx
- unity
- axis-conversion
- bake_space_transform
- empties
- hierarchy
- import
- orientation
applies_to: []
source: ''
severity: high
suggested-category: engine/lessons
---

# FBX with parent EMPTIES imports tipped 90° when exported with bake_space_transform=True

## Symptom
A multi-part machine modeled in Blender (Z-up) imported into Unity **lying on its side**:
- mesh leaf objects came in rotated **270° (or 90°) about X**
- bounding-box height and depth were **swapped**
- base sat ~0.8 m **below** the floor (min.y negative) instead of resting at y≈0

The empties themselves (root + pivot groups) read as clean (identity / correct positions); only the visible mesh orientation was wrong. Toggling Unity's `bakeAxisConversion` flag did NOT fix it — it only flipped the tilt to the other side, proving the rotation was baked into the FBX, not an import setting.

## Root cause
The project's documented FBX export standard uses `bake_space_transform=True` (Blender → Unity, with `axis_forward='-Z'`, `axis_up='Y'`). That works for **flat** models (every mesh at the top level — e.g. the Pelletizer here). It BREAKS when the export contains a **parent-empty hierarchy** (root empty → group empties → meshes): with empties present, `bake_space_transform=True` applies the Y↔Z axis conversion **twice** — once on the root/group transforms and again baked into the mesh transforms — netting a ~90–270° flip. The empty-less reference model never showed it, which is why the "standard" looked safe.

## Fix
Re-export the **hierarchical** model with **`bake_space_transform=False`** (leave every other setting at the project standard, and keep Unity import = Scale 1 / Convert Units OFF / Bake Axis Conversion ON). The model then stands upright with the base at y≈0 and pivots intact.

Verify the fix the right way: a plain Blender→Blender re-import can show a misleading rotation difference (each FBX carries its own axis metadata). Force BOTH files into the identical Unity-native frame (Y-up, -Z-forward) and compare base_z / height / depth against a known-good upright reference — that ignores per-file metadata and predicts the Unity result.

## Caveat introduced by the fix (know it, handle it in the prefab)
With `bake_space_transform=False` + empties, the upright orientation is carried by a **+90° X rotation on the imported prefab ROOT** (not baked into the vertices the way a flat `=True` export bakes it). Consequences:
- Do NOT zero/reset the root rotation in the scene or it tips again (yaw/positioning is safe).
- When building the gameplay prefab, **wrap the imported model in an identity-rotation parent** so designers and code see a clean root.
- Pivot empties survive correctly: a rotating sub-group (e.g. a drum rotor) still spins about its axle in place — verify functionally by rotating it and confirming the part's bounds-center drift ≈ 0.

## Decision rule
- **Flat model (no empties):** `bake_space_transform=True` is fine (the existing standard).
- **Model with parent/pivot empties (rotors, hinges, separable groups):** export with `bake_space_transform=False`, and neutralize the resulting +90° root rotation by wrapping in an identity parent in the prefab.

## Bonus gotcha (same session, Unity 6.5)
In the unity-mcp `RunCommand` C# sandbox, `Object.GetInstanceID()` is obsolete and errors out — use **`GetEntityId()`**. Also `System.Collections.Generic.HashSet<>`/`ISet<>` fail to compile there (missing assembly ref) — use `List<>` instead.
