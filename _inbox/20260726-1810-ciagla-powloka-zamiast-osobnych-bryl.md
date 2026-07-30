---
title: '"Zle przyklejone konczyny" to nie blad ustawienia, tylko blad architektury'
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-26'
project: Kerf - Sawmill Tycoon
tags:
- blender
- postac
- topologia
- low-poly
- cieniowanie
- proceduralne
applies_to: []
source: ''
suggested-category: blender/lessons
---

# "Zle przyklejone konczyny" to nie blad ustawienia, tylko blad architektury

## Objaw

Proceduralna postac low-poly, w ktorej rece, nogi i glowa sa osobnymi zamknietymi
brylami wetknietymi w tulow. Rezyser ocenil to slowami: "rece i nogi bardzo zle
przyklejone do korpusu, glowa bardzo zle lezy na szyi". Wszystkie kontrole liczbowe
byly zielone: wymiary poprawne, stopy na zerze, symetria idealna.

## Blad, ktory latwo popelnic

Pierwszy odruch to poprawiac USTAWIENIE bryl: wsunac ramie glebiej, dopasowac
promien, dodac zgrubienie w miejscu styku. To nie moze zadzialac. Dwie osobne
zamkniete powierzchnie przecinajace sie w przestrzeni ZAWSZE pokazuja linie
przeciecia, bo tam nie ma wspolnych wierzcholkow i cieniowanie po obu stronach
liczy sie niezaleznie. Ile by nie poprawiac pozycji, krawedz zostaje.

## Wlasciwe rozwiazanie

Jedna ciagla powloka. Konkretnie:

1. **Jedna tuba od kroku po czubek glowy.** Miednica, tulow, szyja i glowa to
   kolejne pierscienie TEJ SAMEJ tuby. Dzieki temu "stopnia pod broda" nie ma
   gdzie powstac - to jest ta sama powierzchnia.
2. **Konczyny przez wyciety otwor.** W tubie usuwa sie prostokatny platek scianek,
   z jego brzegu bierze petle wierzcholkow i zszywa ja z pierwszym pierscieniem
   konczyny. Warunek: obwod platka musi rownac sie liczbie segmentow konczyny.
   Platek 3x3 scianki daje obwod `2*(3+3) = 12`, czyli konczyne 12-segmentowa.
3. **Kontrola, ktora to udowadnia:** liczba WYSP (spojnych skladowych grafu
   krawedzi) musi wynosic 1. To jedyna kontrola, ktora odroznia "wetkniete"
   od "wrosniete" - szczelnosc, symetria i objetosc przechodza w obu przypadkach.
   Dowod trybu porazki: wylaczyc zszywanie, wyspy skacza z 1 na 5.

## Drugi wniosek: kanciastosc to najczesciej cieniowanie, nie gestosc

Rownolegla uwaga brzmiala "wszystko dalej zbyt kanciaste". Zwiekszenie gestosci
siatki dwukrotnie tego NIE naprawilo - sprawdzone renderem. Przyczyna byla inna:
model mial 100% scianek PLASKICH, a wzorzec (istniejace modele w tej samej grze)
100% GLADKICH. Zmiana cieniowania zalatwila wiecej niz podwojenie liczby trojkatow.

**Zasada:** zanim zaczniesz dokladac geometrie, zmierz wzorzec. Trzy liczby
wystarczyly, zeby postawic diagnoze wszystkich trzech uwag naraz:

| | model odrzucony | wzorzec z gry |
|---|---|---|
| trojkaty | 1332 | 4810 |
| scianki gladkie | 0% | 100% |
| liczba obiektow | 10 | 1 |

## Pulapki po drodze

- **Lustrzane odbicie odwraca normalne.** Po odbiciu trzeba odwrocic kolejnosc
  wierzcholkow w kazdej sciance. Kontrola: objetosc ze znakiem (tw. o dywergencji)
  musi byc dodatnia.
- **Spawanie po siatce tolerancji nie scali wartosci roznych o jeden bit.**
  Pierscienie trzeba symetryzowac co do bitu, a nie liczyc osobno dla obu stron.
- **Zblizone pierscienie tworza zalamanie.** Warunek "sasiednie pierscienie nie
  roznia sie szerokoscia wiecej niz 1,2x" usuwa ostra kreske na szczece skuteczniej
  niz jakiekolwiek wygladzanie.

## Wniosek przenosny

Gdy krytyka dotyczy MIEJSCA STYKU dwoch czesci, sprawdz najpierw, czy te czesci sa
w ogole jedna powierzchnia. Jesli nie, zadna korekta polozenia nie pomoze i cala
runda poprawek pojdzie w bloto.
