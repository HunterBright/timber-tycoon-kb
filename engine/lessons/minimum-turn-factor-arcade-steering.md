---
title: Minimum turnFactor 0.3 for Low-Speed Arcade Steering
type: lesson
status: draft
confidence: medium
verified: ''
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- unity
- vehicle
- physics
- arcade
- steering
- npc
applies_to:
- unity-projects
source: ''
severity: medium
suggested-category: engine/lessons
time_lost: ''
---

# Minimum turnFactor 0.3 for Low-Speed Arcade Steering

## Problem
Speed-dependent steering produces a turnFactor of 0 at zero speed - the vehicle cannot rotate to escape a stuck state. NPCs get permanently stuck against walls/curbs. Player's car feels "dead" at low speed.

## Root cause
Common arcade steering pattern: `turnFactor = Mathf.Clamp01(speed / maxSpeed)` - gives no steering authority at standstill. Edge case: vehicle stuck at speed=0, turnFactor=0, cannot rotate.

## Solution
Floor the turnFactor at 0.3:
```csharp
turnFactor = Mathf.Max(0.3f, Mathf.Clamp01(speed / maxSpeed));
```

0.3 = enough to rotate in place slowly; not enough to feel like full power steering. Applies to both player `VehicleController` and NPC vehicles.

## What didn't work
`Mathf.Clamp01(speed / maxSpeed)` without floor - stuck vehicles can't escape.

## Transferability
Any arcade vehicle game with speed-dependent steering. The 0.3 floor is a tuned heuristic; adjust to taste for different vehicle feels. The underlying principle (always allow some minimum steering authority) applies universally.

## Related
- [Forward axis quirk](forward-axis-blender-fbx-quirk.md)
- [NPC parking PD controller](../../genre/tycoon/patterns/npc-parking-pd-controller.md)

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[freeze-inertia-tensor-not-restored|FreezeAll + automaticInertiaTensor=false Zeroes the Inertia Tensor]] - wspolne: vehicle, npc, physics
- [[npc-parking-pd-controller|NPC Parking PD Controller]] - wspolne: vehicle, npc, physics
- [[20260714-2220-maxspeed-clamp-is-not-a-speed|maxSpeed to KLAMRA, nie prędkość - pojazd i tak stanie na (napęd / tłumienie)]] - wspolne: vehicle, physics
- [[20260714-2350-null-physics-material-silently-becomes-default-friction|Collider z materiałem `null` NIE ma zerowego tarcia - ma DOMYŚLNE 0.6]] - wspolne: vehicle, physics
- [[20260724-0907-arcade-car-climbs-walls|Arkadowe auto na Rigidbody wspina sie po scianach i odlatuje w niebo]] - wspolne: vehicle, physics
- [[self-collision-compound-colliders-ignore|Self-Collision Compound BoxColliders → Physics.IgnoreCollision]] - wspolne: vehicle, physics
<!-- /POWIAZANE:auto -->
