---
title: Miara optymalizowana samotnie znajduje rozwiazanie zdegenerowane
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-07-28'
project: Kerf - Sawmill Tycoon
tags:
- miary
- optymalizacja
- dopasowanie
- 3d
- bramki
- pomiar
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Miara optymalizowana samotnie znajduje rozwiazanie zdegenerowane

## Co sie stalo

Dopasowywalem siatke A do siatki B, szukajac kata obrotu ramienia. Miara:
**srednia odleglosc kazdego wierzcholka A do najblizszego punktu powierzchni B**.
Wydawala sie oczywista i niewinna.

Przemiatanie katow dalo:

```
  os    kat   blad[mm]  zasieg[m]
    Z   -30       42.2      0.579     <- POPRAWNE dopasowanie
    Z   +30       31.4      0.217     <- "najlepszy" wynik
```

Wygral kat +30 z bledem 31 mm, czyli o jedna czwarta lepszym niz rozwiazanie
poprawne. **Rece obrocily sie do wewnatrz i schowaly w tulowiu**, gdzie kazdy
ich wierzcholek mial blisko do powierzchni klatki piersiowej. Miara byla
liczona poprawnie. Odpowiadala na pytanie, ktore zadalem. Pytanie bylo zle.

Zdradzil to dopiero **zasieg ramienia**: 0,217 m przy 0,331 m w pozie
spoczynkowej. Rozwiazanie "lepsze" mialo rece krotsze niz przed optymalizacja.

## Dlaczego to jest wzorzec, a nie wpadka

Miary typu "odleglosc do najblizszego" maja **minimum w zapadnieciu**: im
bardziej obiekt sie skurczy i schowa wewnatrz drugiego, tym lepszy wynik.
To samo dotyczy calej rodziny:

- odleglosc do najblizszego punktu / powierzchni / sasiada
- pokrycie zbiorow, gdy jeden moze sie skurczyc
- srednia z bledow liczona po zbiorze, ktory sam zalezy od rozwiazania
- kazda kara bez odpowiadajacej jej nagrody za "rozmiar"

Wszedzie tam istnieje rozwiazanie zdegenerowane, ktore wygrywa nie dlatego,
ze jest dobre, tylko dlatego, ze **przestalo byc mierzone**.

## Reguly

1. **Nigdy nie optymalizuj miary "odleglosci do" samotnie.** Musi jej
   towarzyszyc wielkosc, ktora ROSNIE, gdy rozwiazanie degeneruje: rozmiar,
   zasieg, objetosc, liczba pokrytych elementow. Wypisuj ja w tej samej
   tabeli, nawet jesli nie wchodzi do wzoru.
2. **Zbior porownywanych elementow ustalaj RAZ, przed optymalizacja.**
   W tej samej sesji drugi blad: wierzcholki reki wybieralem po wspolrzednej
   przy KAZDYM kacie, wiec ich liczba spadala z 404 do 102 w miare chowania
   rak, a "sredni blad" liczyl sie na garstce resztek i wychodzil tym
   mniejszy, im gorzej bylo naprawde.
3. **Patrz na tabele, nie na minimum.** Gdyby kod wypisal tylko "najlepszy
   kat: +30", wzialbym to za dobra monete. Zdradzil to dopiero sasiedni
   slupek, ktorego nie musialem drukowac.
4. **Sprawdz, czy rozwiazanie idealne wypada lepiej niz zdegenerowane.**
   Jesli nie - miara jest zla, niezaleznie od tego, jak rozsadnie brzmi.

## Powiazane

Ta sama sesja, ta sama rodzina bledow, druga strona medalu: prog bramki
ustawiony POWYZEJ zmierzonego sufitu sprawia, ze zadne rozwiazanie nie moze
przejsc. Patrz `20260728-1500-bramka-ponad-sufitem.md`.

Wspolny mianownik: **liczba, ktorej nikt nie skonfrontowal z przypadkiem
skrajnym, nie jest jeszcze miara.** Trzeba sprawdzic oba konce - co daje
rozwiazanie idealne i co daje rozwiazanie absurdalne.

## Gdzie to sie stalo

Kerf - Sawmill Tycoon, dopasowanie kupionej bazy postaci do siatki MakeHumana,
2026-07-28. Pelny opis: `Docs/KERF_GENERATOR_POSTACI.md` rozdzial 5.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260728-0030-wymiar-ktorego-nie-widzi-zadne-ujecie-dopasuj-do-wszystkich|Wymiar, którego nie widzi żadne ujęcie, mierzy się dopasowaniem do wszystkich naraz]] - wspolne: dopasowanie, pomiar
- [[20260727-0510-rozproszenie-kartek-punkty-zaczepienia|Rozsypywanie kartek po szkielecie: licz NOSNIKI, nie kartki]] - wspolne: optymalizacja, pomiar
- [[20260728-1320-mniej-wiekszych-kartek|Polowa kartek o rozmiarze wiekszym o jedna trzecia wyglada tak samo]] - wspolne: optymalizacja, pomiar
- [[20260728-1500-bramka-ponad-sufitem|Prog bramki ponad zmierzonym sufitem zamienia kazda runde w porazke]] - wspolne: bramki, pomiar
- [[20260801-0500-gestszy-pomiar-odslania-dlug|20260801-0500-gestszy-pomiar-odslania-dlug]] - wspolne: bramki, pomiar
- [[20260728-1140-miernik-ktory-klamie-inaczej|Zanim zaufasz bramce, sprawdz, czy mierzy to, co widac]] - wspolne: bramki, pomiar
<!-- /POWIAZANE:auto -->
