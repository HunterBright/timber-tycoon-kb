---
type: pattern
project: Timber Tycoon
suggested-category: engine/patterns
tags: [blender, mixamo, fbx, rigging, mcp, auto-rig, mesh-export, tripo]
date: 2026-05-31
status: draft
---

# Batch-extract clean mesh-only FBX from rigged .blend for Mixamo re-rig

## Problem
A character with a defective auto-rig (e.g. Blender's auto-rig copying instead of
mirroring a foot bind-pose) can't be salvaged by tweaking the bad rig. The reliable
fix is to discard the rig entirely and re-rig via Mixamo. Mixamo needs a **mesh-only**
FBX (no armature/skin data). When you have N such characters as `.blend` files, you
want a deterministic, verifiable batch that produces N clean FBX without hand-clicking
each one — and without trusting the tool's own "it worked" report.

## Pattern

**1. Never `open_mainfile` to load a source `.blend` over an MCP-connected Blender.**
`read_factory_settings` / `read_homefile` disconnect the MCP socket; `open_mainfile`
replaces the whole scene. Instead **append** datablocks into the live scene:

```python
with bpy.data.libraries.load(blend_path, link=False) as (df, dt):
    dt.objects = list(df.objects)
for o in dt.objects:
    if o is not None:
        bpy.context.scene.collection.objects.link(o)
```

Clear between models by deleting objects + `bpy.data.orphans_purge(...)` — never reset.

**2. Strip rig data at the DATA level, not via operators** (avoids context/override pain):

```python
for mod in list(mesh.modifiers): mesh.modifiers.remove(mod)   # drop Armature modifier(s)
mesh.vertex_groups.clear()                                    # drop skin weights
if mesh.parent:                                               # parent_clear KEEP_TRANSFORM
    mw = mesh.matrix_world.copy(); mesh.parent = None; mesh.matrix_world = mw
```

Leave the armature object in the scene (just don't export it) rather than deleting it.

**3. Apply transforms** with a `temp_override` (location/rotation/scale). Wrap in
try/except — shape keys make `transform_apply` raise; on failure SKIP + report, don't
force.

**4. Export mesh-only** with `object_types={'MESH'}` + `use_selection=True` (select only
the mesh). For Mixamo: `axis_forward='-Z', axis_up='Y', apply_unit_scale=True,
apply_scale_options='FBX_SCALE_ALL', global_scale=1.0, mesh_smooth_type='OFF',
bake_anim=False`.

**5. VERIFY independently — don't trust the exporter's word.** FBX (binary) stores
property/class names as plain ASCII, so rig data is detectable by byte search. Read the
written file and count `LimbNode`, `Deformer`, `BindPose`, `Skin`, `Cluster` — all must
be **0** for a true mesh-only file. Do this byte scan from a *second process* (e.g.
host-side PowerShell reading the file on disk with Latin1 encoding), so the QA isn't
circular with the tool that produced the file.

```powershell
$enc = [System.Text.Encoding]::GetEncoding(28591)  # Latin1, byte-faithful
$txt = $enc.GetString([System.IO.File]::ReadAllBytes($path))
foreach ($m in "LimbNode","Deformer","BindPose","Skin","Cluster") { $txt.Contains($m) }
```

**6. Record per-model metrics to catch problems before wasting a Mixamo upload:**
world-bbox height (humans ≈ 1.6–1.9 m), exported vertex count, mesh-piece count
(unexpected extra pieces = hair/eyes split), file size (a mesh-only body FBX is small,
~140–170 KB for ~2.5k verts; a suspiciously large file may still contain a rig), and the
pre-strip rig snapshot (proves you actually discarded a rig).

## Gotchas observed
- A T-pose character authored facing +X has arm-span along **Y**, so its Y-extent ≈
  height (`dy ≈ dz`). That looks alarming in raw bbox numbers but is correct — confirm
  with one front-view screenshot rather than flagging on bbox alone.
- Sources can carry **more than one** Armature modifier (double auto-rig). Iterate over
  `list(mesh.modifiers)` and remove all; don't assume exactly one.
- Validate the full recipe on ONE model end-to-end before batching the rest.

## Why it matters
Cuts an error-prone manual per-character chore to a deterministic batch, and the
two-source byte-scan verification means a bad export is caught at the desk, not after
an hour of Mixamo uploads.
