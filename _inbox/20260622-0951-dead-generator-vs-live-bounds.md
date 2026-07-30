---
title: Hardcoded bounds in a (possibly dead) generator are NOT the map's real size — use the live scene
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-06-22'
project: Kerf - Sawmill Tycoon
tags:
- unity
- scene
- terrain
- dead-code
- bounds
- source-of-truth
applies_to: []
source: ''
severity: medium
time_lost: avoided — caught during recon
suggested-category: engine/lessons
---

# Hardcoded bounds in a (possibly dead) generator are NOT the map's real size — use the live scene

## Problem
Building a perimeter wall around the map. A `LowPolyTerrainGenerator.cs` script defined `terrainWidth = 400 / terrainDepth = 400` and even spawned its own 4 barrier colliders at x/z 0..400. Documentation elsewhere said "600×650m". Two different "truths", and neither matched reality: gameplay objects (player, sawmill) sat at NEGATIVE Z, impossible under a 0..400 generator.

## Root cause
The procedural generator was **dead code** — never referenced/instantiated in the live scene. The live terrain was an imported FBX placed as a prefab instance, NOT centered at origin. Its real world extent (read from `Renderer.bounds`) was X[−350..250], Z[−150..500] = 600×650m. The generator's constants were a leftover from an abandoned procedural approach.

## Solution
- Compute geometry from the LIVE object at build time: `MeshRenderer.bounds` (world-space AABB), cross-checked against `MeshFilter.sharedMesh.bounds` × `localToWorldMatrix`.
- Confirm the suspect script is dead: grep the scene for its component GUID/class — zero hits = it never runs. Leave it untouched (separate cleanup decision), but never treat its constants as truth.

## What didn't work
- Trusting code constants (`terrainWidth`) — wrong by 200m and wrong origin assumption.
- Trusting prose documentation ("600×650") — happened to be right on size but gave no origin/offset, and contradicted the code.

## Transferability
Any Unity project that has accumulated multiple terrain/level-gen approaches: the live imported asset's `Renderer.bounds` is ground truth; serialized fields on dead MonoBehaviours and stale docs are not. Generalises to any "where/how big is X" question — query the live object, don't read a constant. Before deriving geometry from a manager's fields, verify that manager is actually in the active scene.

## Related
- Pattern: invisible map boundary from live Renderer.bounds + foot-only IgnoreCollision gate (same session).
