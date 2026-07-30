---
title: Wavy dark lines in EEVEE preview renders = shadow acne, not geometry
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-06-12'
project: Timber_Tycoon
tags:
- blender
- eevee
- rendering
- shadow-acne
- debugging
- preview-renders
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Wavy dark lines in EEVEE preview renders = shadow acne, not geometry

## Symptom
Thin wavy black lines running across a large flat surface (cork board panel) in Blender EEVEE preview renders. Looked exactly like stray texture scribbles or loose geometry edges crossing the surface.

## Root cause
Shadow acne: a leftover Sun light from a previous session hit the surface at a near-grazing angle. EEVEE's shadow map self-shadowing produces wavy banding patterns on faces nearly parallel to the light direction. The waviness mimics hand-drawn lines, which misleads diagnosis toward texture/UV bugs.

## Diagnosis method (transferable)
Before touching the model, verify programmatically - 2 minutes rules out whole classes of causes:
1. Inspect the texture/atlas image directly (is the artifact in the texture? No → not a texture bug).
2. Dump per-face data via bmesh/mesh API: loose edges (`not e.link_faces`), loose verts, per-face UV bboxes vs intended atlas regions, oversized faces sampling wrong regions. All clean → not geometry/UV.
3. What remains is lighting/render: re-aim the light frontally (or toggle shadows) and re-render. Lines gone → shadow acne confirmed.

## Fix
For preview renders: aim the key light at the surface at a healthy angle (≥ ~30° incidence), or raise shadow bias. The artifact is preview-only - the game engine with its own lighting/bias setup won't reproduce it, so do NOT modify the model.

## Why it matters
Without the programmatic check, the obvious move is rebuilding geometry/UVs that were never broken. Cheap verification first, model surgery last.
