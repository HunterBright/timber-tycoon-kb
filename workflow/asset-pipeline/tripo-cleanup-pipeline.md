---
title: Tripo Cleanup Pipeline
type: lesson
status: draft
confidence: medium
verified: ''
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- tripo
- blender
- cleanup
- decimation
- retopo
applies_to:
- blender-pipelines
source: ''
description: 'Tripo AI generates models with inner faces, broken UVs, 8k+ tris. Standard cleanup: inner faces → UV rework → bake → decimate to 5k tris → FBX export. 30-60 min per asset.'
severity: medium
suggested-category: workflow/asset-pipeline
name: tripo-cleanup-pipeline
---

# Tripo Cleanup Pipeline

## Context

Raw Tripo output is not production-ready for Unity. Common issues in every Tripo export:
- Inner faces (hidden geometry inside the mesh = extra draw calls, lighting artifacts)
- Broken or non-existent UV maps
- 8,000-20,000 tris (overbudget for TT's 5,000 hero asset limit)
- Disconnected/floating elements (see [[tripo-asymmetric-floating-retopo]])

Standard cleanup pipeline: ~30-60 minutes per asset, vs. 3-8 hours to model from scratch.

## Steps

**Step 1: Import**
Import `.obj` or `.glb` from Tripo into Blender (File → Import → OBJ/glb).

**Step 2: Inner face removal**
```
Edit Mode → Mesh → Select All → Select → All by Trait → Interior Faces
(if no "Interior Faces" option: Face Select → Alt+Click to isolate → identify manually)
Delete selected faces (X → Faces Only)
```

**Step 3: Recalculate normals**
```
Edit Mode → Select All → Mesh → Normals → Recalculate Outside
(Shift+N shortcut)
```

**Step 4: UV rework**
- Background props: `Smart UV Project` (angle 66°, island margin 0.02)
- Hero assets: manual unwrap for important surfaces, Smart UV for obscured geometry
- Verify: no overlapping islands for baked assets

**Step 5: Bake**
Cycles bake, Diffuse Color Only:
- 512×512 for background props
- 1024×1024 for hero / interactive assets
- Output: `{Name}_Diffuse_Bake.png`

Optional: bake Normal map separately if high-detail surface needs normal-mapped fidelity.

**Step 6: Decimate**
- Target: 5,000 tris for hero props, 2,000 for background
- Modifier: Decimate → Collapse → Ratio (adjust until target)
- Apply modifier after confirming topology acceptable

**Step 7: Export FBX**
Standard TT settings:
- `FBX_SCALE_ALL`
- `bake_space_transform = True`
- `mesh_smooth_type = 'OFF'`

## When to skip cleanup

Background props at distance (>20m from player) where player won't notice:
- Skip inner face removal (not visible)
- Skip UV rework if no baked texture needed (vertex color only)
- Still decimate to budget

## Time summary

Full cleanup: 30-60 minutes. Partial (background prop): 10-15 minutes. From-scratch modeling: 3-8 hours. Tripo + cleanup = fastest path to production asset for most TT prop types.

See also: [[tripo-asymmetric-floating-retopo]], [[tripo-vocab-firewood-wedge]], [[blender-headless-python-generation]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[tripo-asymmetric-floating-retopo|Tripo Asymmetric / Floating Elements - Blender Retopo]] - wspolne: retopo, cleanup, tripo
- [[20260719-2015-ai-gen-model-geometry-debt|Modele generowane przez AI (Tripo/Hunyuan): dług geometryczny - czasem taniej zbudować od zera]] - wspolne: cleanup, tripo, blender
- [[20260602-1500-mtree-nonmanifold-voxel-remesh|Reducing MTree (Modular Tree) meshes to low-poly: Decimate & Quadriflow refuse, Voxel remesh works]] - wspolne: retopo, blender
- [[character-pipeline-tripo-mixamo-unity|Character pipeline: Tripo mesh → Mixamo rig → Unity (clean, working recipe)]] - wspolne: tripo, blender
- [[20260531-0934-tripo-polygon-soup-inverted-winding-fix|Tripo / AI-generated meshes import as "polygon soup" - see-through holes under single-sided rendering are a winding problem caused by UNWELDED verts, not interior faces]] - wspolne: tripo, blender
- [[20260531-1530-unity-humanoid-autorig-mirrored-foot|Crooked foot under Unity Humanoid = auto-rig copied the foot bind pose instead of mirroring it]] - wspolne: tripo, blender
<!-- /POWIAZANE:auto -->
