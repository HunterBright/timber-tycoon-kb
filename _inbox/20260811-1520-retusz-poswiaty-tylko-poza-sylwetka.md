---
title: Retusz poświaty wokół glifu — przemalowuj wyłącznie tło poza sylwetką
type: lesson
status: draft
confidence: high
verified: ''
date: 2026-08-11
project: Discord_Studio (branding MGDB Studio)
tags:
- branding
- obrobka-obrazu
- poswiata
- glow
- numpy
- pillow
applies_to: []
source: ''
severity: medium
suggested-category: workflow/lessons
time_lost: '~40 min'
---

# Retusz poświaty wokół glifu — przemalowuj wyłącznie tło poza sylwetką

## Problem

Logotyp wygenerowany przez model obrazowy (Qwen-Image) miał poświatę narysowaną
niesymetrycznie wokół litery: po prawej i u góry sięgała 55–68 px, po lewej i u dołu
tylko 18–31 px. Przy płaskich zakończeniach lewej nogi litery światło urywało się
kanciasto i z daleka czytało się jako prostokątna plama, a nie jako świecąca litera.

## Root cause

Model rysuje poświatę jako część obrazu, nie jako osobną warstwę. Nie ma czego
„przestawić" — trzeba ją policzyć od nowa z sylwetki litery i wstawić w miejsce starej.

## Solution

1. Maska litery z progu jasności **i** ciepła barwy (`lum > 105` oraz `R − B > 30`),
   `binary_opening` na śmieci, największa spójna plama.
2. Poświata liczona z **wypełnionej** sylwetki (`binary_fill_holes`) jako suma trzech
   rozmyć gaussowskich (σ 5/14/34 przy obrazie 1280×720) — jedno rozmycie daje albo
   twardą obwódkę, albo mgłę bez wyrazu.
3. Siła kalibrowana na **tej krawędzi, która w oryginale wyglądała dobrze** (tu: prawy
   bok), a nie na średniej z całego obwodu — średnia jest ściągana w dół przez zepsutą stronę.
4. Podmiana wyłącznie w obszarze `binary_erosion(~sylwetka)`: 2 px przy literze zostaje
   z oryginału, więc krawędź glifu i jej antyaliasing są nietknięte.
5. Tło odtworzone ze średniej i odchylenia próbki ciemnych pikseli — samo płaskie tło
   bez ziarna byłoby widoczne jako gładka łata na ziarnistym tle.
6. Wejście w obszar sąsiedniej litery przez poziomą rampę alfa (15 px), żeby nie
   zostawić pionowego szwu.

## What didn't work

**Przemalowanie całego prostokąta wokół litery.** Pierwsza wersja podmieniała każdy piksel
na prawo od sąsiedniej litery, czyli także **oczka glifu** (wnętrza brzuszków B). Miękkie
gradienty w oczkach zostały zastąpione płaskim tłem z nową poświatą, a granica maski dała
schodkowaną, poszarpaną krawędź — efekt gorszy niż wada, którą naprawiano.

Zasada: maska litery służy do **liczenia** światła z całej bryły, ale do **podmiany pikseli**
używaj tylko obszaru na zewnątrz sylwetki. Wnętrze glifu nie jest tłem.

## Transferability

Dotyczy każdego retuszu grafiki wygenerowanej przez model, gdzie efekt świetlny jest wtopiony
w piksele: logotypy, ikony, splash screeny, tekstury emisyjne do gry. Ta sama metoda działa
na mapach emisji w Unity — poświatę wokół kształtu lepiej policzyć z maski niż domalować.

## Related
- [[podpikselowe-skalowanie-logotypu-polem-odleglosci]]
