---
title: Conform a mesh terrain to a path by sampling its centreline per-axis (not a fixed scan line, not a single plane)
type: pattern
status: draft
confidence: low
verified: ''
date: '2026-06-09'
project: Kerf - Sawmill Tycoon
tags:
- unity
- terrain
- mesh-editing
- raycast
- road
- leveling
- editor-script
applies_to: []
source: ''
suggested-category: engine/patterns
---

# Conform a mesh terrain to a path by sampling its centreline per-axis (not a fixed scan line, not a single plane)

## When to use
Flattening / leveling a **mesh-based** ground (MeshFilter, not Unity Terrain) so it
follows the grade of a road, river bank, trail, or any baked path — e.g. "level the
build/planting area to sit flush with the road." Especially when the path is not
guaranteed to run straight along a world axis.

## Steps
1. **Don't trust "the path runs straight along X" recon.** Verify the path's real
   footprint first: enumerate objects on the path's layer (report renderer/collider
   bounds), then a coarse grid raycast over the area onto the path layer only. A path
   that "runs along X" often **meanders in Z** within a band.
2. **Sample height per primary axis.** For each X (≈1 m steps), scan the cross axis Z
   across the path band (e.g. z step 0.25 m), raycast DOWN onto the path layer, and
   **average the hit Ys** = path centreline height at that X. Collect `(X, Y)`.
   This captures the grade regardless of how the path curves in plan.
3. **Smooth** the 1-D profile (moving average ±2 m) → `target(X)`. Report max
   adjacent step as a smoothness/noise gate (abort if a jump implies holes).
4. **Apply with a blend ring.** For each terrain vertex (world space):
   `newY = Lerp(target(X), y0, SmoothStep(0,1, dist/blendDist))` inside a padded core
   box, leaving terrain untouched beyond `blendDist`. **Skip a Z band over the path**
   to preserve the baked path channel. Safety-clamp `|newY - y0|` (skip + count
   outliers) to survive a bad fit.
5. **Never mutate the source FBX.** Temporarily set `ModelImporter.isReadable = true`
   + reimport to read verts; build a **new** `Mesh` asset (preserve vertex
   colors/colors32, every UV channel at its real dimension, submeshes, indexFormat,
   tangents); `RecalculateNormals/Bounds`; re-point `MeshFilter`+`MeshCollider` under a
   single `Undo` group; restore `isReadable = false`. Do **not** auto-save the scene.

## Why this works
- A **fixed cross-line raycast** (e.g. always z = -81) only hits a meandering path
  where the curve happens to cross that line — here a 50-point line scored 12 hits and
  tripped the "<15 samples" abort. Per-X centreline sampling scored 78 clean hits with
  zero coverage gaps.
- A **single least-squares plane** can't sit flush with a non-linear grade: this road
  was ~flat in the west then ramped ~0.9 m up east, leaving a plane **0.24 m off** the
  road (RMS 0.13). A per-X smoothed profile stays flush everywhere (4 cm/m, seams
  mostly ≤10 cm).

## Trade-offs
- The result is **not a single geometric plane** — it keeps the path's gentle curve.
  Fine when the goal is "flush with the path"; pick the plane variant if the brief is
  literally "one flat plane" and you can accept the residual.
- A **straight preservation band over a meandering path** leaves variable-width
  shoulders: the seam at the band's *far* edge can reach ~0.2 m where the path hugs the
  opposite side of the band (worst case seen at the path's extreme end). Mitigate by
  making the preserved band follow the path centreline, or widen the blend.

## Variants
- **Linear plane**: fit `y = m·x + b` over the per-X samples; gate on max residual
  (<0.10 m) and abort/relax if the grade isn't actually linear. Cleanest geometry.
- **Per-X smoothed profile** (this pattern): flush with a gently non-linear grade.
- **Full 2-D conform**: if the path curves strongly in plan, sample `target(x,z)` from
  a nearest-path lookup instead of a 1-D X profile.

## Safety scaffolding that paid off
Timestamped scene backup + MD5 of the FBX **before and after** (proved the binary was
untouched — only `.meta isReadable` flips true→false). One atomic, abort-guarded editor
script (road-fit/coverage gate first) so nothing is modified unless the path is
measurable. Edit-Mode-only guard as the script's first line.
