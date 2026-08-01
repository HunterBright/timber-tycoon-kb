---
title: Trzymanie boczne liczone z osi nadwozia zjada grawitacje przewroconemu autu
type: lesson
status: draft
confidence: high
verified: 2026-07-31
tags: [unity, fizyka, pojazdy, rigidbody, grawitacja, sonda]
date: 2026-07-31
project: Kerf - Sawmill Tycoon
source: zdjecia testera 2026-07-27 (jaskinia za wodospadem, barierka mostu)
applies_to: [unity, dowolny arkadowy kontroler pojazdu na Rigidbody]
severity: high
time_lost: ok. 3 h (w tym 5 slepych wersji bramki)
---

# Trzymanie boczne liczone z osi nadwozia zjada grawitacje przewroconemu autu

## Objaw
Auto przewrocone na bok (oparte o sciane jaskini albo barierke mostu) przy wcisnietym gazie
"zaczyna jechac do nieba". Gracz zglasza to jako zepsuta grawitacje pojazdu.

## Przyczyna
Arkadowy kontroler kasuje poslizg boczny:

    Vector3 sideDir = Vector3.Cross(transform.up, fwd).normalized;
    rb.linearVelocity -= sideDir * Vector3.Dot(rb.linearVelocity, sideDir) * lateralGrip;

`sideDir` liczy sie z osi NADWOZIA. Dopoki auto stoi na kolach, jest pozioma i wszystko gra.
Gdy auto lezy na boku, ta os staje sie **PIONOWA** - a `lateralGrip` = 0,9 kasuje 90% predkosci
wzdluz niej co krok fizyki. Czyli kasuje spadanie.

Grawitacja dokłada 0,196 m/s na krok (9,81 x 0,02), trzymanie zabiera 90% - auto ustala sie na
opadaniu ~0,22 m/s zamiast swobodnego spadku. Praktycznie wisi. Do tego lezace auto dalej ma
promienie spod kol skierowane w dol, wiec gra uznaje je za stojace na ziemi i **dalej daje mu gaz**
wzdluz zadartej osi nadwozia. Nic go nie sciaga w dol - leci w gore.

Grawitacja byla przez caly czas poprawna (`rb.useGravity = true`, zero wlasnego kodu).

## Rozwiazanie
1. Splaszczyc os trzymania do poziomu: `sideDir = Vector3.ProjectOnPlane(sideDir, Vector3.up)`.
   Trzymanie boczne NIE MOZE dotykac pionu - grawitacja to nie poslizg.
2. Bramka pionu: auto przechylone ponad 60 stopni (`Dot(transform.up, Vector3.up) < 0.5`) nie
   dostaje gazu, skretu ani trzymania. Spada i slizga sie jak zwykly przedmiot.
3. Osobno: samo prostowanie po ~1,5 s lezenia bez ruchu (plynny obrot do normalnej terenu,
   kurs zachowany, bez teleportu).

## Co NIE zadzialalo (5 slepych wersji bramki)
Bramka bez udowodnionego trybu porazki przechodzila mimo zepsutej gry - piec razy z rzedu,
za kazdym razem z innego powodu:
1. **Sterowanie predkoscia zamiast gazem** - podmiana `rb.linearVelocity` co klatke robi ze
   sondy spychacz, ktory wjezdza po kazdej skale (13,5 m!) i omija bramke gazu. Test sterujacy
   pojazdem MUSI isc przez jego wlasny gaz.
2. **Zly obrot** - `Euler(0,0,90)` to postawienie auta na NOSIE, nie na boku. Podejrzana os
   zostawala pozioma i blad sie nie odtwarzal. Bok to obrot wokol osi JAZDY.
3. **Za dlugie okno pomiaru** - przy 0,5 s auto zdazylo wyladowac i pomiar mierzyl zderzenie
   z gruntem, nie spadanie. 0,16 s (12 cm) rozdzielilo wyniki.
4. **Prog zbyt luzny** - "wrocilo na kola" przy pionie 0,50 zaliczalo przypadkowe przewalenie.
5. **Poza niestabilna** - auto polozone na BOKU sama fizyka przewraca na kola w 1,3-2,1 s
   (niski srodek ciezkosci), wiec test prostowania przechodzil takze z wylaczona naprawa.
   Dowod wymaga pozy STABILNEJ: na dachu.

## Wzorzec pomiaru, ktory zadzialal
POROWNAWCZY zamiast progu z palca: to samo auto, ta sama wysokosc, raz na kolach (wzorzec),
raz na boku z gazem do dechy. Grawitacja nie zna pojecia "przechyl", wiec oba spadki musza byc
podobne. Znika zaleznosc od oporu powietrza, wysokosci i masy.

## Dowod
Sonda w prawdziwym buildzie, przebiegi 2026-07-31:
- `Bok/grawitacja` zielona: na boku -1,02 m/s, wzorzec na kolach -1,31 m/s -> PASS
- `Bok/grawitacja` z `-carsideredproof` (stara os): na boku **-0,21 m/s** -> FAIL
- `Bok/prostowanie` zielona: auto na dachu wstaje po 1,8 s -> PASS
- `Bok/prostowanie` z `-carrightredproof`: lezy 5 s, pion -0,97 -> FAIL

## Czy to przeniesie sie na inny projekt
Tak. Kazdy arkadowy kontroler pojazdu kasuje poslizg boczny w ten sposob, a niemal kazdy liczy
os z `transform`. Ogolna zasada: **kazda sila liczona z osi obiektu przestaje mieć sens, gdy
obiekt przestaje byc w zalozonej pozycji** - i im mocniejszy wspolczynnik, tym gorzej.

## Powiazane
- [[gate-must-have-provable-failure-mode]]
- [[20260731-1600-domyslne-wypychanie-z-przenikania-wystrzeliwuje-pojazdy]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260731-1600-domyslne-wypychanie-z-przenikania-wystrzeliwuje-pojazdy|Domyslne wypychanie z przenikania wystrzeliwuje pojazdy w niebo]] - wspolne: fizyka, pojazdy, sonda
<!-- /POWIAZANE:auto -->
