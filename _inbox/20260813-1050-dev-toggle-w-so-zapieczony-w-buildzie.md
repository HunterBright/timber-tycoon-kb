---
title: Dev-toggle w ScriptableObject zapieczony w buildzie wydaniowym (klasa bledu + obrona)
type: lesson
status: draft
confidence: high
verified: ''
date: 2026-08-13
project: Kerf - Sawmill Tycoon
tags:
- unity
- scriptableobject
- release
- build-gate
- probe
applies_to: []
source: audyt przedwydaniowy 2026-08-13
severity: critical
suggested-category: engine/lessons
time_lost: ''
---

# Dev-toggle w ScriptableObject zapieczony w buildzie wydaniowym

## Problem
Flaga `spawnOnlyOnDemand` (tryb "klienci tylko na klawisz F10", wlaczany do sesji
zdjeciowych) zostala przestawiona w assecie 8.08 i ZOSTALA tam. Build z 10.08 - kandydat
do wydania na Steam - mial ja zapieczona: w takiej grze zaden klient nigdy nie przyjezdza
sam, czyli zero sprzedazy i zero progresji u kazdego gracza. Wykryl to dopiero audyt
przedwydaniowy; sonda buildowa przechodzila na zielono.

## Root cause
Klasa bledu: NARZEDZIE deweloperskie (fotosesja) wymaga stanu X w assecie, wydanie wymaga
stanu Y, a przelaczanie jest RECZNE. Kazde przelaczenie to przyszly incydent - asset nie
wraca sam. Gorzej: sonda fotosesji WYMAGALA X (flagi wlaczonej), wiec zadna istniejaca
bramka nie mogla pilnowac Y.

## Solution
Dwie zmiany naraz (obie konieczne):
1. Narzedzie dev przestawia flage W PAMIECI na czas swojego procesu (mutacja ScriptableObject
   w buildzie znika z koncem procesu; w Edytorze narzedzie startuje tylko z argumentow exe,
   wiec asset zostaje nietkniety). Asset trzyma NA STALE wartosc wydaniowa.
2. Bramka wydaniowa w sondzie buildu: check "flaga MUSI byc w stanie wydaniowym"
   + dzwignia (-releasegateredproof) udajaca zly stan, zeby udowodnic tryb porazki checku.

## What didn't work
Poleganie na pamieci czlowieka ("przestawie z powrotem po zdjeciach") - to bylo zrodlo
incydentu.

## Transferability
Kazdy projekt z narzedziami dev sterowanymi assetami/konfigami: fotosesje, cheaty, tryby
nagraniowe, debug spawny. Regula: dev-tryb wlacza sie RUNTIME przez narzedzie, nie edycja
assetu; kazda wartosc wydaniowa dostaje bramke w sondzie buildu z udowodnionym FAIL.

## Related
- [[20260813-1030-unity-cloud-id-wycieka-do-builda]]
