---
type: lesson
project: timber-tycoon
suggested-category: engine/lessons
tags: [unity, vehicle, physics, arcade, steering, npc]
severity: medium
time_lost: ""
date: 2026-05-17
status: draft
applies_to: ["unity-projects"]
---

# Minimum turnFactor 0.3 for Low-Speed Arcade Steering

## Problem
Speed-dependent steering produces a turnFactor of 0 at zero speed — the vehicle cannot rotate to escape a stuck state. NPCs get permanently stuck against walls/curbs. Player's car feels "dead" at low speed.

## Root cause
Common arcade steering pattern: `turnFactor = Mathf.Clamp01(speed / maxSpeed)` — gives no steering authority at standstill. Edge case: vehicle stuck at speed=0, turnFactor=0, cannot rotate.

## Solution
Floor the turnFactor at 0.3:
```csharp
turnFactor = Mathf.Max(0.3f, Mathf.Clamp01(speed / maxSpeed));
```

0.3 = enough to rotate in place slowly; not enough to feel like full power steering. Applies to both player `VehicleController` and NPC vehicles.

## What didn't work
`Mathf.Clamp01(speed / maxSpeed)` without floor — stuck vehicles can't escape.

## Transferability
Any arcade vehicle game with speed-dependent steering. The 0.3 floor is a tuned heuristic; adjust to taste for different vehicle feels. The underlying principle (always allow some minimum steering authority) applies universally.

## Related
- [Forward axis quirk](forward-axis-blender-fbx-quirk.md)
- [NPC parking PD controller](../../genre/tycoon/patterns/npc-parking-pd-controller.md)
