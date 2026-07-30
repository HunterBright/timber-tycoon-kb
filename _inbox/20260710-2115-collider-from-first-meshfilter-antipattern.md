---
title: Collider z GetComponentInChildren&lt;MeshFilter&gt; na wielosiatkowym FBX = collider z fragmentu modelu
type: anti-pattern
status: draft
confidence: low
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
suggested-category: engine/anti-patterns
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
