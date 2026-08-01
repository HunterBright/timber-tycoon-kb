---
title: MeshCollider na siatce bez Read/Write = TWARDY natywny crash builda przy odsloniecie (nie w Edytorze)
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-15'
project: Kerf - Sawmill Tycoon
tags:
- unity
- meshcollider
- read-write-enabled
- build-vs-editor
- native-crash
- physx
- setactive
applies_to:
- unity-6
- physx
- standalone-build
source: ''
severity: high
time_lost: ~1h diagnoza + zlapanie 2 kolejnych instancji
promoted: '2026-07-30'
---

# MeshCollider na siatce bez Read/Write = TWARDY natywny crash builda przy odsloniecie (nie w Edytorze)

## Problem
Gra wywala sie DO PULPITU (twardy crash, nie wyjatek C#) w momencie, gdy obiekt z MeshColliderem
zostaje odsloniety w runtime (`SetActive(true)` po zakupie/spawnie). Raport crashu ma SAME ramki
natywne UnityPlayer/UnityMain, ZERO warstwy C# - gdyby to byl zwykly blad kodu, silnik by go
zalogowal i gral dalej. W EDYTORZE ten sam obiekt dziala bezblednie. Objaw pojawia sie dopiero
w zbudowanej grze i tylko przy pierwszym "ugotowaniu" collidera.

## Root cause
Trzy warunki naraz:
1. Model FBX ma **Read/Write Enabled = OFF** (`isReadable: 0` w .meta) -> w buildzie dane siatki
   po stronie CPU sa STRIPPED (zostaja tylko na GPU).
2. Na siatce jest **MeshCollider** (zwlaszcza `convex = false`).
3. **Player Settings -> Prebake Collision Meshes = OFF** (`bakeCollisionMeshes: 0`) -> kolizja NIE
   jest przeliczona przy budowaniu; cook nastepuje dopiero w runtime.

Gdy collider "kuka sie" pierwszy raz w runtime (moment `SetActive`/enable), PhysX siega po
skasowane dane siatki -> natywny access fault. W Edytorze dane siatki sa ZAWSZE dostepne, wiec cook
przechodzi - stad "dziala w edytorze, pada w buildzie". Szczegolnie zdradliwe, gdy obiekty startuja
UKRYTE i pojawiaja sie po zdarzeniu gry (zakup maszyny, spawn) - cook nie nastepuje przy ladowaniu
sceny, tylko przy odsloniecie, wiec build startuje OK i pada dopiero w trakcie gry.

## Solution
Najwezszy sprawdzony fix: **wlacz Read/Write Enabled na modelu FBX** (`isReadable: 0 -> 1` w .meta,
reimport). Dane siatki zostaja w buildzie, runtime-cook przechodzi. Alternatywy: (a) zamien
MeshCollider na prymityw (BoxCollider) - cook siatki znika calkowicie; (b) wlacz globalnie Prebake
Collision Meshes - przelicza kolizje przy budowaniu dla WSZYSTKICH colliderow (szeroki zasieg,
wiekszy/wolniejszy build).

**Wykrywanie w smoke-tescie:** dodaj check, ktory dla kazdego runtime-odslanianego MeshCollidera
asertuje `sharedMesh.isReadable == true`. WAZNE: tu check FLAGOWY, nie behawioralny - behawioralny
(faktyczny cook) wywalilby SAMA sonde tym samym crashem. Falsyfikowalny: wylaczenie Read/Write ->
isReadable=false -> FAIL. W Timber Tycoon ten check od razu zlapal DWIE kolejne maszyny z tym samym
uspionym crashem (jedna nie crashowala przy odsloniecie bo jej siatkowe kolizje byly tylko na
elementach aktywowanych dopiero w minigrze).

## What didn't work
- Poleganie na tym, ze "dziala w Edytorze" - Edytor NIGDY nie odtworzy tego crasha (dane siatki
  zawsze dostepne). Dowodem moze byc TYLKO build.
- Behawioralny check w sondzie (realne odsloniecie + cook) - crashuje sonde bez raportu. Dla
  TWARDEGO crasha uzyj flagi, nie zachowania (wyjatek od zasady "sonda behawioralna > flagowa").
- Zalozenie "jedna instancja" - crash z tej rodziny zwykle wystepuje na WIELU obiektach; po
  znalezieniu jednego przeskanuj wszystkie podobne (u nas: wszystkie kupowane maszyny).

## Transferability
Dotyczy KAZDEGO projektu Unity ze standalone buildem, ktory: (a) importuje modele z Read/Write OFF
(domyslne, oszczedza pamiec), (b) uzywa MeshColliderow na tych modelach, (c) ma Prebake Collision
Meshes OFF, (d) odslania/spawnuje takie obiekty w runtime. Klasyczna pulapka "Editor lies": bug
jest niewidoczny az do builda i tylko przy pierwszym cooku collidera.

## Related
- Rodzina "Editor lies" / mesh isReadable w buildzie (naprawa 27 modeli, 2026-07-13)
- Wzorzec: smoke-test/sonda jako build-smoke pilnujacy przeszlych bledow

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260713-1830-runtime-meshcollider-needs-readwrite-and-editor-cannot-prove-it|Runtime MeshCollider wymaga Read/Write Enabled - a Edytor NIE JEST w stanie tego udowodnić]] - wspolne: read-write-enabled, physx, meshcollider
<!-- /POWIAZANE:auto -->
