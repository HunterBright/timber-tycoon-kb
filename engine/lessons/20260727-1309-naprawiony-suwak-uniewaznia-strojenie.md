---
title: Naprawa suwaka, ktory po cichu klamal, uniewaznia CALE wczesniejsze strojenie
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-27'
project: Kerf - Sawmill Tycoon
tags:
- proceduralne
- narzedzia
- suwaki
- blender
- ux-narzedzi
- regresja-wizualna
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Naprawa suwaka, ktory po cichu klamal, uniewaznia CALE wczesniejsze strojenie

## Objaw

Uzytkownik ocenil wynik jako "duzo gorszy" bezposrednio po poprawce, ktora
z technicznego punktu widzenia byla czystym zyskiem (ten sam koszt, kilka razy
wieksze pokrycie). Nic sie nie zepsulo - a jednak efekt byl gorszy.

## Przyczyna

Suwak "liczba skupisk" w generatorze drzew rozdzielal N zadanych skupisk na M
istniejacych punktow zaczepienia. Gdy N > M, nadmiar siadal DOKLADNIE na tych
samych wspolrzednych. Przy N=1300 i M=526 uzytkownik dostawal w praktyce 526
widocznych obiektow, mimo ze suwak pokazywal 1300.

Uzytkownik przez wiele iteracji stroil ten suwak OKIEM - podkrecal go, bo
"nie widac roznicy", az doszedl do 1300. Ta wartosc byla poprawna WYLACZNIE
w swiecie zepsutego suwaka.

Naprawa (rozsuniecie nadmiaru wzdluz gałezi) sprawila, ze suwak zaczal robic
to, co obiecuje - i uzytkownik natychmiast dostal 2,5 raza wiecej geometrii
niz kiedykolwiek widzial. Efekt: przegeszczenie.

## Regula

**Kazda wartosc, ktora uzytkownik dostroil okiem, jest zwiazana z wersja kodu,
w ktorej ja stroil.** Gdy naprawiasz parametr, ktory wczesniej po cichu
przycinal, dublowal albo ignorowal wejscie:

1. Nie oddawaj samej naprawy do oceny. Oddaj naprawe RAZEM z przeliczonymi
   wartosciami (tu: 1300 -> ok. 500, bo tyle realnie dzialalo wczesniej).
2. Powiedz wprost, ktore ustawienia trzeba przestroic i dlaczego - inaczej
   uzytkownik oceni naprawe jako regresje, bo dla niego to jedyne, co widzi.
3. Jesli nie wiadomo, jaka wartosc jest teraz wlasciwa, wygeneruj DRABINKE
   (3-4 warianty obok siebie na jednym obrazku) zamiast iterowac po jednym.
   Przy parametrze czysto gustowym jedna runda z czterema opcjami jest tansza
   niz cztery rundy z jedna.

## Sygnal ostrzegawczy

Jesli w historii projektu widzisz, ze uzytkownik podkrecal jakis suwak
schodkowo coraz wyzej ("80 -> 200 -> 900 -> 1600") - to prawie zawsze znaczy,
ze suwak gdzies po drodze przestal dzialac. Podkrecanie jest objawem, nie
preferencja. Zmierz, ile faktycznie trafia do wyniku, ZANIM zaproponujesz
kolejna wartosc.

## Powiazane

- Ta sama rodzina co ciche przycinanie zakresu suwaka (`min`/`max` w tabeli
  parametrow), ktore rowniez powodowalo, ze zadana liczba nie docierala do
  generatora.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260726-1810-ciagla-powloka-zamiast-osobnych-bryl|"Zle przyklejone konczyny" to nie blad ustawienia, tylko blad architektury]] - wspolne: proceduralne, blender
- [[20260727-0525-jeden-suwak-dwie-role|Jeden suwak sterujacy dwiema roznymi rzeczami]] - wspolne: suwaki, blender
- [[20260731-2200-slepa-dzwignia-debugger-bramek|20260731-2200-slepa-dzwignia-debugger-bramek]] - wspolne: proceduralne, blender
- [[20260801-0500-gestszy-pomiar-odslania-dlug|20260801-0500-gestszy-pomiar-odslania-dlug]] - wspolne: proceduralne, blender
<!-- /POWIAZANE:auto -->
