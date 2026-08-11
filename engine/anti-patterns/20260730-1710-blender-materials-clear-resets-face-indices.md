---
title: Mesh.materials.clear() zeruje material_index na ściankach
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-07-30'
project: Kerf - Sawmill Tycoon
tags:
- blender
- python
- materials
- material_index
- mesh
- gotcha
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Mesh.materials.clear() zeruje material_index na ściankach

## Anti-pattern

W skrypcie budującym siatkę przypisujemy ściankom materiał ze slotu 1
(`polygon.material_index = 1`), a później w innym miejscu skryptu
"porządkujemy" listę materiałów przez `mesh.materials.clear()` +
`append(...)`.

## Dlaczego nie działa

Blender przy usuwaniu slotów materiałów (w tym `clear()`) wykonuje
remap/clamp indeksów materiałów na wszystkich ściankach. Po `clear()`
wszystkie ścianki wracają na indeks 0 - przypisania giną BEZ ŻADNEGO
błędu ani ostrzeżenia. Kolejne `append` odtwarza sloty, ale ścianki już
wskazują slot 0.

Objaw w praktyce (LowPolyHumanGenerator 2026-07-30): sloty w pliku
wyglądały poprawnie (`['Mat_Skora', 'Mat_Usta']`), ale liczba ścianek
z indeksem 1 wynosiła 0 i usta renderowały się kolorem skóry.

## Poprawny wzorzec

Podmieniać materiały W MIEJSCU, bez usuwania slotów:

```python
while len(mesh.materials) < 2:
    mesh.materials.append(None)
mesh.materials[0] = mat_skora
mesh.materials[1] = mat_usta
```

Albo: najpierw ustalić ostateczną listę slotów, dopiero potem przypisywać
`material_index` (kolejność ma znaczenie - przypisania muszą być OSTATNIE,
jeśli gdziekolwiek w potoku występuje usuwanie slotów).

## Diagnostyka

Szybki test w pliku wynikowym: policzyć ścianki per indeks -
`len([p for p in mesh.polygons if p.material_index == 1])`. Sloty mogą
być poprawne, a przypisania puste.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260704-1732-blender-linked-basecolor-recolor|Recoloring a Blender material whose Base Color is LINKED does nothing via default_value]] - wspolne: python, materials, blender
- [[20260719-2015-ai-gen-model-geometry-debt|Modele generowane przez AI (Tripo/Hunyuan): dług geometryczny - czasem taniej zbudować od zera]] - wspolne: mesh, blender
- [[20260606-0930-baked-atlas-texture-foreign-uvs|Don't apply a baked-atlas texture to a mesh whose UVs were authored for a different atlas]] - wspolne: materials, blender
- [[20260704-1322-blender-file-image-scale-bake-revert|Blender: image.scale() na file-backed image nie trzyma się podczas bake - użyj images.new()]] - wspolne: gotcha, blender
- [[20260728-0915-fbx-skala-100-w-dzieciach-psuje-pomiary|FBX z Blendera: przelicznik jednostek siedzi w SKALI DZIECI, nie w korzeniu]] - wspolne: mesh, blender
- [[20260719-1605-mesh-seethrough-audit-pattern|Wzorzec audytu prześwitów w siatkach: render 3-przebiegowy > heurystyki geometryczne]] - wspolne: mesh, blender
<!-- /POWIAZANE:auto -->
