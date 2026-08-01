---
title: MTree (modular_tree) crown_shape crashes the native C++ core - use a Ramp node into Length instead
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-06-02'
project: Kerf - Sawmill Tycoon
tags:
- blender
- modular-tree
- mtree
- crash
- procedural-trees
- crown-shape
- ramp-node
applies_to:
- blender-5.1
- modular_tree-extension
source: ''
severity: high
time_lost: ~60min
promoted: '2026-07-30'
---

# MTree (modular_tree) crown_shape crashes the native C++ core - use a Ramp node into Length instead

## Problem
Setting a Branch node's `crown_shape` to anything other than `CYLINDRICAL` (e.g. `CONICAL`)
and then running `build_tree` hard-crashed the BlenderMCP socket every time
(`WinError 10054`, connection forcibly closed). Blender stayed alive, but the
`modular_tree` extension got **disabled**, which silently removed the custom
`mt_MtreeNodeTree` node group from memory. Symptom looked like a random/flaky MCP
connection; it was 100% reproducible and tied to crown builds only. Every
non-crown build (radial-res sweeps, density/length changes, ramp builds) was
perfectly stable.

## Root cause
`branch_node.py` passes the crown shape straight to the compiled core:
`func.crown.shape = lazy_m_tree.CrownShape(int(py_shape))`. The native m_tree
library segfaults on the non-cylindrical crown code path in this build. A native
crash kills the socket handler thread AND trips Blender's add-on safety, which
disables the extension; disabling unregisters the custom node-tree datablock, so
the tree "disappears." Not fixable from Python.

## Solution
Reproduce the crown envelope with a **stable feature**: the crown is only a
length-vs-height multiplier. Wire a **Ramp node (`mt_RampPropertyNode`) into the
Branches `Length` property socket** (property sockets accept a linked
`from_node.get_property()`, see `property_socket.py`). The SimpleCurveProperty
maps height-along-trunk → branch length:
- `start` = length at the bottom (long, e.g. 5.5)
- `end`   = length at the top (short, e.g. 0.6)
- `power` = curve bias (1.5 keeps it wide low, tapers sharply near the crown)
This gives a genuine conical spruce silhouette with no crash and no mesh faking.

Also: re-enable the extension after each crash with
`bpy.ops.preferences.addon_enable(module="bl_ext.blender_org.modular_tree")`, and
**save the .blend immediately after a good build** so a future drop can't lose the
node tree.

## What didn't work
- `crown_shape='CONICAL'` (and by extension any non-cylindrical shape) - crashes.
- Waiting for the MCP socket to "recover" - it won't; the add-on is disabled and
  must be re-enabled, and the node tree must be rebuilt/reloaded.

## Gotcha worth remembering
modular_tree's `CONICAL` envelope math is `0.2 + 0.8*ratio` - **widest at the top**
(an inverse cone). A real fir/spruce silhouette is their `InverseConical`
(`1.0 - 0.8*ratio`). The in-UI tooltip ("Conical … pine/fir") is misleading. The
Ramp-into-Length approach sidesteps the naming entirely.

## Transferability
Any project using the modular_tree Blender extension for procedural trees on
Blender 5.x. The "native crash → add-on auto-disabled → custom datablock vanishes"
failure signature also generalises: if a custom-node add-on's datablocks keep
disappearing after a build, suspect a native crash disabling the add-on, not data
loss - re-enable and persist to disk.

## Related
- 20260602-1500-mtree-nonmanifold-voxel-remesh.md
- blender-mcp-interactive-remodel-loop.md

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260602-1500-mtree-nonmanifold-voxel-remesh|Reducing MTree (Modular Tree) meshes to low-poly: Decimate & Quadriflow refuse, Voxel remesh works]] - wspolne: modular-tree, mtree, blender
<!-- /POWIAZANE:auto -->
