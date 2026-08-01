---
title: 'Luz między warstwami ubrań: mierz WYSOKOŚĆ NAD CIAŁEM, nie odległość do najbliższej ścianki'
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-07-30'
project: Kerf - Sawmill Tycoon
tags:
- blender
- clothing
- layers
- collision
- bvh
- measurement
- heightfield
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Luz między warstwami ubrań: mierz WYSOKOŚĆ NAD CIAŁEM, nie odległość do najbliższej ścianki

## Problem

Relacja "warstwa A ≥ 2 mm nad warstwą B" mierzona jako odległość punktu A
do najbliższej ścianki B (BVH find_nearest + rzut na normalną ścianki)
sypie się na gęstych, zdeformowanych siatkach:

1. najbliższy punkt bywa NAROŻNIKIEM/krawędzią ścianki - rzut na normalną
   daje "-27 mm w środku", choć punkt leży OBOK powierzchni;
2. rękaw koszuli wisi NA ZEWNĄTRZ kamizelki (przy torsie) - kieszeń
   rękaw-tors jest niespełnialna i korekty wpadają w wieczny cykl;
3. wąskie ścianki rantu (solidify), powłoka wewnętrzna i ścianki
   przenicowane deltami kluczy dają odczyty o odwróconym kierunku -
   automatyczne "poprawki" ciągną siatkę w złą stronę (baza -9 mm).

Filtry (klasyfikacja powłok po normalnej ciała, wykluczanie rantów,
wykluczanie ścianek odwróconych) leczą część objawów, ale każdy filtr
robi DZIURY w powierzchni pomiarowej i rodzi nowe artefakty narożnikowe.

## Rozwiązanie

Przeformułowanie: warstwy to FUNKCJE WYSOKOŚCI NAD SKÓRĄ.
Dla punktu p warstwy zewnętrznej:
1. najbliższy punkt b CIAŁA (w danym stanie suwaków) + normalna ciała n;
2. h_p = (p - b) . n  (wysokość punktu nad skórą);
3. h_s = najdalsze trafienie promienia z b wzdłuż n w PEŁNĄ siatkę
   źródła (≤ 60 mm) = zewnętrzna powierzchnia źródła nad tym punktem
   skóry; brak trafienia = źródło nie kryje tego punktu → brak warunku;
4. warunek: h_p ≥ h_s + margines; poprawka wzdłuż n (normalnej CIAŁA),
   czyli zawsze NA ZEWNĄTRZ → monotoniczna zbieżność.

Rękawy przestają przeszkadzać AUTOMATYCZNIE: wiszą nad punktami skóry
RĘKI, a kamizelka pyta o punkty TORSU. Zero filtrów powłok w pomiarze.
Relacje ubranie-vs-ciało zostają na starym pomiarze (ciało to pojedyncza
zdrowa powłoka).

## Bonus z tej samej sesji

Poprawki anty-przebiciowe per stan suwaka: TYLKO przesunięcia jednolite
(baza + wszystkie klucze naraz) ze strefą martwą (ruszaj wyłącznie luz
< 0.5 mm, odkładaj na pełny margines) - gonienie marginesu przez tysiące
wierzchołków deformowało krawędzie w nawisy, a poprawki w danych
pojedynczego klucza oscylują na suwakach dwustronnych.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260626-1807-bridge-abutment-seam-fix|Two-stage seam fix: terrain edge-loop + road BVH re-drape at a bridge abutment]] - wspolne: bvh, heightfield, blender
- [[20260730-2140-shape-key-layer-corrections-oscillate|Poprawki anty-przebiciowe w danych POJEDYNCZEGO klucza kształtu oscylują]] - wspolne: clothing, layers, blender
- [[20260730-1950-proxy-clothing-tangential-smoothing|Ubrania proxy na low-poly ciele: wygładzanie styczne zamiast laplasjanu]] - wspolne: clothing, blender
- [[20260626-1808-probe-heightfield-before-terrain-edit|Probe the real heightfield before scripting terrain edits - assumed profiles drift]] - wspolne: heightfield, blender
- [[20260728-0915-fbx-skala-100-w-dzieciach-psuje-pomiary|FBX z Blendera: przelicznik jednostek siedzi w SKALI DZIECI, nie w korzeniu]] - wspolne: measurement, blender
<!-- /POWIAZANE:auto -->
