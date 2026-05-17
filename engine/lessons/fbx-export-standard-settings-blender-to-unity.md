---
type: lesson
project: timber-tycoon
suggested-category: engine/lessons
tags: [blender, fbx, export, unity, scale-factor]
severity: medium
time_lost: ""
date: 2026-05-17
status: draft
applies_to: ["unity-projects", "blender-pipelines"]
---

# FBX Export Standard Settings (Blender → Unity)

## Problem
Without a fixed FBX export configuration, assets arrive in Unity with wrong scale, incorrect rotation, or broken normals. "Why is my asset 100× too small?" and "why is it inside-out?" are the recurring symptoms.

## Root cause
Blender and Unity use different coordinate systems and unit conventions. FBX export settings control how the axis/scale conversion is applied — wrong settings produce wrong output.

## Solution
Standard FBX export settings that eliminate scale/rotation/unit bugs:

```python
axis_forward='-Z'
axis_up='Y'
apply_unit_scale=True
apply_scale_options='FBX_SCALE_ALL'
global_scale=1.0
mesh_smooth_type='OFF'          # preserves flat shading
use_mesh_modifiers=True
bake_space_transform=True       # BUT see bake_space_transform bug for linked duplicates
```

Apply transforms before export: Ctrl+A → All Transforms in Blender Object Mode.

Unity Import Settings:
- Scale Factor = 1
- **Convert Units: OFF** (critical — do not double-convert)
- Bake Axis Conversion: ON

Validation: if Unity shows Scale 100 → export settings wrong. If mesh invisible → normals flipped (recalculate outside in Blender).

## What didn't work
Various combinations of axis settings without `FBX_SCALE_ALL` produce scale=100 imports.

## Transferability
These 7 settings are the standard Blender → Unity FBX baseline. Any project using this pipeline should lock these settings and never deviate unless there's a documented reason.

## Related
- [bake_space_transform rotation bug](bake-space-transform-linked-duplicates-rotation-bug.md)
- [Procedural textures must be baked](procedural-textures-need-bake.md)
