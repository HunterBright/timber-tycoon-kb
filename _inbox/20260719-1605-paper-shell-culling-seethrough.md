---
title: 'Prześwity w modelach low-poly: najpierw sprawdź _Cull materiału, nie geometrię'
type: lesson
status: draft
confidence: low
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
suggested-category: engine/lessons
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
