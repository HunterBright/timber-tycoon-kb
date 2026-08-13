---
title: Wycinanie elementu z rastra logo — po barwie, nie po przerwie między kształtami
type: lesson
status: draft
confidence: high
verified: ''
date: 2026-08-13
project: Discord_Studio (MGDB Studio)
tags:
- branding
- raster
- pipeline-assetow
- python-pil
applies_to: []
source: ''
severity: low
suggested-category: workflow/lessons
time_lost: '~20 min'
---

# Wycinanie elementu z rastra logo — po barwie, nie po przerwie między kształtami

## Problem
Z logotypu `MGDB STUDIO` (raster z kanałem alfa, bez wektora i warstw) trzeba było wyciąć
samą literę B, żeby użyć jej jako wielkiego elementu graficznego w tle. Automat szukał
czystej kolumny tła między literami i ciął w ostatniej znalezionej. Wyciął `DB` — w kadrze
wylądowała ogromna litera D, czyli dokładnie nie ta litera, o którą chodziło.

## Root cause
Litera B ma wokół siebie poświatę o zasięgu kilkudziesięciu pikseli. Poświata **wypełnia
odstęp między D a B**, więc kolumna „czystego tła" w tym miejscu nie istnieje. Ostatnia
przerwa spełniająca próg leży dopiero przed literą D. Im ładniejsze światło wokół elementu,
tym pewniej ta metoda trafia obok.

## Solution
Wycinać po cesze, która odróżnia element od sąsiadów — tu po barwie, bo B jest jedyną
ciepło zabarwioną literą:

1. maska `(R − B_kanał > 18) & (alfa > 0.3)`
2. rozmycie maski gaussem (σ ≈ 18 px przy szerokości logotypu 2679 px) i wzmocnienie
   `clip(maska × 6, 0, 1)` — miękki brzeg i objęcie poświaty, która też jest ciepła
3. `alfa_wyniku = alfa_źródła × maska`, dopiero potem bounding box

Efekt: żadnego skrawka sąsiedniej litery, poświata zachowana, brzeg bez schodków.

## What didn't work
- **Przerwa w profilu kolumnowym alfy** — znajduje przerwę przed D, opisane wyżej.
- **Przycięcie prostokątem z zapasem w lewo** (`x0 − 22% szerokości`) — zostawiało w kadrze
  wąski sierp prawej krawędzi litery D. Przy jasnym elemencie na ciemnym tle taki skrawek
  jest doskonale widoczny, nawet przy kryciu 20%.

## Transferability
Dotyczy każdego rastrowego assetu marki, z którego trzeba wyjąć fragment: logotypy z glow,
ikony z cieniem, sprite'y z bloomem w atlasie. Reguła ogólna: **separator geometryczny
zawodzi wszędzie tam, gdzie efekt świetlny wykracza poza sylwetkę** — trzeba wtedy sięgnąć
po cechę, która nie rozlewa się razem ze światłem (barwa, kanał, warstwa źródłowa).
Najtańsza profilaktyka: zanim zaufasz automatowi, obejrzyj wycinek — 10 sekund.

## Related
- [[podglad-w-docelowym-rozmiarze-zamiast-oceny-pelnego-kadru]]
