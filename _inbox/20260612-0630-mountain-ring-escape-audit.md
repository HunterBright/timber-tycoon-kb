---
type: pattern
project: Timber Tycoon
suggested-category: engine/patterns
tags: [unity, collision, level-design, mesh-sampling, map-boundary, editor-script]
date: 2026-06-12
status: draft
---

# Programmatic escape audit for mountain-ring map boundaries

## Problem
Maps bounded by a ring of mountain meshes (especially AI-generated: Tripo/Hunyuan) look impassable but often have lower slopes below the player's `CharacterController.slopeLimit`. Manual play-testing every approach angle is slow and misses spots.

## Pattern
Editor script that audits the whole ring in seconds instead of hours of play-testing:

1. Collect all mountain `MeshRenderer`s; add non-convex static `MeshCollider`s (fine for static scenery).
2. For each mesh, iterate triangles in **world space** (TransformPoint), compute face normal angle vs `Vector3.up`. Stride-sample if tri count is high (`stride = max(1, triCount/20000)`).
3. Filter to **inner faces**: horizontal normal component pointing toward the ring centroid (mean of all renderer bounds centers), dot > 0.1.
4. Filter to **reachable bands**: raycast down (terrain layer only) for ground height; classify faces into LOW band (0–6 m above ground = direct reach incl. jump) and MID band (6–20 m = continuation upward).
5. Classify: climbable (< slopeLimit), hop-risk (slopeLimit..~55°, jump-spam can beat it). A mountain with climbable faces in BOTH bands = escape risk (continuous path up).
6. Output per-mountain counts + worst-spot world coordinates → player teleports there to verify.

## Key gotchas
- Read actual `slopeLimit`/`jumpForce` from the **scene** components, not script defaults (Timber Tycoon: script default jumpForce=7, scene had 5 → jump 0.63 m, not 1.22 m).
- Where the down-raycast misses terrain (mountain base beyond terrain edge / buried below), ground-height proxy = bounds.min.y produces false positives — flag, don't trust blindly.
- Adding colliders does NOT invalidate a baked NavMesh (colliders only matter at bake time) — no rebake needed.
- In Editor, mesh vertex/triangle access works even without Read/Write enabled (player-only restriction).

## Result in Timber Tycoon
24 of 29 ring mountains had climbable inner low slopes (many near 0–20°). Conclusion: AI-generated mountains have gentle aprons; an invisible wall ring is usually needed regardless — the audit tells you whether and where.
