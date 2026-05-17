---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, materials, runtime, color, performance, mpb]
date: 2026-05-17
status: draft
---

# MaterialPropertyBlock for Runtime Color Variants

## When to use
Multiple instances of the same prefab need different colors (NPC car variants, tree species leaf tints, faction colors). Naive approach: duplicate material per color variant = N materials × M renderers = memory waste.

## Steps
```csharp
MaterialPropertyBlock mpb = new MaterialPropertyBlock();
mpb.SetColor("_BaseColor", carColor);
bodyRenderer.SetPropertyBlock(mpb);
```

Important caveats:
- MPB doesn't survive scene reload — re-apply in `Start()` / on spawn
- For save/load: persist the color as a `Color` field in ISaveable, re-apply from saved data on load
- Separate body renderer (`Mat_Body`) from trim/glass/wheels — only body color varies, reducing MPB calls

Convention in TT: `CarVariantData SO` stores 8 body colors, assigned randomly at spawn via MPB.

## Why this works
MPB overrides shader properties per-renderer without creating new material instances. GPU batching still works (same material asset), VRAM stays minimal (1 shared material, N runtime overrides).

## Trade-offs
MPB overrides must be re-applied after any renderer update (e.g., after LOD switch, after material assignment from script). Slightly more runtime code than static materials.

## Variants
Same pattern for: emission pulse (machine state glow), health-based tinting (damage indication), season tints (leaf color per season), faction color (RTS units).
