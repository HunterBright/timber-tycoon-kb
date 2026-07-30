---
title: 'NPC chodza po "niewidzialnych gorkach": bake NavMesh z propsow + za gruby voxel nad niskopoly terenem'
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-06'
project: Kerf - Sawmill Tycoon
tags:
- unity
- navmesh
- ai-navigation
- navmeshsurface
- layermask
- voxelsize
- low-poly-terrain
- pathfinding
- colliders
applies_to: []
source: ''
suggested-category: engine/lessons
---

# NPC chodza po "niewidzialnych gorkach": bake NavMesh z propsow + za gruby voxel nad niskopoly terenem

## Objaw
Agenci NavMesh (piesi NPC) w drodze do celu "wchodza pod niewidzialna gorke" i schodza z drugiej
strony, mimo ze widoczny grunt jest plaski.

## Dwie przyczyny (czesto naraz)

### 1. Bake bierze WSZYSTKIE collidery, nie samo podloze
`NavMeshSurface` z `collectObjects = CollectObjects.All` + `useGeometry = PhysicsColliders`
wpieka KAZDY collider nie-trigger: drzewa, kamienie, platformy, budynki. Plaski szeroki prop
sterczacy nad gruntem (np. betonowa plyta) staje sie chodliwa "wyspa" z rampowym brzegiem ->
agent na nia wchodzi. Wysokie przeszkody (skaly, budynki) sa carve'owane, ale ich pochyle
DOLNE boki tez potrafia wypiec sie jako maly chodliwy garb.

### 2. Za gruby voxel nad niskopoly terenem
Nawet po wykluczeniu propsow siatka potrafi lezec ~0.10-0.20 m NAD terenem, nierowno. To NIE
sa propsy - to granulacja voxelizacji (domyslny voxel humanoida ~0.166 m) nad duzymi plaskimi
trojkatami low-poly terenu. Agent z `baseOffset = 0` jedzie po tej siatce, wiec "unosi sie" i
faluje o kilkanascie cm = wrazenie chodzenia po garbach.

## Diagnoza (bez patrzenia na gizmo)
Nie zawsze da sie wyrenderowac gizmo NavMesh. Zamiast tego zmierz siatke liczbowo:
- `NavMesh.CalculatePath(spawn, cel)` -> sprawdz `status == PathComplete` (czy cel osiagalny).
- GESTO probkuj wzdluz sciezki (co ~0.5 m, nie tylko rogi!): w kazdym punkcie porownaj
  `NavMesh.SamplePosition().y` (po czym agent chodzi) z raycastem w dol na warstwe terenu
  (gladkie podloze). Roznica = ile siatka wystaje nad grunt = garb. Licz ile probek > 0.10 m
  i maksimum. (Analiza tylko rogow MYLI - garb miedzy rogami na prostym odcinku nie tworzy rogu.)

## PULAPKA: "bake tylko z gruntu" wywala blokowanie scian
Kuszace jest ustawic `surface.layerMask = LayerMask.GetMask("Terrain","Road")`, zeby wyrzucic
propsy. NIE ROB TEGO na slepo: collidery scian/budynkow/drzew sa w bake po to, by WYCIAC "dziure"
w siatce i ZABLOKOWAC agenta. Jak je wykluczysz, siatka powstaje plaska POD budynkiem i **agent
przechodzi przez sciany** (u nas zmierzone: sciezka na wylot przez sciane, ratio dlugosci 1.00).
Voxel/garb to problem WALKABLE, a blokady to problem CARVE - to dwie rozne rzeczy, nie mieszac.

## Fix (wlasciwy)
1. **Zostaw wszystkie collidery w bake** (`CollectObjects.All`, `layerMask = ~0`) - sciany/drzewa/
   kamienie maja blokowac (carve). NIE wykluczaj ich layerMask'iem.
2. **Drobniejszy voxel na garb terenu**: `surface.overrideVoxelSize = true; surface.voxelSize = 0.08f`
   (~polowa domyslnego). Siatka przylega do niskopoly terenu. NIE oslabia carve scian (sprawdzone:
   sciany blokuja tak samo jak przy domyslnym voxelu). Wolniejszy bake, ale mala mapa = bez znaczenia.
3. **Plaskie propsy, ktore staly sie chodliwa wyspa (np. platforma/rampa)** oznacz punktowo
   `NavMeshModifier` (`overrideArea = Not Walkable`, `applyToChildren = true`) - wtedy sie WYCINAJA
   zamiast byc chodliwym garbem, a inne obiekty dalej normalnie blokuja.
4. Po zmianie ZAWSZE re-bake i zapis sceny (sam modyfikator/ustawienie nic nie da bez przeliczenia).

Efekt zmierzony u nas: garb terenu max 0.20 m -> 0.09 m (probki >0.10 m: 416 -> 0), sciezka do celu
PathComplete, a solidne sciany dalej blokuja (ratio detour 2-5x), tylko otwarty front sklepu jest
przejsciowy (zgodnie z zamyslem - uwaga: sprawdz czy Twoj cel/lada nie lezy w obrysie budynku od
otwartej strony, bo wtedy "przejscie" tamtedy jest OK).

## Powiazana pulapka: NISKIE propsy sa "wchodzalne" (agent chodzi PO nich / przez nie)
Obiekt nizszy niz `agentClimb` (domyslnie ~0.4 m dla humanoida) NIE jest wycinany jako przeszkoda
- NavMesh generuje sie PO jego wierzchu (agent "wchodzi na stopien"). Efekt: NPC przechodzi przez
niska lade/blat/skrzynie jak przez powietrze. To NIE zalezy od voxela (sprawdzone: identyczne przy
0.166 i 0.08). Fix: jawnie `NavMeshModifier` NotWalkable na takim niskim blacie + re-bake, wtedy sie
wycina i agent obchodzi. (Diagnoza: probkuj `NavMesh.SamplePosition` na footprincie obiektu - jesli
siatka jest tuz przy nim, znaczy ze NIE jest wyciety.)

## Uwaga narzedziowa
Re-bake z poziomu skryptu: metoda `[MenuItem]` bywa `private` - `execute_script`/refleksja
zawola tylko `public static`. Zmien na public albo `EditorApplication.ExecuteMenuItem("<menu>")`.
