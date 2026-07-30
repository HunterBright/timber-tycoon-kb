---
title: Normalize Inconsistent Asset-Pack Scale at the Source (ModelImporter.globalScale)
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-05-31'
project: Kerf - Sawmill Tycoon
tags:
- unity
- fbx
- modelimporter
- scale
- normalization
- asset-pack
- scatter
- globalScale
applies_to: []
source: ''
suggested-category: assets/lessons
---

# Normalize Inconsistent Asset-Pack Scale at the Source (ModelImporter.globalScale)

## Problem
Free/marketplace model packs (Quaternius Stylized Nature MegaKit here) ship with **wildly
inconsistent real-world scale**. Measured base heights from one "nature" pack: ferns 2.69 m,
flowers 2.0–2.5 m, clover 1.1–1.3 m, `Plant_1_Big` **3.76 m** — all taller than the 1.87 m
player — while petals/pebbles were 0.1–0.3 m. Scattering them as-is looks absurd (knee-high
plants rendering as tree-sized). Fixing scale per-instance in the scatter is fragile and
non-reusable.

## Fix: bake a target height into each FBX at import
1. **Measure** each model's true height: instantiate at scale 1, read combined
   `Renderer.bounds.size.y`, destroy.
2. Pick a **target height per category** (realistic vs the character: grass 0.3–0.6, bush ~1.15,
   fern 0.5, mushroom 0.2, decorative rock ~1.0, pebble ~0.22, etc.).
3. `mult = target / measured;` then `modelImporter.globalScale *= mult; importer.SaveAndReimport();`
   This is the "Scale Factor" field — asset-level, so EVERY future instance/prefab is correct.
4. **Re-measure after reimport** and flag any model off target by >5%.

## Why this is robust
- **Idempotent**: because `mult` is recomputed from the *current* measured height each run, a
  second pass measures ≈target → mult≈1 → no change. Safe to re-run.
- **Asset-level**: instances come in at the right size; a scatter tool then only needs a small
  ±15% jitter (`localScale` on top) for variety.
- Worked for all 38 models via `globalScale` with **0 fallbacks** (Unity 6). Keep a fallback:
  if `globalScale` is ineffective for some model, bake the residual multiplier into a normalized
  **prefab** (`PrefabUtility.SaveAsPrefabAsset` with the root scaled) and have consumers prefer
  that prefab over the raw FBX.

## Process note
Do this as an **approval-gated** step: measure → propose a target/multiplier table → show a
scale-comparison render of one model per category next to a known-height reference (the game's
NPC, or an exact-height pole) → get sign-off → bake → re-scatter. Scaling blind looks wrong;
the human eye on a side-by-side vs a character is the real check.
