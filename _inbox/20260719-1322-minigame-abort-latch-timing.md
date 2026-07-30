---
type: anti-pattern
project: Timber_Tycoon
suggested-category: engine/anti-patterns
tags: [minigame, abort, exploit, game-economy, state-machine, unity]
date: 2026-07-19
status: draft
---

# Zatrzask "punkt bez odwrotu" ustawiany przy wejsciu w faze zamiast przy akcji gracza

## The trap
Minigra z celowym designem anty-reroll ("przerwanie po starcie = czesciowy wynik, nie zwrot
surowca") ustawia zatrzask `scoringStarted = true` w orkiestratorze, w momencie WEJSCIA w faze
punktowana - a faza zaczyna sie od bramki "kliknij, aby zaczac". Wydaje sie rownowazne
("przeciez zaraz zacznie grac"), ale nie jest.

## Why it fails
Miedzy wejsciem w faze a pierwszym kliknieciem gracza istnieje okno (najazd kamery + bramka
startowa), w ktorym gracz nie zrobil NIC, a system juz traktuje sesje jako "gra sie toczy".
ESC w tym oknie przechodzi galezia "poddanie sie" i commituje produkt z zerowych punktow.
Efekt: "wejdz-wyjdz" produkuje gotowy przedmiot w ~5 s bez grania (w Timber Tycoon: mebel
Standard wart 125% surowca - exploit ekonomiczny szybszy per godzina niz uczciwa gra).

Drugi poziom pulapki: pola sesji (punkty, zatrzask) zerowane dopiero PO najezdzie kamery
(w srodku korutyny sesji), nie przy jej starcie. Kazda KOLEJNA sesja przez czas najazdu
widzi zwiechniete wartosci poprzedniej - ESC w tym oknie moze commitowac produkt z JAKOSCIA
z poprzedniej sesji (po jednym perfekcyjnym przejsciu: najlepszy przedmiot za sam ESC).

## Symptoms
- Gracz wchodzi do minigry, natychmiast ESC - a produkt i tak powstaje.
- Przerwanie w oknie "kliknij, aby zaczac" nie zwraca surowcow.
- Jakosc produktu z przerwania bywa rozna w zaleznosci od POPRZEDNIEJ sesji (stale state).

## Correct approach
1. Zatrzask "bez odwrotu" ustawiaj wylacznie na FAKTYCZNA akcje gracza (callback z fazy po
   pierwszym kliknieciu/inpucie), nigdy na wejscie w faze czy koniec przejazdu kamery.
2. Caly stan sesji (punkty, zatrzaski) zeruj SYNCHRONICZNIE na starcie sesji, przed
   uruchomieniem korutyny - zero okna, w ktorym stare wartosci sa widoczne dla ESC.
3. UWAGA na "poddanie sie = produkt najnizszej jakosci" jako anty-reroll: to wciaz jest
   szybki craft (klik start + ESC = produkt w kilka sekund, u nas 125% wartosci surowca).
   Ostrzejsza i szczelniejsza wersja (wybor rezysera w Timber Tycoon): abort po starcie =
   surowiec PRZEPADA, produkt NIE powstaje. Zabija szybki craft, a reroll kosztuje pelny
   surowiec za probe. Slaby, ale DOKONCZONY przebieg dalej daje produkt minimalnej jakosci,
   wiec uczciwy gracz nie jest karany.
4. Test regresyjny (build probe): (a) abort na bramce startowej = pelny refund + zero
   produktu; (b) abort 1 klatke po starcie DRUGIEJ sesji z rzedu = pelny refund (lapie
   zwiechniety stan); (c) abort po symulowanym pierwszym kliknieciu = zgodnie z decyzja
   designu (przepadek albo najnizsza jakosc), ale NIGDY pelnowartosciowy produkt.
   Test pisz PRZED fixem i udowodnij FAIL na starym kodzie (red-proof).
