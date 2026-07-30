---
title: CharacterController + cienki jednostronny teren-siatka = gracz pod mapą (i jak się przed tym bronić)
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-16'
project: Kerf - Sawmill Tycoon
tags:
- unity
- charactercontroller
- physics
- depenetration
- terrain
- safety-net
- kill-plane
applies_to: []
source: ''
suggested-category: engine/lessons
---

# CharacterController + cienki jednostronny teren-siatka = gracz pod mapą (i jak się przed tym bronić)

## Problem
Animowany transformem collider (padające drzewo obracane `localRotation` klatka po klatce, bez
Rigidbody) wbił się w kapsułę gracza. CharacterController przy następnym `Move()` robi
auto-depenetrację: wypycha kapsułę wzdłuż osi najmniejszej penetracji. Gdy teren to CIENKA,
jednostronna siatka (custom mesh + niewypukły MeshCollider, nie Unity Terrain), ta oś potrafi
prowadzić POD powierzchnię. Po przebiciu `isGrounded=false`, ręczna grawitacja ciągnie w dół
i gracz spada w nieskończoność - o ile gra nie ma żadnego ratunku.

## Lekcja 1: collider animowany transformem to teleportująca się ściana
Fizyka nie "widzi" jego ruchu (brak prędkości), więc każda klatka to nowa penetracja statyczna.
Jeśli taki obiekt może dotknąć gracza, na czas animacji wyłącz pary kolizji:
`Physics.IgnoreCollision(animCol, playerCol, true)` (+ przywrócenie po chwili; UWAGA:
IgnoreCollision wymaga obu colliderów aktywnych i włączonych, inaczej loguje błąd; stan
ignorowania resetuje się przy deaktywacji collidera).

## Lekcja 2: każda gra z ręczną grawitacją potrzebuje siatki ratunkowej
Trzy warstwy (od taniej do ostatecznej):
1. Zapis ostatniej bezpiecznej pozycji: gdy `isGrounded` ORAZ promień w dół potwierdza
   "prawdziwy grunt" (maska warstw podłoża, ignoruj triggery) - throttle np. 0.5 s.
2. Detektor "pod mapą": (a) promień w dół z gracza NIE trafia podłoża w dużym zasięgu
   (kilkaset m), (b) stan trwa >= 1 s (odsiewa skoki), (c) POTWIERDZENIE Z NIEBA: promień
   z wysokości (x, 400, z) w dół trafia grunt POWYŻEJ gracza. GOTCHA: promień W GÓRĘ nie
   zadziała - raycast NIE trafia backface'ów jednostronnej siatki; dlatego sondujemy z nieba.
   Jaskinie/mosty nie dają fałszywek, jeśli ich podłogi są na warstwie podłoża (warunek (a)
   przechwytuje je wcześniej).
3. Awaryjne: twarde kill-Y (poniżej dna mapy) = ratunek natychmiast; timeout swobodnego
   spadania (np. 4 s bez gruntu) = ratunek nawet gdy sonda z nieba nie trafi.
Ratunek = teleport wzorem "CC-disable" (wyłącz CharacterController, ustaw pozycję, włącz,
wyzeruj prędkość pionową) + cooldown.

## Lekcja 3 (sonda w buildzie)
Check automatyczny: wyłącz siatkę (publiczny hak), teleportuj gracza pod teren, on MUSI tam
zostać (dowód, że check umie zawieść); włącz siatkę, teleportuj ponownie, gracz MUSI wrócić.
Ewaluator odzysku = NIEZALEŻNY promień w dół (nie testuj kodu nim samym).

**Why:** depenetracja CC + cienki teren to klasa bugów niezależna od gatunku gry; siatka
ratunkowa chroni też przed WSZYSTKIMI przyszłymi bugami tego typu, nie tylko znanym.
**How to apply:** w każdym projekcie FPP/TPP z CharacterController i customowym terenem-siatką
dodaj trójwarstwową siatkę ratunkową od pierwszego playtestu.
