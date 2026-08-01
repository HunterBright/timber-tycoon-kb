---
title: 'Prześwity w modelach low-poly: najpierw sprawdź _Cull materiału, nie geometrię'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-19'
project: Kerf - Sawmill Tycoon
tags:
- unity
- urp
- materials
- backface-culling
- low-poly
- see-through
- vehicles
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Prześwity w modelach low-poly: najpierw sprawdź _Cull materiału, nie geometrię

## Objaw
Model (auto NPC) "czasami prześwituje" - przez karoserię/koła widać na wylot. Podejrzenie
pada na brakujące face'y albo odwrócone normalne; malowanie face'ów w Blenderze nic nie daje.

## Root cause
Modele low-poly budowane jako "papierowa skorupa" (jedna warstwa ścianek na zewnątrz,
otwarty spód, burty bez wewnętrznej strony) są NORMALNE i tanie - ale wyglądają dobrze
tylko z materiałami dwustronnymi. W URP Lit: `_Cull: 0` = obie strony, `_Cull: 2` =
domyślne, tylko przód. Gdy kamera zajrzy do wnętrza skorupy (szpara przy zderzaku,
wnętrze paki, nadkole pod ostrym kątem), jednostronny materiał nie rysuje nic = prześwit.

W Timber Tycoon: auto gracza miało wszystkie materiały `_Cull: 0` (wyglądało OK), auto
NPC `_Cull: 2` (prześwitywało) - IDENTYCZNA klasa budowy geometrii, różnica tylko w
materiałach. Geometria obu zdrowa.

## Lekcja
1. Przy zgłoszeniu "prześwit/dziura w modelu" NAJPIERW porównaj `_Cull` materiałów
   (grep `_Cull` w .mat - to zwykły YAML) między modelem zepsutym a podobnym działającym.
   Dopiero potem audytuj geometrię.
2. Dla propów/pojazdów NPC dwustronny materiał to pełnoprawna naprawa - domodelowanie
   wewnętrznych ścianek jest 10x droższe i nie daje nic ponad to samo.
3. `normalImportMode: Import` (Unity używa normalnych z FBX) przepuszcza odwrócone
   normalne 1:1 - ale odwrócona NORMALNA psuje tylko cieniowanie; o cullingu decyduje
   kolejność wierzchołków (winding). W Blenderze oba wynikają z orientacji ścianki.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260721-1830-linerenderer-flat-on-surface-invisible|LineRenderer lezacy plasko na powierzchni znika, bo material jest jednostronny]] - wspolne: backface-culling, materials, urp
- [[20260702-2140-shader-property-stale-serialized-material-values|Dodanie property do shadera może aktywować STARE, ukryte wartości w materiałach]] - wspolne: materials, urp
- [[20260710-1300-vertex-colors-vs-basecolor-linear|Vertex colors vs _BaseColor w Linear color space - ten sam hex renderuje się INACZEJ]] - wspolne: materials, urp
- [[20260713-2145-urp-transparent-material-silent-failure|URP: źle skonfigurowany materiał przezroczysty to CICHA porażka, której wykrywacz magenty nie widzi]] - wspolne: materials, urp
- [[procedural-textures-need-bake|Procedural Textures Must Be Baked Before FBX Export]] - wspolne: materials, urp
- [[stale-reflection-probe-night-whitening|Stale Skybox Reflections Whiten PBR Materials at Night (Day/Night Cycle)]] - wspolne: materials, urp
<!-- /POWIAZANE:auto -->
