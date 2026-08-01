---
title: Tool Viewmodel as Child of Camera
type: pattern
status: draft
confidence: medium
verified: ''
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- unity
- fps
- viewmodel
- camera
- tool
applies_to: []
source: ''
suggested-category: engine/patterns
---

# Tool Viewmodel as Child of Camera

## When to use
FPS-style game where player holds a tool (axe, shovel, weapon) that should always be visible and move with the camera.

## Steps
1. Parent tool viewmodel to `PlayerCamera`:
   ```
   Player → PlayerCamera → ToolViewmodel
   ```
2. Set ToolViewmodel local position/rotation once (right side, slightly below center, angled inward - e.g., localPos (0.3, -0.2, 0.6))
3. Tool auto-follows camera (Unity transform inheritance)
4. Tool swap: `ToolWheel` selects → `ToolViewmodel.Show(ToolType)` enables matching child mesh
5. Each tool = child GameObject, only one active at a time

No per-frame follow code needed - parenting handles it.

## Why this works
Transform inheritance in Unity handles the follow for free. Child transform = parent transform + local offset. No Update() loop, no Lerp, no lag.

## Trade-offs
Tool clips through walls (standard FPS issue). Acceptable for low-poly stylized games; add layer-based render queue for high-fidelity FPS.

## Variants
Same for: flashlight attachment (child of camera), HUD-space elements (child of camera at fixed position), scope overlay (child of camera at z=0.1).

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[stacked-carry-system-camera-viewmodel|Stacked carry system - camera viewmodel + LIFO + species-agnostic prefab refs]] - wspolne: viewmodel, camera
<!-- /POWIAZANE:auto -->
