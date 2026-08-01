---
title: 'Proceduralne okrągłe sęki w Blenderze: Voronoi F1, nie DISTANCE_TO_EDGE (+ kompensacja proporcji)'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-06-15'
project: Kerf - Sawmill Tycoon
tags:
- blender
- procedural-texture
- voronoi
- wood
- baking
- ui-sprite
applies_to:
- blender
- eevee
- procedural-shading
source: ''
severity: low
time_lost: ~20 min (kilka iteracji renderu)
promoted: '2026-07-30'
---

# Proceduralne okrągłe sęki w Blenderze: Voronoi F1, nie DISTANCE_TO_EDGE (+ kompensacja proporcji)

## Problem
Generując drewnianą teksturę (baked PNG do UI) chciałem „sęki" jako okrągłe ciemne plamy.
Użycie węzła Voronoi w trybie `DISTANCE_TO_EDGE` dało zamiast sęków **długie ukośne ciemne linie /
spękania** krzyżujące deskę. Przy mocniejszym ustawieniu wyglądało jak rozbita, popękana powierzchnia.
Dodatkowo na wydłużonym sprite (proporcja ~1:3.2) plamy były owalne/rozciągnięte, nie okrągłe.

## Root cause
- `DISTANCE_TO_EDGE` zwraca odległość do **granicy komórki** Voronoi → próg na małych wartościach
  daje siatkę LINII wzdłuż granic komórek (czyta się jak pęknięcia), a nie plamy w środkach komórek.
- Współrzędne Generated/UV są normalizowane 0..1 na każdej osi niezależnie. Na niekwadratowej
  (wydłużonej) geometrii jednakowy `Scale` Voronoi tworzy komórki **rozciągnięte w pikselach** →
  owalne/ukośne kształty.

## Solution
1. Ustaw Voronoi `feature = 'F1'` i podłącz wyjście **`Distance`** (odległość do najbliższego
   PUNKTU cechy). Próg na NISKICH wartościach (`MapRange` From 0..promień → To 1..0) daje okrągłe
   ciemne plamy DOKŁADNIE w punktach cechy = sęki.
2. **Kompensuj proporcje**: w `Mapping` przed Voronoi przeskaluj oś długą o współczynnik proporcji
   sprite'a (np. Y-scale = 3.2 dla sprite 1:3.2), żeby sęki były okrągłe w pikselach, nie owalne.
3. Strojenie: liczba sęków = `Scale` Voronoi; rozmiar = `From Max` w MapRange; siła/ciemność =
   mnożnik faktora miksu z kolorem sęka.

## What didn't work
- `DISTANCE_TO_EDGE` ze zwiększoną siłą → linie graniczne stawały się jeszcze wyraźniejszymi
  „pęknięciami", nie sękami.
- Pozostawienie mappingu bez kompensacji proporcji → sęki owalne/rozjechane na wydłużonej desce.

## Transferability
Dotyczy KAŻDEJ proceduralnej tekstury w Blenderze, gdzie potrzeba okrągłych plam (sęki, piegi,
nity, wżery rdzy, kropki) - zwłaszcza na wydłużonej/niekwadratowej powierzchni lub sprite UI:
F1+Distance zamiast DISTANCE_TO_EDGE, plus pre-skalowanie współrzędnych o proporcję.

## Related
- (Timber Tycoon) generator deski: _TempEditor/plank_timingbar_build.py
- [[project_blender51_mcp_bridge]] - headless render bez żywego mostka MCP

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260618-0724-blender-ortho-ui-sprite-bake-framing|Baking flat UI sprites in Blender: ortho frame width = ortho_scale × 2]] - wspolne: ui-sprite, blender
- [[procedural-textures-need-bake|Procedural Textures Must Be Baked Before FBX Export]] - wspolne: baking, blender
- [[cycles-bake-for-solid-colors|ANTI-PATTERN: Cycles Bake for Solid Color Regions]] - wspolne: baking, blender
<!-- /POWIAZANE:auto -->
