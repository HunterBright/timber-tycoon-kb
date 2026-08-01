---
title: 'Wzorzec audytu prześwitów w siatkach: render 3-przebiegowy > heurystyki geometryczne'
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-07-19'
project: Kerf - Sawmill Tycoon
tags:
- unity
- blender
- audit
- mesh
- normals
- rendering
- pipeline
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Wzorzec audytu prześwitów w siatkach: render 3-przebiegowy > heurystyki geometryczne

## Problem
Jak obiektywnie wykryć prześwity (odwrócone ścianki / dziury) w modelach, nie polegając
na oku ani na zawodnych heurystykach?

## Wzorzec (zweryfikowany na 5 modelach aut Timber Tycoon)
Render każdego modelu na tle magenta (1,0,1), każde ujęcie w 3 przebiegach:
- A: materiały jak w grze,
- B: kopie materiałów z `SetFloat("_Cull", 0)` (dwustronne),
- C: czarny Unlit Cull Off = maska sylwetki.
Klasyfikacja piksela: `covered = C nie-magenta`; `prześwit = covered && A magenta &&
B nie-magenta`; `dziura na wylot = covered && B magenta`. Liczby do CSV, ujęcia ponad
progiem dostają PNG dowodowe. Przebieg B to jednocześnie DARMOWY podgląd naprawy
"materiał dwustronny".

Macierz ujęć: 8 yaw x 2 elewacje + spód + zbliżenia newralgicznych części (koła: osiowe
+ styczne przy gruncie). UWAGA na artefakt: przy ciasnych zbliżeniach kamera może wejść
DO WNĘTRZA siatki - wtedy cały kadr to "prześwit" i wynik trzeba odrzucić (sprawdź
przebieg B: jeśli wypełnia cały kadr od środka, kamera jest w bryle).

## Anty-wzorzec: heurystyki odwróconych normalnych
Głosowanie parzystość-raycastu / recalc-diff / centroid-shell daje MASOWE fałszywe
pozytywy na otwartych siatkach wieloskorupowych (cienkie progi, scalone bryły):
recalc ma losowy znak globalny na otwartych skorupach, centroid myli się na elementach
doklejonych do cudzego pivota. Ground truth = render z cullingiem + RENDER KONTROLNY
z celowo odwróconymi wszystkimi normalnymi (dowód, że wykrywacz umie zawieść -
por. lekcja "sonda musi umieć zawieść").

## Bonus: wykrywanie przestarzałego eksportu
Sygnatura per obiekt (nazwa, verts, tris, tris-per-slot, bbox 1e-3, nazwy materiałów)
liczona z depsgraph .blend vs import FBX; identyczny timestamp pliku .blend i .fbx to
mocna poszlaka wspólnego eksportu, ale rozstrzyga sygnatura.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260728-1105-samotest-sprawdzajacy-wlasne-normalne-jest-slepy|Samotest sprawdzajacy WLASNE normalne jest slepy na odwrocona scianke]] - wspolne: normals, mesh
- [[river-mesh-semi-ellipse-cross-section|River Mesh Semi-Elliptical Cross-Section]] - wspolne: mesh, blender
- [[20260612-1200-eevee-shadow-acne-wavy-lines|Wavy dark lines in EEVEE preview renders = shadow acne, not geometry]] - wspolne: rendering, blender
- [[20260531-0934-tripo-polygon-soup-inverted-winding-fix|Tripo / AI-generated meshes import as "polygon soup" - see-through holes under single-sided rendering are a winding problem caused by UNWELDED verts, not interior faces]] - wspolne: normals, blender
- [[20260801-0826-trellis-2-generator-3d-bez-blokady-ue|TRELLIS.2 jako generator 3D bez blokady licencyjnej w UE]] - wspolne: pipeline, blender
- [[character-pipeline-tripo-mixamo-unity|Character pipeline: Tripo mesh → Mixamo rig → Unity (clean, working recipe)]] - wspolne: pipeline, blender
<!-- /POWIAZANE:auto -->
