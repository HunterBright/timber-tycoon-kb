---
title: 'Blender: image.scale() na file-backed image nie trzyma się podczas bake — użyj images.new()'
type: lesson
status: draft
confidence: low
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
suggested-category: engine/lessons
---

# Blender: image.scale() na file-backed image nie trzyma się podczas bake — użyj images.new()

## Objaw
Chcąc wypalić istniejący atlas w wyższej rozdzielczości (1024 → 2048), otwarto istniejący
plik-obraz (source = FILE) i wywołano `image.scale(2048, 2048)` przed `bpy.ops.object.bake()`.
Bake wyprodukował mimo to PNG **1024×1024**.

## Root cause
Obraz o źródle FILE podczas bake'u **cicho wraca do rozdzielczości z dysku** — `image.scale()`
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
(To dokładnie robi oryginalny build-script gry — używa `images.new`, nie re-używa pliku.)

## Zabezpieczenie
Zawsze `assert atlas_img.size == (W, H)` bezpośrednio PO `object.bake`, ZANIM zapiszesz PNG —
łapie ciche revert-y rozdzielczości.

## Kontekst
Re-tekstura kompostownika (FertilizerMaker) w Timber Tycoon: podbicie atlasu 1024→2048.
Pierwszy przebieg dał 1024 mimo `scale()`; drugi przez `images.new(2048)` dał poprawne 2048.
