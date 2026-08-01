---
title: 'FBX z Blendera: przelicznik jednostek siedzi w SKALI DZIECI, nie w korzeniu'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-28'
project: Kerf - Sawmill Tycoon
tags:
- unity
- blender
- fbx
- import
- scale
- mesh
- measurement
- editor-script
applies_to: []
source: ''
promoted: '2026-07-30'
---

# FBX z Blendera: przelicznik jednostek siedzi w SKALI DZIECI, nie w korzeniu

## Objaw

Skrypt edytorowy mierzy model wprost z siatki (`mesh.bounds`, `mesh.vertices`)
i dostaje wyniki sto razy za male: drzewo o wysokosci 5,48 m wychodzi 0,04 m.
W scenie model wyglada poprawnie, wiec blad wyglada na blad skryptu, nie modelu.

## Przyczyna

Importer FBX nie zawsze wpisuje przelicznik jednostek w korzen modelu. Przy
eksporcie z Blendera potrafi wyladowac w `localScale` DZIECI:

```
Kerf_Spruce            localScale = (1, 1, 1)      <- korzen wyglada niewinnie
  Wood                 localScale = (100, 100, 100)
  Foliage              localScale = (100, 100, 100)
```

`mesh.bounds` i `mesh.vertices` sa w przestrzeni SIATKI, czyli przed ta skala.
Sprawdzenie skali samego korzenia niczego nie wykrywa.

## Rozwiazanie

Kazdy wierzcholek przepuscic przez macierz z przestrzeni siatki do przestrzeni
korzenia modelu:

```csharp
Matrix4x4 doModelu = model.transform.worldToLocalMatrix * mf.transform.localToWorldMatrix;
Vector3 v = doModelu.MultiplyPoint3x4(vLokalny);
```

Dziala niezaleznie od tego, gdzie siedzi przelicznik i ile poziomow ma hierarchia.
`Renderer.bounds` tez daje dobre liczby, ale tylko dla obiektu faktycznie
w scenie - przy pracy na zawartosci prefabu (`PrefabUtility.LoadPrefabContents`)
pewniejsza jest macierz.

## Sygnal ostrzegawczy

Jesli pomiar z siatki wychodzi rownym rzedem wielkosci obok prawdy (100x, 0,01x),
to prawie na pewno jest to skala jednostek, a nie blad liczenia. Sprawdz
`localScale` DZIECI, nie korzenia.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260629-1145-blender-empties-bake-space-transform-double-axis|FBX with parent EMPTIES imports tipped 90° when exported with bake_space_transform=True]] - wspolne: import, fbx, blender
- [[flatten-must-be-baked-into-geometry-when-code-forces-uniform-scale|Flatten Must Be Baked Into Geometry When Code Forces Uniform Scale]] - wspolne: scale, fbx, blender
- [[20260531-1705-normalize-assetpack-scale-via-modelimporter|Normalize Inconsistent Asset-Pack Scale at the Source (ModelImporter.globalScale)]] - wspolne: scale, fbx
- [[20260612-1340-unity-batch-fbx-import-meta-mirroring|Batch FBX import with pre-authored .meta files + prefab build in temp additive scene]] - wspolne: import, fbx
- [[20260531-0934-tripo-polygon-soup-inverted-winding-fix|Tripo / AI-generated meshes import as "polygon soup" - see-through holes under single-sided rendering are a winding problem caused by UNWELDED verts, not interior faces]] - wspolne: fbx, blender
- [[20260626-1203-fbx-pivot-direction-vs-procedural-placement|Pivot/geometry direction of an FBX must match what a procedural placement tool assumes]] - wspolne: fbx, blender
<!-- /POWIAZANE:auto -->
