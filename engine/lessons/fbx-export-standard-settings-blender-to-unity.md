---
title: FBX Export Standard Settings (Blender → Unity)
type: lesson
status: needs-reproduction
confidence: medium
verified: '2026-07-30'
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- blender
- fbx
- export
- unity
- scale-factor
applies_to:
- unity-projects
- blender-pipelines
source: zweryfikowane reprodukcja i dokumentacja 2026-07-30, patrz AUDYT-SPORNYCH-WPISOW
severity: medium
suggested-category: engine/lessons
time_lost: ''
audit_verdict: DO SPRAWDZENIA
---

# FBX Export Standard Settings (Blender → Unity)

> [!warning] Ten wpis zostal zweryfikowany 2026-07-30 i werdykt brzmi: **DO SPRAWDZENIA**
>
> To nie jest uniwersalny standard, tylko **wersjonowany preset TEGO projektu** dla tej pary Blender-Unity.
> Przy zmianie wersji trzeba przetestowac go od nowa.
>
> Pelne uzasadnienie, dowody i proponowane poprawki: [[AUDYT-SPORNYCH-WPISOW]].
> Tresc ponizej NIE zostala jeszcze przepisana - czytaj ja z ta uwaga.


## Problem
Without a fixed FBX export configuration, assets arrive in Unity with wrong scale, incorrect rotation, or broken normals. "Why is my asset 100× too small?" and "why is it inside-out?" are the recurring symptoms.

## Root cause
Blender and Unity use different coordinate systems and unit conventions. FBX export settings control how the axis/scale conversion is applied - wrong settings produce wrong output.

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
- **Convert Units: OFF** (critical - do not double-convert)
- Bake Axis Conversion: ON

Validation: if Unity shows Scale 100 → export settings wrong. If mesh invisible → normals flipped (recalculate outside in Blender).

## What didn't work
Various combinations of axis settings without `FBX_SCALE_ALL` produce scale=100 imports.

## Transferability
These 7 settings are the standard Blender → Unity FBX baseline. Any project using this pipeline should lock these settings and never deviate unless there's a documented reason.

## Related
- [bake_space_transform rotation bug](bake-space-transform-linked-duplicates-rotation-bug.md)
- [Procedural textures must be baked](procedural-textures-need-bake.md)

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260713-1030-verify-in-target-engine-not-source-tool|Weryfikuj asset w silniku DOCELOWYM, nie w narzędziu źródłowym]] - wspolne: export, fbx, blender
- [[bake-space-transform-linked-duplicates-rotation-bug|bake_space_transform + Linked Duplicates = 90° Rotation Injection]] - wspolne: export, fbx, blender
- [[20260531-0934-tripo-polygon-soup-inverted-winding-fix|Tripo / AI-generated meshes import as "polygon soup" - see-through holes under single-sided rendering are a winding problem caused by UNWELDED verts, not interior faces]] - wspolne: fbx, blender
- [[20260626-1203-fbx-pivot-direction-vs-procedural-placement|Pivot/geometry direction of an FBX must match what a procedural placement tool assumes]] - wspolne: fbx, blender
- [[20260629-1145-blender-empties-bake-space-transform-double-axis|FBX with parent EMPTIES imports tipped 90° when exported with bake_space_transform=True]] - wspolne: fbx, blender
- [[20260704-2330-blender-unity-flat-panel-dual-face-texture|Blender flat panel textured on one face renders BLANK in Unity (axis-flip picks the wrong face)]] - wspolne: fbx, blender
<!-- /POWIAZANE:auto -->
