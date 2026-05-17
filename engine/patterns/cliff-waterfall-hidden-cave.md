---
name: cliff-waterfall-hidden-cave
description: Waterfall (no collider) hides cave entrance behind it — player walks through to discover cave
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: engine/patterns
  tags: [unity, level-design, secret, waterfall, cave, player-discovery]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Cliff + Waterfall Hidden Cave Pattern

## When to use
Any map with a waterfall or curtain effect where you want a "free" exploration moment — player discovers hidden space by walking through the visual barrier.

## Steps

**Two-mesh split:**
- `Cliff_Waterfall.fbx` (640v): cliff rock face with internal cave carved out. Has `MeshCollider` — player walks on rock, collides with cliff.
- `Waterfall.fbx` (140v): water curtain hanging in front of cave entrance. Has **NO collider** — player walks through it.

**Cave design:**
- Cave mouth behind the waterfall curtain, visible only after stepping through water
- Interior: dome-shaped 12×8×17.5m space, vertex colors average (0.43, 0.39, 0.32) linear
- Lighting: dark except for light entering through cave mouth and waterfall light scatter

**Waterfall details:**
- Inverted-L shape: 8m horizontal ledge + 45m vertical drop, 14m wide
- `Custom/Waterfall` shader: vertex color animation (white foam top/bottom, teal water center), `Cull Off`
- No collider on waterfall mesh — critical, don't accidentally add one

**Player discovery flow:** approach cliff → walk into waterfall (visual pass-through) → find cave interior → "Oh, there's a secret here."

**Position:** both pieces are manually positioned by designer and hardcoded. See [[project-cliff-position]] (project memory) for exact Unity coordinates.

## Why this works
The waterfall is purely visual — mesh without collision. Player physically can't be blocked. Cave is hidden from all approach angles. Cost: 2 meshes, 1 shader, 0 scripts.

## Trade-offs
- Visible on minimal distance: at extreme viewing angles, cave opening might be visible before the player reaches the waterfall — acceptable in low-poly style
- Single-use pattern: hardcoded positions. For a procedural/reusable version, abstract into a prefab with configurable cave orientation
- Waterfall must be `Cull Off` — if materials ever get reset, waterfall becomes solid or invisible from one side

## Variants
- **Fog density wall:** same principle with a fog plane instead of a waterfall
- **Dense foliage:** hanging vines with no collider, cave behind them
