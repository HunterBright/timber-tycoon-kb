---
title: Prog bramki ponad zmierzonym sufitem zamienia kazda runde w porazke
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-28'
project: Kerf - Sawmill Tycoon
tags:
- bramki
- testy
- progi
- jakosc
- iteracja
- pomiar
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Prog bramki ponad zmierzonym sufitem zamienia kazda runde w porazke

## Objaw

Sesja pracy nad modelem postaci rozsypala sie w ciag "nieudanych prob".
Kazda runda konczyla sie czerwona tablica, wiec kazda wygladala na krok
wstecz. Uzytkownik stracil zaufanie do calego kierunku i przerwal prace.

## Przyczyna

Bramka porownywala render modelu ze zdjeciem referencyjnym i wymagala
pokrycia sylwetek co najmniej 0,90. W tym samym pliku lezal ZMIERZONY sufit,
czyli wynik osiagany przez najlepszy ksztalt w ogole zgodny ze zdjeciami:

```
ref-01 0,878   ref-02 0,913   ref-03 0,870
ref-04 0,872   ref-05 0,890   ref-06 0,861
```

Piec z szesciu ujec mialo sufit PONIZEJ progu. Te piec nie moglo przejsc
nigdy, takze przy rozwiazaniu doskonalym.

Rozumowanie, ktore do tego doprowadzilo, bylo w komentarzu i wygladalo
poprawnie: "progi leza teraz MIEDZY tym, co mamy (0,661-0,879), a tym, co
osiagalne (0,861-0,913)". Blad polega na porownaniu progu z ZAKRESEM
sufitow zamiast z sufitem KAZDEGO przypadku osobno. 0,90 faktycznie lezy
wewnatrz przedzialu 0,861-0,913, ale wyzej niz piec z szesciu jego elementow.

## Lekcja

Bramka, ktora nie moze zapalic sie na zielono, niesie doklad0nie tyle samo
informacji co taka, ktora nie moze zapalic sie na czerwono. Obie sa stale.
Sprawdzamy zwykle tylko jedna strone (czy test umie oblac) i uznajemy to za
komplet - a druga strona kosztuje wiecej, bo do braku informacji dokłada
falszywy sygnal "jest coraz gorzej".

Koszt nie jest techniczny, tylko ludzki: kilka godzin pracy i porzucenie
dobrego kierunku.

## Reguly

1. Jesli dla zadania da sie zmierzyc SUFIT (najlepszy mozliwy wynik przy
   danych, jakie sa), prog musi byc porownywany z sufitem KAZDEGO przypadku
   z osobna, nie ze zbiorczym zakresem.
2. Samotest bramki powinien sprawdzac OBA konce naraz:
   - czy progi sa osiagalne (rozwiazanie idealne przechodzi),
   - czy nie sa za miekkie (wyrazna wada oblewa).
   Jeden bez drugiego nie wystarcza.
3. Gdy sufit nie jest znany, prog jest hipoteza i trzeba go tak opisac.
   "Prog wziety z sufitu" (w sensie: wymyslony) jest najgorszym rodzajem
   liczby w bramce, bo wyglada na pomiar.

## Kod, ktory to realizuje

```python
def progi_ujecia(nazwa):
    suf = SUFIT.get(nazwa)
    if suf is None:                       # przypadek nieprzemierzony,
        return PROG_BEZWZGLEDNY           # a nie cichy przepust
    return min(PROG_BEZWZGLEDNY, suf - LUZ)

# samotest:
#   F. dla kazdego przypadku prog <= sufit          (osiagalne)
#   G. wynik gorszy od sufitu o LUZ*4 musi oblac    (nie za miekkie)
```

## Gdzie to sie stalo

Kerf - Sawmill Tycoon, `_BlenderScripts/kerf_postac/kerf_zgodnosc.py`,
2026-07-28. Po poprawce bramka rozroznia: przod i tyl zaliczaja (sa przy
suficie), bok i ujecia skosne oblewaja - i to jest prawdziwa diagnoza
(model ma zla GLEBOKOSC, a nie zla szerokosc), ktorej stala czerwien
wczesniej nie pokazywala.
