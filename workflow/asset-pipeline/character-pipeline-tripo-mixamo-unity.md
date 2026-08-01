---
title: 'Character pipeline: Tripo mesh → Mixamo rig → Unity (clean, working recipe)'
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-05-31'
project: Kerf - Sawmill Tycoon
tags:
- tripo
- mixamo
- unity
- blender
- character
- rigging
- generic-animation
- npc
- pipeline
applies_to: []
source: ''
name: character-pipeline-tripo-mixamo-unity
promoted: '2026-07-30'
---

# Character pipeline: Tripo mesh → Mixamo rig → Unity (clean, working recipe)

## Context
Use this when you need animated humanoid NPCs from an AI-generated mesh, retargeting a small set of shared clips (Walk/Idle) across many character bases, in a low-poly stylized project. This is the distilled, working path - the dead-ends that produced it are linked under **Traps to avoid**.

## Steps

**1. Tripo - generate the mesh**
- Smart Mesh, **~5k tris**, **T-pose**, **mitten hands** (no separate fingers - matches the rig and the low-poly look).
- Basecolor texture at **4K**, **NO PBR maps** (we want a flat-shaded look, not metal/roughness).
- **Bottom Center Pivot = ON** (feet at origin - see [asset-origin bottom-center convention](../../engine/patterns/asset-origin-bottom-center-convention.md)).
- Export **GLB**.

**2. Blender prep (headless)**
- Strip to **mesh only** (no Tripo armature/empties).
- **Re-orient**: character faces **−Y**, arms along **X** (Mixamo's expected import orientation).
- **Scale to ~1.75 m** tall; **apply all transforms** (object scale exactly (1,1,1)).
- **Recalculate normals outside.**
- Export **FBX** with the locked TT settings (`FBX_SCALE_ALL`, `bake_space_transform=True`, `mesh_smooth_type='OFF'` - see [FBX export standard settings](../../engine/lessons/fbx-export-standard-settings-blender-to-unity.md)). General Tripo cleanup details: [Tripo cleanup pipeline](tripo-cleanup-pipeline.md); headless Blender driving: [blender headless python generation](../3d-models/blender-headless-python-generation.md).

**3. Mixamo - rig AND bake the clips onto THIS character**
- Upload the mesh, **auto-rig with No Fingers**.
- Critically, **bake Walking + Idle onto the character itself** (search the animation, apply it to *your* uploaded model): **In Place**, **With Skin**, **30 fps**.
- Download **the character** (skinned) **and both clips**. Baking onto the character means the clips are already proportion-correct for this body - no cross-proportion retarget needed later.

**4. Unity - import as Generic, no avatar**
- Import the character as a **CLEAN new asset** (own GUID / fresh `.meta`). **Never overwrite an existing FBX's bytes** to keep a GUID - that corrupts bindposes ([why](../../engine/lessons/fbx-binary-overwrite-corrupts-bindposes.md)).
- Character `ModelImporter`: **Animation Type = Generic**, **No Avatar**.
- Import the clips as **Generic** too.
- **One shared Generic Animator controller** (Idle ⇄ Walk on an `isWalking` bool) drives **all** bases: every Mixamo rig uses identical `mixamorig:` transform paths, so a Generic clip binds by path directly to any base.
- `applyRootMotion = false` - the **NavMeshAgent moves the body**; the clip only animates the limbs in place.

## Why Generic, not Humanoid
Humanoid retargeting converts every clip into Unity's **muscle space** and back through each avatar's proportions. With auto-rigged characters that introduces cross-proportion distortion (one-sided feet, splayed limbs) and a whole avatar-calibration surface to fight. Generic binds the clip to the skeleton **by transform path** with no muscle-space round-trip - and because Mixamo names are identical across bases, one Generic clip set serves every character with zero retarget. The proportion-awareness we *do* want is baked in once, at the Mixamo step.

## Traps to avoid
- **Do not byte-overwrite an FBX under a foreign `.meta`** to preserve a GUID - it desyncs the mesh bindposes and the mesh collapses to ~T-pose while bones animate. Import clean and re-point references by name. → [FBX binary-overwrite corrupts bindposes](../../engine/lessons/fbx-binary-overwrite-corrupts-bindposes.md)
- **Do not diagnose a "wrong-looking" character from bone-space numbers alone.** Decouple the skeleton from the mesh and trust the render. → [Discriminating CLIP vs RIG vs SKIN](../../engine/patterns/discriminating-clip-vs-rig-vs-skin-humanoid-defect.md)
- **Do not chase Humanoid avatar T-pose/muscle-axis tuning** for an auto-rigged base; Generic sidesteps the entire problem.

## Trade-offs
- Generic loses Unity's retarget-to-arbitrary-avatar and Humanoid IK conveniences. Acceptable here: clips are baked per-base in Mixamo and NPCs are NavMesh-driven, so neither is needed.
- You re-bake Walk/Idle per character in Mixamo (a few minutes each) instead of retargeting one clip set in-engine. In exchange you get correct deformation with zero in-engine calibration.

## Related
- [FBX binary-overwrite corrupts bindposes](../../engine/lessons/fbx-binary-overwrite-corrupts-bindposes.md)
- [Discriminating CLIP vs RIG vs SKIN for a one-sided humanoid defect](../../engine/patterns/discriminating-clip-vs-rig-vs-skin-humanoid-defect.md)
- [Tripo cleanup pipeline](tripo-cleanup-pipeline.md)
- [FBX export standard settings (Blender → Unity)](../../engine/lessons/fbx-export-standard-settings-blender-to-unity.md)

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260531-1500-mixamo-clean-mesh-extraction|Extract a clean mesh-only FBX from a rigged source for Mixamo re-rig]] - wspolne: character, rigging, mixamo
- [[20260531-2000-blender-mesh-only-fbx-for-mixamo|Batch-extract clean mesh-only FBX from rigged .blend for Mixamo re-rig]] - wspolne: rigging, tripo, mixamo
- [[fbx-binary-overwrite-corrupts-bindposes|FBX binary-overwrite under a stale .meta corrupts skinned-mesh bindposes (mesh collapses to T-pose while bones animate)]] - wspolne: generic-animation, rigging, mixamo
- [[20260531-1530-unity-humanoid-autorig-mirrored-foot|Crooked foot under Unity Humanoid = auto-rig copied the foot bind pose instead of mirroring it]] - wspolne: tripo, mixamo, npc
- [[20260725-2320-fartuch-skinning-srednia-dwoch-ud-daje-zero|Fartuch ważony po połowie na oba uda NIE RUSZA SIĘ przy chodzie]] - wspolne: character, rigging, blender
- [[20260717-0010-generated-rig-bone-axis-defect-skeleton-transplant|Rigi z generatorów AI (Hunyuan): osie kości rozjechane z frontem modelu = wykrzywiona stopa w retargecie; lek = przeszczep szkieletu w Blenderze]] - wspolne: rigging, mixamo, blender
<!-- /POWIAZANE:auto -->
