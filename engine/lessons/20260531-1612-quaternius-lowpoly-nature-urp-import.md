---
title: Importing Quaternius "Stylized Nature MegaKit" (and similar low-poly packs) into URP
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-05-31'
project: Kerf - Sawmill Tycoon
tags:
- unity
- urp
- quaternius
- megakit
- foliage
- vertex-color
- shader
- fbx-import
- alpha
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Importing Quaternius "Stylized Nature MegaKit" (and similar low-poly packs) into URP

Established empirically while importing the free MegaKit (CC0). Likely applies to other
Quaternius low-poly nature/asset packs.

## Pack layout
Ships parallel folders: `FBX (Unity)`, `FBX`, `OBJ`, `glTF`, `Textures`. Use **only**
`FBX (Unity)` + `Textures`; the rest are duplicates for other engines. (Watch Windows paths
with `[brackets]` - PowerShell treats them as wildcards; use `-LiteralPath`.)

## Materials: NO magenta in URP (Unity 6)
Unity 6 + active URP imports the FBX-embedded materials **as URP/Lit automatically**, already
bound to the correct textures by name. So the classic "Standard → magenta in URP" does NOT
happen for this pack. Always **probe 3 models first** to confirm before bulk-importing 60+.
Texture binding: foliage materials reference the `_C` ("colored") texture variants as albedo
(e.g. `Leaves_NormalTree_C`); the non-`_C` versions are masks/AO, not the albedo.

## The two traps (cost the most time)
1. **Foliage is SOLID low-poly geometry, not alpha cards.** The diffuse PNGs carry an alpha
   channel but `alphaIsTransparency = FALSE` (and some, e.g. `Grass`, have no alpha at all).
   That alpha is junk/near-zero. If you enable alpha-clipping (`clip(alpha - 0.5)`), **all
   foliage vanishes**. → Render foliage **opaque, double-sided (Cull Off); do NOT alpha-clip.**
2. **Vertex colors are grayscale AO shading** (r=g=b), present on most flora (e.g. grass
   `0.0-1.0`, fern `0.02-0.98`), but some meshes are uniform white (unaffected). URP/Lit
   **ignores** vertex colors → foliage looks flat/pale. Multiply `albedo = texture * vertexColor`
   in a custom lit shader to restore the baked shading. Because the vcolor is grayscale it only
   darkens - it never re-tints, so a re-textured model (e.g. red→green bush) stays the new color.

## Recommended setup
- Share materials **by texture**, not per model (SRP-batches well; use the model importer's
  `AddRemap(SourceAssetIdentifier(typeof(Material), srcName), sharedMat)` + `SaveAndReimport`).
  AddRemap was reliable here in Unity 6 (despite its historical flakiness).
- Foliage → a hand-written URP lit shader (texture × vertex color, Cull Off, opaque, no clip).
  Keep an `_AlphaClipEnable` toggle (default off) for genuine alpha-card packs.
- Opaque props (rocks, pebbles, mushrooms) → plain URP/Lit. Set `*_Normal.png` to NormalMap
  import type.
- Vert cost (this pack): bushes ~2300 v, mushrooms ~1180, grass ~450, flowers ~314,
  pebbles ~123. Bushes dominate - keep them sparse.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260728-0910-urp-jednostronne-kartki-listowia-czarne|Jednostronne kartki listowia na URP/Lit wychodza CZARNE]] - wspolne: foliage, shader, urp
- [[20260710-1300-vertex-colors-vs-basecolor-linear|Vertex colors vs _BaseColor w Linear color space - ten sam hex renderuje się INACZEJ]] - wspolne: vertex-color, shader, urp
- [[20260720-0940-dwa-niezalezne-bledy-gamma-w-jednym-modelu|"Wszystko za jasne" po podmianie modelu: dwa niezależne błędy gamma dające ten sam objaw]] - wspolne: vertex-color, urp
- [[20260605-1250-urp-flow-shader-scroll-sign|Scrolling/flow shaders: visual motion runs OPPOSITE to the flow vector]] - wspolne: shader, urp
- [[20260702-2140-shader-property-stale-serialized-material-values|Dodanie property do shadera może aktywować STARE, ukryte wartości w materiałach]] - wspolne: shader, urp
- [[20260606-0615-meandering-river-flow-map-baked-tangent|Curved/meandering water flow via a baked flow map (arc-length V + per-vertex tangent)]] - wspolne: vertex-color, shader
<!-- /POWIAZANE:auto -->
