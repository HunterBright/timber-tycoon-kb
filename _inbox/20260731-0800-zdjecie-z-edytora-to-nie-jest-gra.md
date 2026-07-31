---
title: Zrzut z Edytora pokazuje scenę zbudowaną, nie rozegraną
type: anti-pattern
project: Kerf - Sawmill Tycoon
suggested-category: engine/anti-patterns
tags:
- unity
- zrzuty-ekranu
- marketing
- edytor-vs-build
- dowody
date: '2026-07-31'
status: draft
confidence: high
verified: '2026-07-31'
source: budowa komend Unity CLI + sesja zdjęciowa do zwiastuna 2026-07-31
---

# Zrzut z Edytora pokazuje scenę zbudowaną, nie rozegraną

## Co się stało

Powstało narzędzie robiące zdjęcie sceny prosto z Edytora: tymczasowa kamera, render
do tekstury, zapis PNG. Szybkie (1,4 s) i wygodne, bo nie wymaga budowania gry.

Ten sam kadr, ta sama pozycja kamery, dwa źródła:

| Z Edytora | Z builda po wczytaniu zapisu |
|---|---|
| hala pełna hałd gruzu, pusta posadzka | maszyny, regały, towar na półkach, wózek |

Obraz z Edytora nie był popsuty. Był **prawdziwy** - tak wygląda scena, zanim gra się
uruchomi. Maszyny, regały i zawartość magazynu powstają dopiero w czasie gry, z zapisu
albo ze skryptów startowych.

## Dlaczego to jest pułapka, a nie oczywistość

Bo w większości kadrów różnicy nie widać. Krajobraz, budynki, teren, drogi, roślinność -
wszystko to leży w scenie i wygląda identycznie w obu źródłach. Różnica dotyczy wyłącznie
tego, co system stawia w czasie gry, a więc dokładnie tych rzeczy, które w grze
ekonomicznej są najważniejsze: maszyn, magazynu, towaru, postaci.

Zrzut z Edytora daje więc obraz, który jest **jednocześnie ładny i pusty**, i łatwo go
uznać za dobry, dopóki nie postawi się obok kadru z gry.

## Reguła

**Zrzut z Edytora nadaje się do dowodu, jak wygląda MODEL. Nie nadaje się do dowodu,
jak wygląda GRA.**

| Do czego | Czym |
|---|---|
| przód, bok, tył modelu z tej samej kamery i światła | zrzut z Edytora (szybki) |
| kontrola, czy asset trafił na scenę i ma materiał | zrzut z Edytora |
| zwiastun, sklep, materiały prasowe | tryb zdjęciowy w buildzie |
| dowód, że mechanika wygląda jak trzeba | tryb zdjęciowy w buildzie |

## Warunek konieczny dla trybu zdjęciowego w buildzie

Kadry celujące we wnętrza trzeba komponować **na rozegranym zapisie**. Zapis świeżej gry
daje puste pomieszczenia i kadry są bezużyteczne mimo poprawnych współrzędnych kamery.
Utrata jednego rozegranego zapisu unieważnia całą listę ujęć wnętrz - warto trzymać
osobny zapis „do zdjęć" i traktować go jak asset, a nie jak plik tymczasowy.

## Powiązane

- [[build-is-the-only-truth-editor-lies]]
- [[unity-photoshoot-mode-cmdline]]
