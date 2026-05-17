---
name: procedural-textures-cycles-commercial
description: For commercial release, use procedural textures (Wave + Voronoi + Noise → baked PNG) — zero external image files = zero licensing concerns. All TT materials are procedural.
metadata:
  type: lesson
  project: timber-tycoon
  suggested-category: workflow/3d-models
  tags: [blender, cycles, procedural, commercial, licensing]
  severity: high
  date: 2026-05-17
  status: draft
  applies_to: [blender-pipelines]
---

# Procedural Textures in Blender Cycles (Commercial Release Rationale)

## Rule

For any project shipping commercially (Steam, itch.io, App Store): use procedural textures only. Zero external image files = zero licensing concerns.

## The licensing risk

PBR texture packs, even "CC0" or "royalty-free" ones, often have terms:
- Attribution required in credits
- Non-commercial use only
- "Personal use" with unclear commercial interpretation
- License changed retroactively by author

With procedural textures: 100% generated in Blender. No third-party rights. No attribution requirements. Sell the game without legal review of every texture source.

## Standard procedural setups for TT materials

**Wood grain:**
```
Wave Texture (Bands, Scale 5, Distortion 2) →
ColorRamp (brown gradient: #5C3D2E to #8B6245) →
Principled BSDF (Roughness 0.8, Metallic 0)
```

**Concrete:**
```
[Noise Texture (Scale 8)] + [Voronoi (Scale 6)] →
Mix Shader (Fac: 0.4) →
ColorRamp (gray with variation: #888 to #AAA) →
Principled BSDF (Roughness 0.9)
```

**Bark:**
```
[Noise (Scale 3)] + [Voronoi (Scale 8, Distance to Edge)] →
Bump Node (Strength 0.5) →
ColorRamp (dark brown: #3D2B1F to #5A3C28) →
Principled BSDF + Normal input from Bump
```

**Metal (light galvanized):**
```
Voronoi (Scale 20, Distance to Edge) + Noise (subtle, Scale 30) →
Mix → ColorRamp (#8A8A8A to #B5B5B5) →
Principled BSDF (Metallic 0.9, Roughness 0.3)
```

## Export requirement

Procedural shaders don't survive FBX export. Always bake to PNG before exporting:
- Cycles render, Diffuse Color Only
- 512×512 for background props, 1024×1024 for hero assets
- OUtput: `{AssetName}_Bake.png` in adjacent folder

Baked PNG is the "source of truth" for Unity — the procedural nodes are just the generator.

## Bonus: small .blend file

Procedural materials = no embedded PNGs in .blend. A model with 10 procedural materials is ~1-3MB. Same model with baked PNGs embedded would be 10-30MB.

See also: [[blender-headless-python-generation]], [[zero-floating-zero-flickering-mandate]], [[tripo-cleanup-pipeline]]
