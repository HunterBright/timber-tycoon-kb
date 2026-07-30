---
title: Swapping a Blender-rigged humanoid for a Mixamo rig without breaking prefab/SO references
type: pattern
status: draft
confidence: low
verified: ''
date: '2026-05-30'
project: Kerf - Sawmill Tycoon
tags:
- unity
- mixamo
- humanoid-avatar
- fbx-import
- arm-splay
- prefab-guid
- retargeting
applies_to: []
source: ''
suggested-category: engine/patterns
---

# Swapping a Blender-rigged humanoid for a Mixamo rig without breaking prefab/SO references

## Problem
A custom Blender-exported humanoid (NPC) splayed its arms outward when playing
retargeted Mixamo Walk/Idle clips. Root cause: the imported humanoid **avatar's
bind pose was bad/asymmetric**. Confirmed by dumping the avatar's upper-arm bind
rotations from the old `.meta`:

```
UpperArm.L rot = (-0.0255, 0, -0.0085, 0.9996)
UpperArm.R rot = ( 0.000, 0.9483, -0.0269, -0.3161)   <- garbage, asymmetric
```

The right arm bind (y=0.948, w=-0.316) is ~nonsense; Mecanim retargeting amplifies
it during animation → splay. A clean Mixamo T-pose fixes it (new binds symmetric,
~0.128 quaternion ≈ 14.7° about X on both arms).

## Working recipe (model swap that keeps all references)
1. **Overwrite the FBX bytes, keep the `.meta`.** Copy the new (Mixamo) FBX over
   the existing `Model.fbx` but DO NOT touch `Model.fbx.meta` → the FBX GUID is
   preserved, so every prefab/SO that referenced it stays linked. Back up the old
   FBX+meta first (to a non-Assets backup dir so Unity doesn't import a duplicate).
2. **Clear the stale humanoid mapping and rebuild the avatar fresh.** The old
   `.meta` still carries `humanDescription.human[]` + `skeleton[]` with the OLD
   bone names (e.g. `UpperArm.L`), which won't match the new `mixamorig:` skeleton.
   In a ModelImporter pass: set animationType=Generic + reimport, then clear
   `human`/`skeleton`, set animationType=Human, `avatarSetup=CreateFromThisModel`,
   `autoGenerateAvatarMappingIfUnspecified=true`, reimport. Avatar is rebuilt from
   the Mixamo rest pose.
3. **Importer settings for Mixamo:** Scale Factor=1, Bake Axis Conversion=ON.
   Verify the resulting height empirically from `SkinnedMeshRenderer.sharedMesh.bounds.size.y`
   (~1.75 m) — don't trust unit-conversion guesses; log it.
4. **Repair the prefab IN PLACE** with `PrefabUtility.LoadPrefabContents` +
   `SaveAsPrefabAsset(root, samePath)`. This preserves the prefab GUID **and the
   root GameObject fileID** — critical when a ScriptableObject (e.g. a spawn config)
   references the prefab's root by `{fileID, guid}`. NEVER recreate the prefab via a
   fresh instantiate+SaveAsPrefabAsset to a new object — that regenerates the root
   fileID and silently nulls the SO reference.
   Re-assert on the repaired prefab: Animator (controller + new avatar,
   applyRootMotion=false), collider, and set `SkinnedMeshRenderer.sharedMaterials`
   to the desired material (FBX material name differs after the swap).

## Verification that actually proves it
Bind-rotation symmetry is a strong static signal, but the real proof is animated:
enter Play Mode, runtime-`Instantiate` the prefab, `Animator.SetBool` Idle→Walk, and
capture. Mecanim only evaluates poses in Play Mode. (Unity's multi-angle scene
capture framed on the instance's id is the cleanest way to see arm position from
front/side without fighting camera setup.) Runtime instantiate + SetBool is safe in
Play Mode — no AssetDatabase writes, no save_scene.

## Why it matters elsewhere
Any project importing Mixamo (or any external) humanoid animations onto a custom
mesh hits this. The two reusable kernels are: (a) **content-overwrite + meta-keep**
to swap a model while preserving its GUID, and (b) **load/save prefab contents in
place** to preserve prefab GUID + root fileID so referencing SOs/scenes survive.
