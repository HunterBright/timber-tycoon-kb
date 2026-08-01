---
title: Blender-rendered 9-slice-ready UI sprites (3D panel → ortho render → Unity Sliced sprite)
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-06-12'
project: Kerf - Sawmill Tycoon
tags:
- blender
- ui
- 9-slice
- sprite
- unity
- render-to-sprite
- low-poly
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Blender-rendered 9-slice-ready UI sprites (3D panel → ortho render → Unity Sliced sprite)

## When to use
Stylized game UI (wood, stone, metal frames) where flat vector art looks too sterile and you want real lit 3D relief - but the sprite must survive Unity's 9-slice stretching for arbitrary window sizes.

## Steps
1. **One master .blend per kit**, one collection per asset. Shared rig: orthographic camera straight down (-Z), known `ortho_scale`; soft Sun key (~15° softness, slight tilt for relief) + shadowless fill + low ambient; `film_transparent = ON`; view transform **Standard** (palette-true flat colors); EEVEE.
2. **Design the silhouette for 9-slice from the start**: pick the corner-zone boundary first (e.g. outer 20% of each edge). Middle ~60% of each edge gets only low-amplitude, **integer-period** sine waviness + fine per-vertex jitter (stretches/tiles invisibly). All unique large bumps, notches and decorations (knots etc.) go in corner zones only. Center surface: clean, no unique features - it stretches both ways.
3. Build geometry procedurally with a **seeded** Python script (deterministic rebuilds, easy iteration on parameters).
4. **Stretch-test before approval**: duplicate the meshes and replay Unity's Sliced math on vertices (`|x| ≤ t → x·(t+d)/t`, else `x ± d`), render second image. This proves midsections hold up *before* anything reaches the engine.
5. Compute Unity border pixels analytically: `px_per_unit = render_width / ortho_scale`; border = distance from texture edge to corner-zone boundary in px. No guessing in the Sprite Editor.

## Why this works
Ortho top-down + known ortho_scale makes the render a measurable 2D artifact - world units map linearly to pixels, so 9-slice borders are exact numbers, not eyeballed. Integer-period waves in edge midsections mean linear stretching only changes wavelength, never breaks continuity at the corner-zone seam.

## Trade-offs
- More setup than painting a sprite; pays off only for a *kit* (shared rig amortizes).
- Per-face shading noise must be anisotropic (elongated along grain): isotropic vertex jitter on a fine grid reads as pixel checkerboard at UI scale. Detail elements (grain streaks, knots) work better as separate thin floating geometry than as material-index painting on grid cells (grid cells read as squares).
- Stretching (Unity default) elongates facets; if the kit needs *tiling* instead, midsection jitter must also be periodic.

## Variants
- Same rig renders icons/buttons/frames - new collection, hide others, adjust ortho_scale, keep light identical for kit consistency.
- For non-rect panels (circular badges) the same rule applies radially: unique features in fixed zones, repeatable low-amp noise in stretch zones.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260613-0610-dim-scrim-must-not-reuse-9slice-panel-factory|Don't build a full-screen dim/scrim by reusing your skinnable panel factory]] - wspolne: sprite, 9-slice, ui
- [[20260613-0625-9slice-ppu-must-scale-to-target-rect-not-stay-100|A large 9-slice sprite at PixelsPerUnit=100 breaks because its fixed corners exceed the panel]] - wspolne: sprite, 9-slice, ui
- [[20260618-0724-blender-ortho-ui-sprite-bake-framing|Baking flat UI sprites in Blender: ortho frame width = ortho_scale × 2]] - wspolne: 9-slice, blender
- [[20260726-1535-blender-addon-parametryczny-suwaki|Parametryczny dodatek do Blendera: trzy pulapki, ktore kosztuja godzine kazda]] - wspolne: ui, blender
- [[20260725-2050-kontrakt-liczbowy-bez-nazw-osi|Wspolny kontrakt liczbowy dla kilku agentow, ktory nie nazywa osi]] - wspolne: low-poly, blender
- [[terrain-skirt-against-seethrough-gap|Terrain Skirt Against the See-Through Gap]] - wspolne: low-poly, blender
<!-- /POWIAZANE:auto -->
