---
title: Blender Headless Python Script Generation
type: pattern
status: draft
confidence: medium
verified: ''
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- blender
- python
- headless
- automation
- generation
applies_to:
- blender-pipelines
- claude-code-projects
source: ''
description: Generate 3D assets via blender.exe --background --python script.py. Script creates geometry, assigns materials, saves .blend, exports FBX, renders preview. Deterministic + version-controllable.
severity: high
suggested-category: workflow/3d-models
name: blender-headless-python-generation
---

# Blender Headless Python Script Generation

## When to use
When generating 3D assets programmatically - especially when: the asset follows geometric rules (racks, machines, furniture), the asset needs to be regenerated with different parameters, or iterative development requires consistent reproducibility.

## Steps

**Command:**
```bash
"C:/Program Files/Blender Foundation/Blender 5.0/blender.exe" \
  --background \
  --python "<project-root>/_BlenderScripts/01_create_frame.py"
```

`--background`: no GUI, headless mode. Script runs, Blender closes.

**Script structure (template):**
```python
import bpy
import math

# 1. Clear scene
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete(use_global=False)

# 2. Set units
bpy.context.scene.unit_settings.system = 'METRIC'
bpy.context.scene.unit_settings.scale_length = 1.0

# 3. Create geometry
bpy.ops.mesh.primitive_cube_add(size=1.0, location=(0, 0, 0.5))
frame = bpy.context.active_object
frame.name = "Frame"
frame.scale = (2.0, 1.0, 1.0)
bpy.ops.object.transform_apply(scale=True)

# 4. Assign material (slot name matches Unity convention)
mat = bpy.data.materials.new("Mat_Frame_Wood")
mat.use_nodes = True
frame.data.materials.append(mat)

# 5. Save .blend
bpy.ops.wm.save_as_mainfile(filepath="D:/..._BlenderOutputs/Frame/Frame.blend")

# 6. Export FBX (standard settings - see [[fbx-export-standard-settings-blender-to-unity]] if it exists)
bpy.ops.export_scene.fbx(
    filepath="D:/.../Frame.fbx",
    use_selection=False,
    global_scale=1.0,
    apply_scale_options='FBX_SCALE_ALL',
    bake_space_transform=True,
    mesh_smooth_type='OFF',
    use_mesh_modifiers=True
)

# 7. Render multi-angle preview
bpy.context.scene.render.filepath = "D:/.../preview_iso.png"
# Set camera, render
bpy.ops.render.render(write_still=True)
```

**Directory conventions:**
- `_BlenderScripts/` - Python generation scripts (version-controlled in git)
- `_BlenderOutputs/{AssetName}/` - generated .blend + .fbx + preview PNGs
- Scripts are numbered: `01_frame.py`, `02_shelves.py` (execution order)

## Why headless generation

| Manual Blender modeling | Headless script |
|------------------------|----------------|
| Not reproducible | Deterministic - same params = same asset |
| Can't be version-controlled precisely | Script is the source of truth |
| Slow for parameterized variants | Script can loop: generate 3 rack sizes |
| Can't run autonomously (Claude Code) | Runs via `Bash` tool, no GUI needed |

## Blender version note

Blender Python API changes between major versions. Scripts target Blender 5.0+. Verify `bpy.app.version` at script start if cross-version compatibility is needed.

See also: [[procedural-textures-cycles-commercial]], [[zero-floating-zero-flickering-mandate]], [[iterative-checkpoint-workflow]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260612-1815-headless-tmp-sdf-font-generation|Headless TMP setup: import Essentials + generate SDF font asset from editor script]] - wspolne: headless, automation
- [[20260726-1535-blender-addon-parametryczny-suwaki|Parametryczny dodatek do Blendera: trzy pulapki, ktore kosztuja godzine kazda]] - wspolne: python, blender
- [[20260610-1820-blender-mcp-failure-headless-fallback|blender-mcp bridge failure modes + headless CLI fallback]] - wspolne: headless, blender
- [[20260730-1710-blender-materials-clear-resets-face-indices|Mesh.materials.clear() zeruje material_index na ściankach]] - wspolne: python, blender
- [[20260704-1732-blender-linked-basecolor-recolor|Recoloring a Blender material whose Base Color is LINKED does nothing via default_value]] - wspolne: python, blender
- [[20260725-1930-blender-pixels-buffer-not-converted-to-srgb-on-png-save|Blender nie przelicza `image.pixels[]` na sRGB przy zapisie PNG]] - wspolne: python, blender
<!-- /POWIAZANE:auto -->
