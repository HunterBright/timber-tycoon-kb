---
title: 'Modele generowane przez AI (Tripo/Hunyuan): dług geometryczny - czasem taniej zbudować od zera'
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-07-19'
project: Kerf - Sawmill Tycoon
tags:
- blender
- tripo
- ai-generated
- mesh
- cleanup
- non-manifold
- asset-debt
- rebuild-vs-patch
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Modele generowane przez AI (Tripo/Hunyuan): dług geometryczny - czasem taniej zbudować od zera

## Kontekst
Auto NPC w Timber Tycoon (model z Tripo AI) miało prześwity. Audyt wykazał, że karoseria
jest zdrowa (naprawa = dwustronne materiały), ale koła to bałagan: WheelLF miał 47 krawędzi
prawdziwych dziur + 32 niezgrzane szwy, cały model był non-manifold i niezespawany (474 v
z 18 duplikatami tylko na jednym kole). Naprawa w Blenderze SIĘ UDAŁA technicznie (dziury
zaszyte, szwy zgrzane, zero dryfu), ale to była druga runda łatania tego samego modelu -
i reżyser zdecydował: "zbudujmy nowy zamiast łatać co chwilę ten badziewny".

## Anty-wzorzec
Iteracyjne łatanie modelu wygenerowanego przez AI, który ma STRUKTURALNY dług
(non-manifold, niezespawane wierzchołki, wielokrotne dziury w wielu miejscach). Każda
runda łata jeden objaw; następny playtest odsłania kolejny. Model AI nie ma czystej
topologii, więc defekty są rozsiane i nie kończą się.

## Kiedy łatać, a kiedy budować od zera
- **Łataj**, gdy defekt jest LOKALNY i jednorazowy (jedna dziura, jeden odwrócony face) i
  reszta siatki jest czysta.
- **Buduj od zera**, gdy audyt pokazuje ROZLANY dług: non-manifold w wielu skorupach,
  masowo niezespawane wierzchołki, dziury w kilku niezależnych miejscach. Wtedy koszt
  kolejnych rund łatania (każda: Blender + re-eksport + re-wire + playtest) przewyższa
  jednorazowy czysty rebuild z kontrolowaną topologią.

## Jak się zabezpieczyć na przyszłość
1. Przy imporcie modelu AI od razu odpal audyt geometrii (boundary edges, non-manifold,
   duplikaty) - liczba defektów mówi, czy to kandydat do łatania czy do rebuildu.
2. Zostaw kontrakt niezależny od modelu (u nas: test EditMode wymuszający dwustronne
   materiały + slot lakieru + komplet remapów) - wtedy NOWY model musi spełnić te same
   wymagania, a wpięcie go jest przewidywalne.
3. Naprawiony wariant trzymaj jako referencję/rollback, nawet jeśli finalnie budujesz od
   nowa - pokazuje docelową topologię i potwierdza, co było zepsute.

## Powiązane
Materiał dwustronny (_Cull:0) leczy prześwit CIENKIEJ SKORUPY, ale NIE domyka brakującej
ścianki - to dwa różne defekty. Patrz [[20260719-1605-paper-shell-culling-seethrough]].

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[tripo-asymmetric-floating-retopo|Tripo Asymmetric / Floating Elements - Blender Retopo]] - wspolne: cleanup, tripo, blender
- [[tripo-cleanup-pipeline|Tripo Cleanup Pipeline]] - wspolne: cleanup, tripo, blender
- [[20260602-1500-mtree-nonmanifold-voxel-remesh|Reducing MTree (Modular Tree) meshes to low-poly: Decimate & Quadriflow refuse, Voxel remesh works]] - wspolne: non-manifold, blender
- [[character-pipeline-tripo-mixamo-unity|Character pipeline: Tripo mesh → Mixamo rig → Unity (clean, working recipe)]] - wspolne: tripo, blender
- [[20260730-1710-blender-materials-clear-resets-face-indices|Mesh.materials.clear() zeruje material_index na ściankach]] - wspolne: mesh, blender
- [[20260728-0915-fbx-skala-100-w-dzieciach-psuje-pomiary|FBX z Blendera: przelicznik jednostek siedzi w SKALI DZIECI, nie w korzeniu]] - wspolne: mesh, blender
<!-- /POWIAZANE:auto -->
