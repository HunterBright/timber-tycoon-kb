---
title: Vehicle Interaction Zones as Triggers
type: pattern
status: draft
confidence: medium
verified: ''
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- unity
- vehicle
- colliders
- triggers
- interaction
applies_to: []
source: ''
suggested-category: engine/patterns
---

# Vehicle Interaction Zones as Triggers

## When to use
Multi-part vehicles where different surfaces have different interaction behaviors (cabin = enter, flatbed = load/unload). Use when you need both physics collisions AND interaction zones on the same vehicle.

## Steps
1. Vehicle root: solid BoxCollider for physics (collision with ground, walls, NPCs)
2. Cabin child: trigger BoxCollider + `VehicleInteractable` script (E to enter). Layer: `Interactable`
3. Flatbed child: trigger BoxCollider + `FlatbedInteractable` script (E to load/unload). Layer: `Interactable`
4. Both children on layer `Interactable` (layer 3) for player raycast
5. Triggers fire `OnTriggerEnter` for interaction detection - no physics response

Auto-setup via Editor: `Tools > Setup All Vehicles` configures from FBX naming convention.

## Why this works
Triggers ignore physics (no bounce/collision) but still detect overlap events. Using a separate solid collider for physics and separate trigger colliders for interaction gives clean separation of concerns.

## Trade-offs
Requires correct layer assignment for raycast filtering. More collider objects to manage (but editor script handles setup).

## Variants
Same pattern for: interactable machine panels (solid collider for player collision + trigger for E-interaction), storage rack zones (player triggers deposit animation), water zones (trigger for splash + wading speed reduction).

See also: [[vehicle-enter-exit-choreography]] for the full enter/exit sequence.
