---
title: Za drobny voxel NavMesh tworzy mikro-dziure, ktora ROZSPAJA trase i wymusza wielki objazd
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-07'
project: Kerf - Sawmill Tycoon
tags:
- unity
- navmesh
- ai-navigation
- navmeshsurface
- voxelsize
- pathfinding
- regression
- git-lfs
- diagnosis
applies_to: []
source: ''
severity: medium
time_lost: duga (wielogodzinna) diagnoza z wieloma falszywymi tropami
promoted: '2026-07-30'
---

# Za drobny voxel NavMesh tworzy mikro-dziure, ktora ROZSPAJA trase i wymusza wielki objazd

## Problem
Po fixie "niewidzialnych gorek" (NPC podskakiwal na terenie) obnizylismy `NavMeshSurface.voxelSize`
z ~0.166 do 0.08. Bumpy zniknely, ale trasa 2. klienta zaczela isc WIELKIM lukiem na zewnatrz budynku.
Objaw wygladal jak "zla trasa" i prowokowal falszywe tropy (sciana, strona wysiadania z auta, kolizje
z zaparkowanym autem, kolejnosc parkingu) - wszystkie ODRZUCONE po pomiarach.

## Root cause
Bardzo drobny voxel (0.08) przy ciasnej geometrii (rog budynku + collidery) zostawil **mikro-dziure /
cienki niechodliwy szew w siatce dokladnie na jednej trasie** (tu: dojscie z jednego miejsca
parkingowego do rogu-wejscia). Ta dziura ROZSPOILA krotka trase - `NavMesh.CalculatePath` nie mogl jej
uzyc i zwracal DLUZSZY objazd na zewnatrz. Kluczowe: dotyczylo to TYLKO jednego miejsca (tego, ktorego
trasa trafiala w szew) - pozostale 11 mialo trase bez zmian. Grubszy voxel "zasypuje" takie szwy
(mostkuje male przerwy), wiec ich nie mial.

## Solution
**Sweep voxelSize i znajdz optimum bump-hug vs route-connectivity.** Zmierzone per wartosc: (a) czy
trasa dalej robi objazd (maxZ/dlugosc), (b) poziom bumpow (ile probek siatki wisi >10cm nad terenem +
max hover). Wynik: 0.12 = trasa OK (dziura zasypana) I bumpy jak przy 0.08. Fix = jedna liczba
`voxelSize = 0.12`.
GOTCHA: liczba bumpow per voxel jest **NIEMONOTONICZNA** - zalezy jak siatka voxeli "trafia" w duze
trojkaty niskopoly terenu (0.10/0.11/0.14 gorsze niz 0.08 i 0.12). Testuj kilka wartosci, nie zakladaj
"drobniej = zawsze mniej bumpow" ani "grubiej = zawsze wiecej".

## Kluczowa technika diagnostyczna: odtworz scene sprzed zmiany z git (nawet binarna/LFS)
"Dzialalo 3 dni temu" - zamiast zgadywac, ODTWORZ tamten stan i zmierz. Scena binarna w LFS:
`git show <commit>:Assets/Scene.unity | git lfs smudge > Assets/_TmpBefore.unity`
-> `AssetDatabase.Refresh(ForceSynchronousImport)` -> `open_scene` -> zmierz `NavMesh.CalculatePath` na
STAREJ siatce -> reopen biezaca scene -> skasuj temp. Nieniszczace (biezaca scena na dysku nietknieta).
To ROZSTRZYGA "co sie zmienilo": u nas pokazalo, ze sciana byla wycieta ZAWSZE (nie ona), a rozdzielczosc
siatki zmienila ksztalt trasy. Bez tego bylbym utknal na falszywych tropach.

## What didn't work (falszywe tropy - zapisz, by nie powtarzac)
- "Sciana blokuje" - sciana byla w siatce tez PRZED zmiana (pomiar starej sceny to obalil). Wczesniejsze
  "demo" z wykluczeniem CALEGO budynku (floor+walls) dawalo prosta linie, ale to artefakt (usuwa tez
  podloge) - mylace, nie rob tak.
- "Strona wysiadania z auta / okrazanie wlasnego auta" - realne, ale drugorzedne; nie tworzylo luku
  (trasa nie dotykala auta).
- "Kolejnosc parkingu / przesun parking / otworz sciane" - wszystkie odrzucone, bo leczyly objaw, nie
  przyczyne (voxel).

## Transferability
Czysto silnikowe (Unity AI Navigation). Kazdy projekt z wypiekana siatka po niskopoly terenie: gdy
drobny voxel poprawia przyleganie do gruntu, moze jednoczesnie rozspajac trasy przy ciasnej geometrii.
Diagnozuj przez sweep voxela + odtworzenie starej sceny z git. Ogolniej: gdy "cos co dzialalo" psuje sie
po zmianie parametru jakosci (voxel/tile/tolerancja), podejrzewaj efekt uboczny na topologii/lacznosci,
nie tylko na "jakosci".

## Related
- [[20260706-1520-navmesh-raised-collider-invisible-bump]] - ten sam system; voxel 0.08 stad pochodzil.
- [[20260707-0715-navmesh-decorative-collider-carves-service-line]]
- [[20260707-0745-worked-before-meant-relying-on-navmesh-phasing]] - inny "dzialalo wczesniej" (tam bylo
  przenikanie; tu odwrotnie - stara siatka byla OK, zmiana ja popsula).
