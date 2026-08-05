---
title: MaterialPropertyBlock for Runtime Color Variants
type: pattern
status: needs-reproduction
confidence: low
verified: '2026-08-05'
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- unity
- materials
- runtime
- color
- performance
- mpb
applies_to: []
source: ''
suggested-category: engine/patterns
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
- MPB doesn't survive scene reload - re-apply in `Start()` / on spawn
- For save/load: persist the color as a `Color` field in ISaveable, re-apply from saved data on load
- Separate body renderer (`Mat_Body`) from trim/glass/wheels - only body color varies, reducing MPB calls

Convention in TT: `CarVariantData SO` stores 8 body colors, assigned randomly at spawn via MPB.

## Why this works
MPB overrides shader properties per-renderer without creating new material instances. GPU batching still works (same material asset), VRAM stays minimal (1 shared material, N runtime overrides).

## Trade-offs
MPB overrides must be re-applied after any renderer update (e.g., after LOD switch, after material assignment from script). Slightly more runtime code than static materials.

## Variants
Same pattern for: emission pulse (machine state glow), health-based tinting (damage indication), season tints (leaf color per season), faction color (RTS units).

<!-- WERYFIKATOR 2026-08-05 -->
Weryfikacja 2026-08-05: zdanie "GPU batching still works (same material asset)"
jest sprzeczne z dokumentacja Unity dla URP. Unity pisze, ze aby obiekt byl zgodny
z SRP Batcherem, "The GameObject mustn't use MaterialPropertyBlocks", a osobna strona
mowi wprost, ze dodanie MaterialPropertyBlocku to sposob, zeby **zabrac** obiektowi
zgodnosc z SRP Batcherem. To samo dotyczy GPU Resident Drawera w Unity 6, ktory
obsluguje tylko obiekty majace "no MaterialPropertyBlocks set". Sam mechanizm nadpisywania
wlasciwosci bez tworzenia kopii materialu jest opisany poprawnie - falszywy jest tylko
wniosek o wsadowaniu, a to on jest w tym wpisie uzasadnieniem calego wzorca.
Zrodla: https://docs.unity3d.com/6000.3/Documentation/Manual/SRPBatcher-Materials.html ,
https://docs.unity3d.com/6000.0/Documentation/Manual/SRPBatcher-Incompatible.html ,
https://docs.unity3d.com/6000.5/Documentation/Manual/urp/gpu-resident-drawer.html
