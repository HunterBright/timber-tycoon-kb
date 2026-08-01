---
title: 'Blender: image.scale() na file-backed image nie trzyma się podczas bake - użyj images.new()'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-04'
project: Kerf - Sawmill Tycoon
tags:
- blender
- bake
- texture
- cycles
- gotcha
- atlas
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Blender: image.scale() na file-backed image nie trzyma się podczas bake - użyj images.new()

## Objaw
Chcąc wypalić istniejący atlas w wyższej rozdzielczości (1024 → 2048), otwarto istniejący
plik-obraz (source = FILE) i wywołano `image.scale(2048, 2048)` przed `bpy.ops.object.bake()`.
Bake wyprodukował mimo to PNG **1024×1024**.

## Root cause
Obraz o źródle FILE podczas bake'u **cicho wraca do rozdzielczości z dysku** - `image.scale()`
na nim nie jest trwałe. Bake pisze do bufora obrazu, ale bufor jest re-alokowany do rozmiaru
pliku źródłowego.

## Fix
Nie re-używać file-backed image do zmiany rozdzielczości. Stworzyć **świeży generated image**
docelowej rozdzielczości i wypalać do niego, potem zapisać na dysk:

```python
atlas_img = bpy.data.images.new("Atlas", width=2048, height=2048, alpha=False)  # generated, nie FILE
# ... przypnij jako aktywny Image Texture node w każdym materiale ...
bpy.ops.object.bake(type="DIFFUSE")
assert tuple(atlas_img.size) == (2048, 2048)   # zweryfikuj PO bake, przed zapisem
atlas_img.filepath_raw = OUT_PATH
atlas_img.save()
```
(To dokładnie robi oryginalny build-script gry - używa `images.new`, nie re-używa pliku.)

## Zabezpieczenie
Zawsze `assert atlas_img.size == (W, H)` bezpośrednio PO `object.bake`, ZANIM zapiszesz PNG -
łapie ciche revert-y rozdzielczości.

## Kontekst
Re-tekstura kompostownika (FertilizerMaker) w Timber Tycoon: podbicie atlasu 1024→2048.
Pierwszy przebieg dał 1024 mimo `scale()`; drugi przez `images.new(2048)` dał poprawne 2048.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260618-0724-blender-ortho-ui-sprite-bake-framing|Baking flat UI sprites in Blender: ortho frame width = ortho_scale × 2]] - wspolne: bake, blender
- [[20260702-1400-bark-atlas-vs-tile-strategy|Rodzina assetów współdzieląca teksturę: wybierz strategię (atlas vs kafel) PRZED budową]] - wspolne: atlas, blender
- [[20260704-2330-blender-unity-flat-panel-dual-face-texture|Blender flat panel textured on one face renders BLANK in Unity (axis-flip picks the wrong face)]] - wspolne: texture, blender
- [[20260725-1930-blender-pixels-buffer-not-converted-to-srgb-on-png-save|Blender nie przelicza `image.pixels[]` na sRGB przy zapisie PNG]] - wspolne: texture, blender
- [[procedural-textures-cycles-commercial|Procedural Textures in Blender Cycles (Commercial Release Rationale)]] - wspolne: cycles, blender
- [[20260606-0930-baked-atlas-texture-foreign-uvs|Don't apply a baked-atlas texture to a mesh whose UVs were authored for a different atlas]] - wspolne: atlas, blender
<!-- /POWIAZANE:auto -->
