---
title: 'Procedurally generated boundary: align to a visible region + auto-gap at roads'
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-06-25'
project: Kerf - Sawmill Tycoon
tags:
- unity
- editor-tool
- procedural
- collider
- layermask
- boundary
- alignment
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Procedurally generated boundary: align to a visible region + auto-gap at roads

## Problem
You need to place a generated structure (fence, wall, marker ring) around an area that the player perceives visually (a clearing, a sparser-grass zone, a plot), and it must not block roads/paths crossing it.

## Pattern

**1. Don't guess the footprint - replicate the region that already defines the visual.**
If another system already computes the bounds of that visual region, replicate ITS computation rather than inventing coordinates. In Timber Tycoon the "cutting zone" (sparser grass) is computed by the scatter tool (`DecorFloraScatter.GatherHarvestTrees`): centers of tree MeshRenderers with `bounds.size.y > 2.5` under named parents, min/max, then directional padding (W/E/S/N). The fence tool replicates exactly that → fence edge lands on the grass boundary 1:1. Guessing padding/edges cost 3 wrong iterations; replicating the source got it right immediately.

**2. Compute from live `Renderer.bounds` (world), never from assumed transform coords.**
A "Zone" GameObject had `localPosition (50,0,0)` but a parent offset it to world ~(95,-88). Reading the local position as world (from the scene YAML) gave coordinates ~45m off. Building from `GetComponentsInChildren<Renderer>().bounds` (world-space, encapsulated) is parent-agnostic and self-correcting.

**3. Mirror a proven generator instead of writing fresh.**
A near-identical tool existed (`BuildMapBoundary.cs`: rectangle-from-bounds, edge gap math, `EnsureLayer`, idempotent root rebuild, Play-Mode guard, config-SO, no auto-save). Mirroring it inherited all the safety/idempotency for free.

**4. Auto-gap by detecting the surface's existing layer with `Physics.CheckSphere`.**
Roads in the project carry MeshColliders on layer `Road` (the scatter tool already uses `roadMask = 1<<6` to avoid seeding grass on them). The fence reuses the SAME signal: per segment, `Physics.CheckSphere(midpoint, clearance, roadMask)` → if a road is there, skip that segment → the gap lands exactly where any road crosses, on any edge, with zero hardcoded coordinates.

## GOTCHA - paired invisible colliders
Visuals and colliders were separate GameObjects (collider = a box, visual = the rotated/scaled mesh, kept separate to avoid scale/rotation distortion). Consequence: leaving a gap (or hand-deleting fence pieces on a road) must remove the collider too - otherwise an **invisible wall** remains and silently blocks vehicles, which is worse than a visible block. Auto-gap handles both together; manual editing does not (Scene-view clicks select the visible mesh, not the invisible collider). If manual deletion is required, either parent collider+visual under one `[SelectionBase]` container, or warn the user explicitly.

## When to reuse
Any "enclose/mark this visual area" editor tool; any generated boundary that must respect crossing paths. Detecting an existing layer beats geometric path-intersection math.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260531-1614-editor-flora-scatter-patterns|Reproducible Editor Flora Scatter onto a Mesh Terrain]] - wspolne: editor-tool, procedural
- [[20260706-1520-navmesh-raised-collider-invisible-bump|NPC chodza po "niewidzialnych gorkach": bake NavMesh z propsow + za gruby voxel nad niskopoly terenem]] - wspolne: layermask, collider
<!-- /POWIAZANE:auto -->
