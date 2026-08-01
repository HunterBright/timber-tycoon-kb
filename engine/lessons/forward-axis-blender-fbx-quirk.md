---
title: Forward Axis = -transform.right (Blender FBX Quirk)
type: lesson
status: needs-reproduction
confidence: low
verified: '2026-08-01'
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- unity
- blender
- fbx
- transform
- vehicle
- forward-axis
applies_to:
- unity-projects
- blender-pipelines
source: ''
severity: high
time_lost: multiple debugging sessions
suggested-category: engine/lessons
---

# Forward Axis = -transform.right (Blender FBX Quirk)

## Problem
Vehicles modeled in Blender and exported via FBX have their local forward axis = `-transform.right` in Unity, not `transform.forward`. The car drives sideways or refuses to move. Steering and parking alignment are broken.

## Root cause
Blender's default Y-forward axis maps to Unity's -X axis after the FBX coordinate conversion (`axis_forward='-Z'`, `axis_up='Y'`). The mesh front ends up pointing along Unity's -X (negative right) direction.

## Solution
Define a forward axis property and use it everywhere instead of `transform.forward`:
```csharp
public Vector3 ForwardAxis => -transform.right;
```

Affects:
- `VehicleController` (AddForce direction)
- `NPCVehicle` (movement direction)
- `NPCParkingPDController` (slot forward alignment)

Detection: if car drives sideways on first test → check this first. It's always this.

Alternative (rejected): rotate model 90° in Blender before export - breaks existing prefabs.

## What didn't work
Using `transform.forward` directly - car drives sideways.

## Transferability
Any Blender-authored vehicle, character, or directional prop exported via FBX will have this quirk unless the Blender artist explicitly rotates the root object 90° in Y before export. Document this as a project convention; every new vehicle developer needs to know.

> **Weryfikacja 2026-08-01: objaw prawdziwy, ale podana przyczyna jest sprzeczna ze zrodlem.**
> Eksporter FBX Blendera liczy przeliczenie osi jako `axis_conversion(from_forward='Y',
> from_up='Z', to_forward='-Z', to_up='Y')`, czyli obrot wokol osi X - os boczna jest
> niezmiennikiem tego obrotu, wiec eksport nie moze zamienic osi "do przodu" na boczna;
> przod modelu musi byc narysowany wzdluz osi X juz w Blenderze
> ([kod eksportera](https://github.com/blender/blender/blob/main/scripts/addons_core/io_scene_fbx/__init__.py),
> [axis_conversion](https://github.com/blender/blender/blob/main/scripts/modules/bpy_extras/io_utils.py)).
> Wniosek "kazdy pojazd z Blendera bedzie mial ten problem" nie ma podstaw.

## Related
- [FBX export standard settings](fbx-export-standard-settings-blender-to-unity.md)
- [Self-collision compound BoxColliders](self-collision-compound-colliders-ignore.md)
- [Reverse-parking entry stub orientation - real-world consequence: Cross(tangent,up) offset in rear-first kinematic](../../genre/tycoon/patterns/reverse-parking-entry-stub-orientation.md)

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260704-2330-blender-unity-flat-panel-dual-face-texture|Blender flat panel textured on one face renders BLANK in Unity (axis-flip picks the wrong face)]] - wspolne: forward-axis, fbx, blender
- [[20260531-0934-tripo-polygon-soup-inverted-winding-fix|Tripo / AI-generated meshes import as "polygon soup" - see-through holes under single-sided rendering are a winding problem caused by UNWELDED verts, not interior faces]] - wspolne: fbx, blender
- [[20260626-1203-fbx-pivot-direction-vs-procedural-placement|Pivot/geometry direction of an FBX must match what a procedural placement tool assumes]] - wspolne: fbx, blender
- [[20260629-1145-blender-empties-bake-space-transform-double-axis|FBX with parent EMPTIES imports tipped 90° when exported with bake_space_transform=True]] - wspolne: fbx, blender
- [[20260713-1030-verify-in-target-engine-not-source-tool|Weryfikuj asset w silniku DOCELOWYM, nie w narzędziu źródłowym]] - wspolne: fbx, blender
- [[20260728-0915-fbx-skala-100-w-dzieciach-psuje-pomiary|FBX z Blendera: przelicznik jednostek siedzi w SKALI DZIECI, nie w korzeniu]] - wspolne: fbx, blender
<!-- /POWIAZANE:auto -->
