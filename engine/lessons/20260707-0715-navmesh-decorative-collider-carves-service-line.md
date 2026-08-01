---
title: Dekoracyjny prop z colliderem na linii chodzenia wycina dziure w NavMesh i wypycha NPC do srodka budynku
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
- navmeshmodifier
- ignorefrombuild
- pathfinding
- collider
- props
applies_to: []
source: ''
severity: medium
time_lost: ''
promoted: '2026-07-30'
---

# Dekoracyjny prop z colliderem na linii chodzenia wycina dziure w NavMesh i wypycha NPC do srodka budynku

## Problem
Piesi NPC z parkingu zamiast isc "po linii" wzdluz frontu do lady, wchodzili w glab budynku i
"przechodzili przez lade". Zaczelo sie, odkad na scenie stanely dekoracyjne kupki (props) w okolicy
lad. Pathfinding wygladal na zepsuty, a kod trasy byl poprawny (NPC dostaje tylko `SetDestination`,
reszte liczy wypieczona siatka).

## Root cause
`NavMeshSurface` z `collectObjects = All` + `useGeometry = PhysicsColliders` wpieka collider KAZDEGO
aktywnego obiektu - takze malych dekoracji. Kilka kupek stalo dokladnie na linii, po ktorej klient
podchodzi do lady. Ich MeshCollidery **wycinaly dziury w chodliwej siatce na tej linii**. Skutki:
1. Prosta droga wzdluz frontu byla przerwana -> siatka prowadzila agenta na OBEJSCIE przez otwarte
   wnetrze budynku (efekt "wchodzi do srodka / przez lade").
2. Dziura pod samym punktem docelowym (customer stand) sprawiala, ze `SamplePosition` snapowal cel
   na dziwny, PODNIESIONY punkt obok (na wierzchu innego collidera) - dodatkowy chaos w dojsciu.

Diagnoza liczbowa (bez gizmo): `NavMesh.CalculatePath` z kazdego spawnu do celu + skan `SamplePosition`
po siatce prostokatnej wzdluz linii dojscia. Dziury w skanie pokrywaly sie 1:1 z pozycjami propsow.

## Solution
Wyklucz te propsy Z BUDOWANIA siatki, NIE ruszajac ich pozycji ani colliderow do gry:
- Na kazdy prop: `NavMeshModifier` z `ignoreFromBuild = true` (`applyToChildren = true`,
  `overrideArea = false`), potem **re-bake + zapis sceny**.
- Prop zostaje widoczny i dalej fizyczny dla gracza; znika tylko z NavMesh -> siatka pod nim jest
  ciagla -> NPC idzie prosto po linii.

Zweryfikowane: 12 pkt spawnu x 3 cele = 36 tras, wszystkie `PathComplete`, 0 wejsc w obrys, 0 przeciec
lady; linia frontu w skanie znow ciagla; lady dalej poprawnie carve'owane.

## What didn't work / czego NIE robic
- **NotWalkable na propsie to BLAD w tym przypadku.** `overrideArea = Not Walkable` wycina jeszcze
  WIEKSZA dziure (caly footprint) - pogarsza, bo o to wlasnie chodzilo. NotWalkable jest dla rzeczy,
  ktore MAJA blokowac (niski blat, plaska platforma). `ignoreFromBuild` jest dla rzeczy, ktore w ogole
  nie maja wplywac na siatke. To dwa rozne narzedzia - nie mylic.
- Przesuniecie propsow: odpada, gdy ich pozycje sa zatwierdzone/celowe. `ignoreFromBuild` rozwiazuje
  problem bez ruszania sceny.

## Rozroznienie (scial): trzy narzedzia NavMeshModifier / kolektora
- collider zostaje w bake (domyslnie) = **carve** (blokuje, dziura w footprincie) -> dla scian, wysokich
  przeszkod.
- `NavMeshModifier` NotWalkable = **carve na sile** nawet dla niskich/plaskich (ktore inaczej byly
  "wchodzalne") -> dla niskiej lady/platformy, po ktorej agent chodzil.
- `NavMeshModifier` `ignoreFromBuild` = **prop znika z siatki** (siatka ciagla pod nim) -> dla
  dekoracji, ktora ma NIE wplywac na trase (agent "ignoruje" prop).

## Transferability
Czysto silnikowe (Unity AI Navigation / NavMeshSurface). Dotyczy kazdego projektu, gdzie agenci
chodza po wypieczonej siatce, a na ich trasie stoja dekoracje/props z colliderami. Regula ogolna:
jesli prop nie ma blokowac agenta, wyklucz go `ignoreFromBuild`, nie kombinuj NotWalkable ani
przesuwaniem.

## Related
- [[20260706-1520-navmesh-raised-collider-invisible-bump]] - ten sam system: bake-all-colliders,
  voxel nad niskopoly terenem, NotWalkable na niska lade. Tu doklejam trzeci przypadek: ignoreFromBuild
  dla dekoracji na linii chodzenia.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260706-1520-navmesh-raised-collider-invisible-bump|NPC chodza po "niewidzialnych gorkach": bake NavMesh z propsow + za gruby voxel nad niskopoly terenem]] - wspolne: ai-navigation, navmeshsurface, pathfinding
- [[20260707-1130-navmesh-fine-voxel-micro-gap-route-detour|Za drobny voxel NavMesh tworzy mikro-dziure, ktora ROZSPAJA trase i wymusza wielki objazd]] - wspolne: ai-navigation, navmeshsurface, pathfinding
<!-- /POWIAZANE:auto -->
