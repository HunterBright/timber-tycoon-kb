---
type: lesson
project: timber-tycoon
suggested-category: engine/lessons
tags: [unity, urp, shadows, performance, road-artifacts, lighting]
severity: high
time_lost: ""
date: 2026-05-17
status: draft
applies_to: ["unity-projects"]
---

# URP Shadow Cascade Tuning for Low-Poly Terrain

## Problem
Default URP shadow cascade settings produce visible sharp transition lines across roads and terrain where cascade boundaries fall. On flat terrain, these artifact lines are highly visible and break immersion.

## Root cause
Default URP Asset: Cascade Count=4, Max Distance=50. With 4 cascades over 50m, cascade boundaries land frequently across the playable area. Low-poly terrain with road surfaces amplifies the artifact because the flat geometry has no detail to hide transitions.

## Solution
- Cascade Count: 4 → **2**
- Last Cascade Border: 4 → **10-15**
- Max Distance: 50 → **80-100**
- Increase Depth Bias and Normal Bias on Directional Light to compensate

Trade-off: slight reduction in shadow detail for close-up objects; massive improvement for road quality. For a ground-level FPP game on flat-ish terrain, this trade-off is always worth it.

## What didn't work
Default settings (4 cascades, 50m) — visible artifact lines across roads.

## Transferability
Any URP project with flat or gently sloping terrain (racing, farming sim, city builder, tycoon, walking sim) will benefit from 2-cascade tuning. 4 cascades are designed for games with complex height variation; flat terrain amplifies cascade boundary artifacts.

## Related
- [4-phase weighted smoothstep day/night](../patterns/four-phase-weighted-smoothstep-day-night.md)
- [Minecraft-style lighting](../../genre/tycoon/patterns/minecraft-style-lighting.md)
