---
title: Collider z GetComponentInChildren&lt;MeshFilter&gt; na wielosiatkowym FBX = collider z fragmentu modelu
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-07-10'
project: Timber_Tycoon
tags:
- unity
- collider
- meshcollider
- fbx
- prefab-generator
- physics
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Collider z GetComponentInChildren&lt;MeshFilter&gt; na wielosiatkowym FBX = collider z fragmentu modelu

## Anty-wzorzec
Generator prefabow robil:
`mc.sharedMesh = go.GetComponentInChildren<MeshFilter>().sharedMesh` (convex MeshCollider).
GetComponentInChildren zwraca PIERWSZA siatke w hierarchii. Modele z Blendera maja czesto
2+ siatek (np. Crown + Trunk osobno, bo 1 material = 1 mesh). Collider objal wiec tylko korone:
obiekt zapadal sie pniem w teren, a fizyka odbijala go od colliderow otoczenia (PhysX depenetration
do 10 m/s przy domyslnym maxDepenetrationVelocity).

## Dlaczego nie dziala
- Liczba i kolejnosc siatek w FBX to szczegol eksportu z Blendera - nie ma gwarancji, ktora bedzie pierwsza.
- Blad jest niewidoczny w edytorze (collider "jest"), objawia sie dopiero fizyka w grze.

## Poprawny wzorzec
Zbudowac laczona siatke ze WSZYSTKICH MeshFilterow (CombineInstance z transformami wzgledem roota,
CombineMeshes(mergeSubMeshes:true), tylko wierzcholki+trojkaty), osadzic jako sub-asset prefabu
("{Nazwa}_ColliderMesh"), przypisac jako convex MeshCollider na roocie. Convex cooking i tak
przytnie hull do limitu PhysX (255 scian), wiec nie trzeba decymowac recznie.
Jesli system pickupu zaklada jeden collider (GetComponent<Collider> na roocie) - pilnowac
inwariantu "dokladnie 1 collider na roocie".

## Diagnostyka bez otwierania Blendera
Nazwy siatek w binarnym FBX da sie odczytac regexem w PowerShell:
`[regex]::Matches($text, '([\x20-\x7E]{3,60})\x00\x01(Model|Geometry)')` na bajtach pliku
(encoding ISO-8859-1). Pozwala policzyc siatki i zweryfikowac strukture 30 FBX w kilka sekund.

## Weryfikacja mechaniczna
Audyt-skrypt porownuje bounds siatki collidera z laczonymi bounds renderowanych siatek
(dol hulla w granicy 0.05 od dolu modelu, pokrycie wysokosci >= 90%) - wykrywa collider
z fragmentu modelu bez wchodzenia do Play Mode.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[dynamic-rigidbody-no-nonconvex-meshcollider|Dynamic Rigidbody → Primitive or Convex Collider, Never Non-Convex Mesh]] - wspolne: meshcollider, collider, physics
- [[mesh-collider-convex-for-clickable-minigame-objects|Convex MeshCollider for Irregular Clickable Objects]] - wspolne: meshcollider, collider
- [[20260628-1615-footstep-surface-raycast-from-feet|Detekcja nawierzchni pod graczem: strzelaj raycastem OD STÓP, nie od środka postaci]] - wspolne: meshcollider, physics
- [[20260628-1702-editmode-collider-cook-scatter-mask|Edit-mode placement that excludes geometry via physics fails for non-convex runtime/embedded MeshColliders]] - wspolne: meshcollider, physics
- [[20260713-1420-convex-meshcollider-swallows-hollow-interiors|Convex MeshCollider na skorupie połyka jej wnętrze - promienie nigdy nie trafią w części w środku]] - wspolne: meshcollider, physics
- [[20260713-1425-runtime-meshcollider-needs-readable-mesh-in-builds|MeshCollider tworzony w runtime działa w Edytorze i cicho pada w buildzie (isReadable)]] - wspolne: meshcollider, physics
<!-- /POWIAZANE:auto -->
