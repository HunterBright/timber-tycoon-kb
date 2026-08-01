---
title: URP Shadow Cascade Tuning for Low-Poly Terrain
type: lesson
status: needs-reproduction
confidence: low
verified: '2026-08-01'
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- unity
- urp
- shadows
- performance
- road-artifacts
- lighting
applies_to:
- unity-projects
source: ''
severity: high
suggested-category: engine/lessons
time_lost: ''
---

# URP Shadow Cascade Tuning for Low-Poly Terrain

## Problem
Default URP shadow cascade settings produce visible sharp transition lines across roads and terrain where cascade boundaries fall. On flat terrain, these artifact lines are highly visible and break immersion.

## Root cause
Default URP Asset: Cascade Count=4, Max Distance=50. With 4 cascades over 50m, cascade boundaries land frequently across the playable area. Low-poly terrain with road surfaces amplifies the artifact because the flat geometry has no detail to hide transitions.

## Solution
- Cascade Count: 4 → **2**
- Last Cascade Border: 4 → **10-15**
- Max Distance: 50 → **80-100**
- Increase Depth Bias and Normal Bias on Directional Light to compensate

Trade-off: slight reduction in shadow detail for close-up objects; massive improvement for road quality. For a ground-level FPP game on flat-ish terrain, this trade-off is always worth it.

## What didn't work
Default settings (4 cascades, 50m) - visible artifact lines across roads.

## Transferability
Any URP project with flat or gently sloping terrain (racing, farming sim, city builder, tycoon, walking sim) will benefit from 2-cascade tuning. 4 cascades are designed for games with complex height variation; flat terrain amplifies cascade boundary artifacts.

> **Weryfikacja 2026-08-01: podane wartosci domyslne sa sprzeczne ze zrodlem.**
> W kodzie zasobu URP domyslnie `m_ShadowCascadeCount = 1` (nie 4) i `m_CascadeBorder = 0.2`
> (nie 4); zgadza sie tylko `m_ShadowDistance = 50`. Dokumentacja opisuje "Last Border"
> jako szerokosc pasa wygaszania cienia przy koncu zasiegu, a nie granice miedzy kaskadami
> ([kod zasobu URP](https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.universal/Runtime/Data/UniversalRenderPipelineAsset.cs),
> [opis zasobu URP](https://docs.unity3d.com/6000.3/Documentation/Manual/urp/universalrp-asset.html)).
> Wartosc 4 pochodzi prawdopodobnie z gotowego szablonu projektu URP, nie z ustawienia domyslnego.

## Related
- [4-phase weighted smoothstep day/night](../patterns/four-phase-weighted-smoothstep-day-night.md)
- [Minecraft-style lighting](../../genre/tycoon/patterns/minecraft-style-lighting.md)

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[urp-distant-caster-shadow-band|"Dark band that follows the player" = terrain self-shadow leaking onto near-coplanar road meshes]] - wspolne: shadows, urp
- [[20260613-1200-unity-offscreen-render-rig-scene-leaks|An "isolated" offscreen render rig still inherits the open scene's lights AND skybox reflection]] - wspolne: lighting, urp
- [[stale-reflection-probe-night-whitening|Stale Skybox Reflections Whiten PBR Materials at Night (Day/Night Cycle)]] - wspolne: lighting, urp
- [[20260623-1508-instanced-grass-cards|Performant stylized grass: textured cards + GPU instancing (no GameObjects)]] - wspolne: performance, urp
<!-- /POWIAZANE:auto -->
