---
type: pattern
project: Timber Tycoon
suggested-category: workflow/asset-pipeline
tags: [blender, mixamo, rigging, fbx-export, character, mcp]
date: 2026-05-31
status: draft
---

# Extract a clean mesh-only FBX from a rigged source for Mixamo re-rig

## When to use
A character's existing in-game rig is defective (e.g. a Blender auto-rig that copied
rather than mirrored a foot bind pose) and you want to discard it and re-rig fresh via
Mixamo. Mixamo's auto-rigger needs an **un-rigged static mesh** — no armature, no skin
weights, no bind pose. This pattern produces that file without altering the artist's
geometry.

## Steps
1. **Pick the source.** Prefer the original `.blend` over the project's rigged `.fbx`.
   The `.blend` holds the artist's mesh directly; the `.fbx` has been through an
   export/import roundtrip (split-normal baking, possible material-slot reordering,
   scale quirks). Read the `.blend`'s data-block directory non-destructively first to
   confirm it has a single clean mesh:
   `with bpy.data.libraries.load(path) as (src, dst): print(src.objects, src.meshes, src.armatures)`
2. **Append both the mesh AND the armature** into a clean scene (not just the mesh) so
   the parent pointer and Armature-modifier target resolve without dangling links.
   Then `scene.collection.objects.link(obj)` each one.
3. **Strip the rig off the mesh** (geometry is untouched because the mesh data IS the
   bind pose):
   - `for m in list(obj.modifiers): obj.modifiers.remove(m)`  (kills the Armature modifier)
   - `obj.vertex_groups.clear()`  (kills all skin weights)
   - Clear parent keeping transform: `mw = obj.matrix_world.copy(); obj.parent = None; obj.matrix_world = mw`
4. **Apply all transforms** on the mesh only (`transform_apply(location,rotation,scale)`).
   Guard first: skip/abort if `obj.data.users > 1` (multi-user mesh can't be applied) or
   shape keys exist. Select ONLY the mesh so you don't bake the armature's transform.
5. **Sanity-check before export:** scene units (METRIC / meters / scale 1.0), world bbox
   height ~1.6–1.9 m for a human, and `0 modifiers + 0 vertex groups` remaining. A T-pose
   shows as a large horizontal axis (~1.5 m arm span) — that's correct, not a bug.
6. **Export mesh-only:** `bpy.ops.export_scene.fbx(use_selection=True, object_types={'MESH'},
   axis_forward='-Z', axis_up='Y', apply_unit_scale=True, apply_scale_options='FBX_SCALE_ALL',
   global_scale=1.0, mesh_smooth_type='OFF', bake_anim=False)`. Leave the armature in the
   scene — `object_types={'MESH'}` + mesh-only selection excludes it.
7. **Verify WITHOUT importing:** read the written FBX as bytes and assert the bone/skin
   markers are absent: `b'LimbNode'`, `b'Deformer'`, `b'Pose'`/`b'BindPose'`, and the
   armature's name must all be `False`; `b'Mesh'` must be `True`. Binary FBX stores names
   with a `\x00\x01` separator, so literal `::` substrings won't match — check the bare
   tokens. `AnimStack` showing `True` even with `bake_anim=False` is a harmless empty
   container Blender always writes; Mixamo ignores it.

## Why this works
Removing the Armature modifier outputs the raw mesh-data coordinates, which already ARE
the bind/T-pose (the artist modelled the mesh, then bound the armature to it). So no
geometry changes — you're only discarding deformation metadata. A `LimbNode`-free binary
scan is a fast, import-free proof that no skeleton leaked into the file.

## Trade-offs
- Re-rigging in Mixamo loses any hand-authored weights/bones from the original rig — fine
  when the original rig was the thing that was broken.
- Mixamo's humanoid skeleton replaces a custom bone layout; downstream Animator setups must
  target the Mixamo/Generic hierarchy (the proven fix on this project used **Generic**, not
  Humanoid, to avoid Blender→Unity foot retarget defects).

## Variants
- **From FBX instead of .blend** when no source blend exists: `bpy.ops.import_scene.fbx`,
  then identical strip/apply/export. Watch for `Armature|mixamorig`-style bone-name prefixes
  and split-normal artifacts.
- **Multiple mesh pieces** (separate hair/clothing/eyes): export all mesh objects together,
  do NOT merge — keep the artist's piece breakdown; Mixamo skins multiple meshes fine.
- **MCP caveat:** never `read_factory_settings`/`read_homefile` to clear the scene — it drops
  the Blender MCP socket. Delete objects or append into the live empty scene instead.
