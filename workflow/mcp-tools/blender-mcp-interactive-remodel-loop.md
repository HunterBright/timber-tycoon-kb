---
title: Blender-MCP Interactive Remodel Loop (GUID-Preserving In-Place Replace)
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-05-29'
project: Kerf - Sawmill Tycoon
tags:
- blender
- blender-mcp
- unity
- fbx
- guid
- prefab
- pipeline
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Blender-MCP Interactive Remodel Loop (GUID-Preserving In-Place Replace)

## Problem
Reworking an existing in-game asset's mesh usually means re-importing a new FBX, which gets a NEW GUID - and every prefab variant / material reference pointing at the old GUID breaks. You then re-wire references by hand. Also, building blind in headless Blender wastes iterations when the human only needs to eyeball the shape.

## Root cause
Unity tracks assets by the GUID in the `.meta` file, not by filename. Deleting and re-adding an FBX, or letting Unity generate a fresh `.meta`, changes the GUID and orphans references. And model approval is a visual judgement call that a screenshot resolves in one round but a headless export cannot.

## Solution
A validated loop using a LIVE Blender session over blender-mcp, with a GUID-preserving overwrite as the key trick:

1. **Build/modify in the live viewport** via blender-mcp (`execute_blender_code`), iterate with `get_viewport_screenshot`.
2. **STOP for human visual approval** - present a 3/4 + side screenshot, do not export until the human approves the shape. (Cheap to iterate here, expensive to redo after export.)
3. **Save source of truth**: the `.blend` (save-AS a per-asset file, don't clobber another asset's blend) + a generation `.py` that reproduces it. State honestly when interactive reshaping isn't byte-reproducible and keep the `.blend` authoritative.
4. **Bake albedo**: reconnect the procedural soil/material to Base Color, bake DIFFUSE color-only to a 512 PNG, then rewire the baked PNG as Base Color so the FBX carries a texture-based material (Unity can't read procedural nodes).
5. **FBX export** with the standard Blender→Unity settings (see [[fbx-export-standard-settings-blender-to-unity]]). Confirmed on **Blender 5.1.2**: `bpy.ops.export_scene.fbx(...)` still accepts every standard kwarg unchanged (`axis_forward='-Z'`, `axis_up='Y'`, `apply_unit_scale=True`, `apply_scale_options='FBX_SCALE_ALL'`, `global_scale=1.0`, `mesh_smooth_type='OFF'`, `use_mesh_modifiers=True`, `bake_space_transform=True`, `use_selection=True`). No API change needed for 5.1.
6. **Backup the old FBX + its `.meta` OUTSIDE the Assets folder** first (a local safety net; never let Unity import a duplicate).
7. **Overwrite the FBX in place** - copy the new `.fbx` bytes OVER the existing one and **do NOT touch the `.fbx.meta`**. The GUID is preserved, so the prefab variant auto-picks up the new mesh with zero re-wiring.
8. **Validate the import**: GUID retained, Scale Factor = 1, vert/tri count, UVs present, normals outward/up, bounding box on the expected axis, no console import errors.
9. **Test in Play Mode** (human-run).

## Why the GUID-preserving overwrite is the linchpin
Because only the FBX bytes change and the `.meta` is untouched, the GUID is stable, so:
- the prefab variant keeps its mesh/material references,
- material slot overrides survive,
- nothing downstream needs re-linking.

This turns "remodel an existing asset" from a re-wiring chore into a one-file copy.

## What didn't work
Letting a new FBX import under a new GUID, then chasing broken prefab references. Also exporting before human approval - wasted full bake+export+import cycles on shapes the human then rejected.

## Transferability
Any Unity + Blender project remodelling existing assets. The GUID-preserving in-place overwrite (keep the `.meta`, replace the bytes) is universal and the single most important habit for not breaking a live project.

## Related
- [[fbx-export-standard-settings-blender-to-unity]]
- [[procedural-textures-cycles-commercial]]
- [[blender-headless-python-generation]]
- [[shared-mesh-and-materials-reference]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260606-1632-in-place-fbx-overwrite-static-vs-rigged|In-Place FBX Overwrite: Safe for Static Meshes, Dangerous for Rigged]] - wspolne: guid, prefab, fbx
- [[20260610-1820-blender-mcp-failure-headless-fallback|blender-mcp bridge failure modes + headless CLI fallback]] - wspolne: blender-mcp, blender
- [[fbx-binary-overwrite-corrupts-bindposes|FBX binary-overwrite under a stale .meta corrupts skinned-mesh bindposes (mesh collapses to T-pose while bones animate)]] - wspolne: guid, fbx
- [[20260612-1340-unity-batch-fbx-import-meta-mirroring|Batch FBX import with pre-authored .meta files + prefab build in temp additive scene]] - wspolne: prefab, fbx
- [[20260719-1605-spawn-pool-raw-fbx-bypasses-prefab|Anty-wzorzec: pula spawnera wskazuje surowy FBX zamiast prefabu-wrappera]] - wspolne: prefab, fbx
- [[20260801-0826-trellis-2-generator-3d-bez-blokady-ue|TRELLIS.2 jako generator 3D bez blokady licencyjnej w UE]] - wspolne: pipeline, blender
<!-- /POWIAZANE:auto -->
