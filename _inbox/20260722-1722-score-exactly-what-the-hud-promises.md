---
title: Punktuj DOKLADNIE te wielkosc, ktora pokazujesz graczowi
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-22'
project: Kerf - Sawmill Tycoon
tags:
- game-feel
- scoring
- hud
- feedback
- minigame
- player-trust
applies_to:
- any-game-with-scored-input
source: ''
severity: medium
time_lost: ~1 tydzien zycia bledu w grze (wprowadzony przy polerowaniu, znaleziony przy przegladzie)
suggested-category: gamedesign/lessons
---

# Punktuj DOKLADNIE te wielkosc, ktora pokazujesz graczowi

## Problem
Minigra ciecia miala na miarce zielone pole. Przy polerowaniu dolozono "martwa strefe": dopoki
gracz siedzi w zielonym, ciecie idzie idealnie po linii i drewno wyglada bez zarzutu. Punktacja
zostala jednak stara - liczyla surowe odchylenie od srodka. Efekt: gracz widzi idealne ciecie,
a dostaje 60/100 i szara kropke zamiast zlotej (prog 80). Obietnica interfejsu byla trzy razy
hojniejsza niz wyplata.

Nikt tego nie zglosil jako bledu przez tydzien, bo kazdy element z osobna byl "poprawny":
strefa dzialala, punktacja dzialala, testy przechodzily.

## Root cause
Dwie osobne wielkosci opisywaly to samo doswiadczenie: (1) widoczny znos ciecia, liczony z
martwej strefy, (2) wynik, liczony z surowego odchylenia. Zmiana feelu ruszyla tylko pierwsza.
Klasyczny rozjazd "co widze" kontra "za co place" - powstaje niemal zawsze wtedy, gdy warstwe
wizualna poprawia sie po fakcie, bez tknięcia formuly wyniku.

## Solution
Wynik liczony z DOKLADNIE tej samej wielkosci, ktora steruje obrazem: bledem jest dopiero
wyjscie poza zielone pole (0 w srodku, 1 na granicy tolerancji). Cala strefa "wyglada idealnie"
= komplet punktow.

Dwie pulapki wykryte przy implementacji:
1. **Blad musi byc liczony PER KLATKA, nie z uśrednionego odchylenia.** Odejmowanie martwej
   strefy od sredniej daje komplet punktow za przebieg "pol drogi idealnie, pol daleko poza
   strefa" - a tam slad widocznie ucieka. Srednia z bledow != blad ze sredniej.
2. **Prog wizualnej nagrody (zlota kropka) trzeba wyciagnac ze stalych magicznych** i zwiazac
   sprawdzeniem z szerokoscia strefy, inaczej przy najblizszym strojeniu obie liczby znow
   rozjada sie po cichu.

## What didn't work
- Rozwazane "dopisanie podpowiedzi na HUD" (trzymaj sie SRODKA zielonego): to opisywanie
  slowami czegos, co ma wynikac z ksztaltu. Lata objaw, zostawia przyczyne.
- Zwezenie strefy do progu punktacji bylo poprawne technicznie i tansze (zero wplywu na
  ekonomie), ale odebralo by graczowi hojnosc, ktora wlasnie zostala dodana swiadomie. Wybor
  miedzy "zwez obietnice" a "zaplac za obietnice" to decyzja PROJEKTOWA, nie techniczna -
  nalezy do dyrektora kreatywnego, z jawnie policzonym skutkiem dla ekonomii.

## Transferability
Dotyczy kazdej gry, w ktorej wejscie gracza jest oceniane i jednoczesnie wizualizowane: rytmiczne,
celowanie, parkowanie, wedkarstwo, QTE, paski "perfect zone" w crafcie. Pytanie kontrolne przy
kazdej zmianie feelu: "czy wielkosc, ktora wlasnie zaczalem pokazywac/wybaczac, jest TA SAMA,
ktora liczy wynik?". Jesli nie - gracz nauczy sie ufac obrazowi i zostanie ukarany za to zaufanie.

Drugi wniosek, szerszy: **zmiany "kosmetyczne" potrafia zmieniac kontrakt z graczem.** Warto przy
nich sprawdzac formule wyniku, nawet gdy sie jej nie dotyka.

## Related
- 20260722-1652-relative-only-test-blind-to-common-mode-error.md
- [[gate-must-have-provable-failure-mode]]
