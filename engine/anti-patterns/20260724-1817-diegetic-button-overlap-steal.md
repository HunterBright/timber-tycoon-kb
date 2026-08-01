---
title: Powiekszone collidery klikania ciasnych guzikow 3D + wybor "pierwsze trafione pudelko" = sasiad kradnie klik
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-07-24'
project: Kerf - Sawmill Tycoon
tags:
- unity
- raycast
- minigame
- diegetic-ui
- collider
- input
- picking
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Powiekszone collidery klikania ciasnych guzikow 3D + wybor "pierwsze trafione pudelko" = sasiad kradnie klik

## Problem
Diegetyczne guziki 3D (konsola maszyny) klikane raycastem z kamery. Guziki male i ciasno
obok siebie (odstep srodkow 8 cm). Kod dokladal im w runtime BoxCollidery powiekszone
pad=1.6 ("zeby latwiej bylo trafic") i wybieral guzik po NAJMNIEJSZEJ odleglosci wejscia
promienia w collider (RaycastAll + min hit.distance).

Efekt: pudelka 12 cm przy odstepie 8 cm NACHODZA na siebie. Kamera minigry patrzy z ukosa,
wiec promien celujacy w guzik SRODKOWY najpierw wbija sie w wystajace pudelko sasiada
od strony kamery. Pomiar (sonda batchmode, celowanie w 72 wierzcholki siatki guzika):
50% klikniec w srodkowy guzik rejestrowalo sie jako sasiedni. Gracz: "przycisk czasem
nie dziala" - najbardziej cierpi guzik SRODKOWY (ma zlodziei z obu stron).

## Dlaczego to podstepne
- Kazdy guzik z osobna "ma collider i da sie trafic" - blad widac dopiero przy ukosnej kamerze.
- Powiekszanie pol klikania (dobre dla malych celow) i wybor po hit.distance (naturalny
  default) sa OSOBNO poprawne; razem przy ciasnym rozstawie daja kradziez klikow.
- Nawet colliery BEZ powiekszenia nie leczą w pelni: pudelko ma GLEBOKOSC, ukosny promien
  potrafi przejsc przez dwa nienachodzace pudelka i "pierwsze wejscie" znow faworyzuje
  pudelko od strony kamery.

## Rozwiazanie
Dla znanego, malego zbioru guzikow: wybor po odleglosci NA EKRANIE od srodka guzika
(WorldToScreenPoint srodka wizualnego guzika vs pozycja myszy; prog = ~1.4 x rzutowany
promien guzika; wygrywa najblizszy srodek = podzial Voronoi). To mierzy dokladnie to,
w co gracz WIZUALNIE celuje. Collidery przestaja byc potrzebne do hoveru.
Pomiar po zmianie: wszystkie widoczne piksele kap guzikow rozwiazuja sie poprawnie.

Uwaga przy modelach FBX z pivotem w zerze modelu: srodek wizualny guzika liczyc
z granic siatki (renderer.bounds.center / TransformPoint(mesh.bounds.center)),
NIE z transform.position (potrafi byc o metry od guzika).

## Metoda diagnozy (przenosna)
Jednorazowy skrypt edytorowy w -batchmode: otworz scene (bez zapisu), odtworz logike
hoveru 1:1, wystrzel promienie z pozycji kamery minigry w kazdy wierzcholek siatki guzika,
policz SELF / INNY_GUZIK / PUDLO. Twarde liczby zamiast zgadywania; ten sam skrypt
sluzy potem jako dowod, ze naprawa dziala (i jako red-proof do sondy buildowej).

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260629-1917-diegetic-buttons-frame-minigame|Console buttons FRAME a skill minigame (they're flow-control, not the mechanic)]] - wspolne: diegetic-ui, minigame, raycast
- [[mesh-collider-convex-for-clickable-minigame-objects|Convex MeshCollider for Irregular Clickable Objects]] - wspolne: collider, minigame, raycast
- [[20260614-1343-singleton-oneshot-flag-bleed|One-shot input flag on a persistent singleton bleeds across re-entries]] - wspolne: input, minigame
- [[20260615-0913-delayed-completion-coroutine-needs-singleshot-latch|A delayed-completion coroutine that still reads input double-fires without a single-shot latch]] - wspolne: input, minigame
- [[20260614-1226-modal-ui-over-world-interactable-guard|Modal UI opened from a world-space interactable must guard the interaction handler]] - wspolne: input, raycast
- [[20260713-0830-primitive-to-fbx-swap-kills-interaction|Podmiana prymitywu Unity na model FBX po cichu zabija interakcję]] - wspolne: collider, raycast
<!-- /POWIAZANE:auto -->
