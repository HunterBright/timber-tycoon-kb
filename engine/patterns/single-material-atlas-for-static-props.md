---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [blender, materials, atlas, static-props, unity, slot-order]
date: 2026-05-17
status: draft
---

# Single-Material Atlas for Static Props

## When to use
Multi-material FBX assets have unpredictable Unity material slot order — after reimport, random materials may be assigned to wrong surfaces. For static props with distinct color regions (PelletBag, FertilizerBag, FirewoodBundle, PelletBox), use a manual single-material atlas instead.

## Steps
1. In Blender Python: create atlas PNG with numpy — composite source PNGs and solid color blocks into one image
2. Map UV per region explicitly: per-face UVs pointing to the correct atlas region
3. Use a single material slot on the mesh from the start
4. Export FBX (only one material to assign → zero ambiguity)
5. In Unity: assign the single material with the atlas PNG as Base Map

Reserve procedural multi-material setups for objects with dynamic state changes (machine on/off, fill levels).

Static prop candidates: PelletBag, PelletBox, FertilizerBag, FirewoodBundle — any prop where visual regions are fixed.

## Why this works
FBX material slot order is non-deterministic in Unity. With a single slot, there's no ordering to get wrong. The atlas packs all visual regions into one texture, one material, zero ambiguity.

## Trade-offs
Explicit UV mapping per face in Python is more setup work than Smart UV Project + multi-material. But multi-material FBX = hours of debugging "why are my bags black/wrong color" across reimports.

## Variants
- [Cycles bake for solid colors anti-pattern](../anti-patterns/cycles-bake-for-solid-colors.md) — what NOT to do instead
- For hero assets with complex shading: multi-material is fine IF the slot order is locked via FBX importer remap scripts
