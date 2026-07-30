---
title: Curved/meandering water flow via a baked flow map (arc-length V + per-vertex tangent)
type: pattern
status: draft
confidence: low
verified: ''
date: '2026-06-06'
project: Kerf - Sawmill Tycoon
tags:
- unity
- shader
- water
- river
- flow-map
- uv
- vertex-color
- meander
- centerline
applies_to: []
source: ''
suggested-category: engine/patterns
---

# Curved/meandering water flow via a baked flow map (arc-length V + per-vertex tangent)

## When to use
A scrolling water shader (waves, foam bands, flow streaks) on a **curved or meandering**
river/stream where the channel changes direction. A single global flow vector
(`_FlowDirection`) or a single straight UV axis cannot follow the bends — water visibly
flows across the channel at every turn.

## The trap that looks correct but isn't
A common "fix" is to bake `V = f(worldX)` (e.g. cumulative arc-length indexed by X-bin) so V
increases along the river's length. It passes a naive check — within a thin X-slice V is
constant and U spans 0..1 — but it is **wrong**: V's iso-lines are world-vertical
(constant-X) lines, NOT perpendicular to the channel. Since wave crests are iso-V lines, at
every bend they stay world-vertical while the channel tilts → crests slide **bank-to-bank**.
Diagnostic that distinguishes good from bad: sample V within a thin X-slice where the channel
is tilted. If V is *constant* there, V is world-X (bad). If V *spans a range* there, V follows
the centerline (good — across-width verts project to different arc-lengths).

## Steps (bake side — editor script over the mesh)
1. Build an ordered centerline polyline through the channel (here: per-X-bin Z-centroid; valid
   because X was monotone with no folds). Compute cumulative **arc-length S[k]** per node.
2. For each vertex, **project onto the nearest centerline segment**:
   - `V (uv.y)` = interpolated arc-length at the projection (downstream metres).
   - downstream **tangent** = unit segment direction (toward increasing arc = downstream).
     Store XZ in **vertex color RG**, encoded `*0.5+0.5` (or a TEXCOORD2).
   - `U (uv.x)` = across-width 0..1 (local bank window) — drives edge mask / side damping.
3. Re-bake the mesh asset; keep vertex count and the existing geometry (flatten/slope).

## Steps (shader side)
- **Waves / foam bands:** phase from V → `sin(time*WaveSpeed - V*WaveFreq)`. Crests are iso-V,
  now perpendicular to the channel, travelling downstream through every bend. Minus sign sets
  waterfall→bridge direction.
- **Flow scroll / streaks:** decode tangent `t = color.rg*2-1`; project world pos onto it
  `flowDist = dot(positionWS.xz, normalize(t))`; scroll noise by `flowDist*scale - time*FlowSpeed`.
  This pans along the per-vertex downstream direction, not a global vector.

## Why this works
Projecting onto the centerline makes V a true scalar "distance downstream" field whose
gradient is parallel to the local flow everywhere; its level sets are perpendicular to flow.
The baked tangent gives the shader a real per-point direction without ddx/ddy (unreliable on
low-poly). Both systems then follow the meander exactly.

## Trade-offs
- Bake is O(verts × centerlineNodes) — trivial for ~1700 verts / ~95 nodes.
- Centerline construction assumes a parametrisable channel. X-bin centroids only work when one
  world axis is monotone along the river (no S-folds back on that axis). For true S-folds, order
  the centerline by a proper path walk instead of by X.
- Tangent in vertex color costs the COLOR channel; use TEXCOORD2 if color is needed for tint.

## Variants
- Store tangent + arc-length in a **baked flow-map texture** sampled by world XZ instead of in
  the mesh — decouples from mesh topology, lets you reuse one map across LODs.
- If only the waves matter and no flow texture is assigned, V alone is enough; the tangent is
  only needed once a `_NoiseTex`/flow texture drives visible directional streaks.
