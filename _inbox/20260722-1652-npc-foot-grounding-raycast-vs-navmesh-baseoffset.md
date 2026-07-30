---
type: pattern
project: Timber Tycoon
suggested-category: engine/patterns
tags: [unity, navmesh, navmeshagent, baseoffset, npc, animation, grounding, skinnedmesh]
date: 2026-07-22
status: draft
---

# Przyklejanie stop NPC do gruntu raycastem zamiast stalej korekty NavMeshAgent.baseOffset

## When to use
Gdy NPC sterowani NavMeshAgentem tona w podlozu albo lewituja, a skala bledu zmienia sie
z MIEJSCA (plaska posadzka kontra nierowny teren) lub z MODELU (rozne rigi, rozne buty).
Objaw diagnostyczny: jedna stala korekta baseOffset daje dobry wynik w jednym miejscu i zly
w drugim, a "dostrajanie liczby" tylko przesuwa problem.

## Steps
1. Ustal, ze siatka NavMesh NIE lezy na ziemi. Przy wypalaniu z voxelizacji unosi sie nad
   geometria nierowno (u nas 0..0,12 m przy voxelu 0,12). baseOffset to stala, wiec nie moze
   pokryc zmiennego unosu.
2. Zostaw dotychczasowa stala jako punkt startowy i wartosc awaryjna (gdy pomiar nie trafi
   w grunt) - nie usuwaj jej, bo bez niej NPC bez gruntu pod soba skacze.
3. Komponent na NPC, co ~0,15 s:
   - raycast w dol po realny grunt (collider podlogi),
   - odczyt biezacej wysokosci PODESZWY,
   - `agent.baseOffset += (grunt - podeszwa)`, dojazd przez `Mathf.MoveTowards` (~1 m/s),
   - clamp do +/-0,3 m wzgledem stalej startowej (bezpiecznik na zle trafienie).
4. Podeszwa = kosc palcow stopy (zywy sygnal: w chodzie sama sledzi POSTAWIONA noge, bo
   bierzemy minimum z obu stop) + zmierzony RAZ NA MODEL odstep kosc->spod buta.
   Odstep licz z `SkinnedMeshRenderer.BakeMesh` (realne wierzcholki), NIE z `renderer.bounds`
   (obrys jest zawyzony, u nas o 9-18 cm) i NIE z bind-pose FK (Unity normalizuje biodra
   awatarow, szacunki byly o kilkanascie cm obok).
   Odstep licz tylko z pasa ~20 cm przy podlodze, inaczej najnizszym punktem zostanie fartuch
   albo plaszcz i NPC zacznie sie podnosic, by pola dotykala ziemi.
5. Wynik trzymaj w statycznym slowniku per model (klucz: nazwa awatara FBX). Instancje sa
   poolowane, modeli jest kilkanascie - jeden bake na model na sesje.

## Why this works
Bledy skladaja sie z dwoch niezaleznych czesci: unos siatki nawigacyjnej (zalezny od MIEJSCA)
i geometria buta wzgledem punktu zerowego postaci (zalezna od MODELU). Stala moze pokryc
najwyzej jedna z nich i tylko srednio. Petla sprzezenia zwrotnego mierzy sume obu w kazdej
chwili i w kazdym miejscu, wiec jest odporna takze na przyszle modele i przepieczona siatke.

## Trade-offs
- Jeden raycast na NPC co ~0,15 s (kilkanascie NPC = kilkadziesiat raycastow/s, pomijalne),
  plus jeden BakeMesh na model na sesje.
- Wyrownanie idzie do COLLIDERA podlogi. Jesli collider rozjezdza sie z widoczna siatka
  (u nas byl taki przypadek terenu), pomiar pokaze zero, a oko dalej zobaczy szpare - to
  osobny bug do zamkniecia, nie wada wzorca.
- GOTCHA: promien MUSI startowac nisko nad stopami (u nas 0,35 m) i odrzucac trafienia
  wyzej niz ~0,2 m nad stopa. Z wysokosci pasa trafi w blat lady albo obudowe maszyny,
  nad ktora NPC stoi, uzna je za "grunt" i WYNIESIE NPC na mebel.

## Variants
- Bez kosci palcow (rigi bez palcow): fallback na kosci stop, odstep kosc->podeszwa wyjdzie
  wiekszy, mechanizm bez zmian.
- Ciezsza wersja: `BakeMesh` co probke zamiast kosci - dokladniejsze, ale alokuje i skaluje sie
  z liczba NPC; niepotrzebne, dopoki but jest bryla sztywna wzgledem kosci.
- Postacie bez NavMeshAgenta (CharacterController): ta sama petla, tylko korekta idzie w
  pozycje Y zamiast w baseOffset.
