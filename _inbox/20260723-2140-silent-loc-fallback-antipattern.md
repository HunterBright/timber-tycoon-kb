---
type: anti-pattern
project: Kerf - Sawmill Tycoon
suggested-category: engine/lessons
tags: [unity, localization, i18n, fallback, testing, probe]
date: 2026-07-23
status: draft
---

# Cichy fallback lokalizacji ukrywa nieprzetłumaczoną treść

## Anti-pattern
Wzorzec `Loc.T(key, nativeFallback)` — gdy klucza nie ma w słowniku, funkcja po cichu
zwraca tekst w języku źródłowym wpisany w miejscu wywołania. Nigdy `[key]`, nigdy błąd,
nigdy warning w buildzie.

## Dlaczego to boli
Każdy NOWY system dodany po "wielkiej passie lokalizacyjnej" (u nas: meble, zlecenia,
załoga, dostawy, magazyn — 271 tekstów) działa wizualnie poprawnie po polsku we
WSZYSTKICH językach, więc nikt nie zauważa braku tłumaczeń. W Kerf uzbierało się
~32% pełnego słownika (271/846) niewidocznie — wykryte dopiero celowym audytem.
Walidator porównujący języki między sobą też tego NIE łapie, bo klucz brakuje
we wszystkich plikach naraz (parzystość = OK).

## Co działa (pattern naprawczy)
1. **Skan kluczy kod→słownik** jako bramka: wyciągnij wszystkie literalne klucze
   z wywołań Loc.T/SetLoc i sprawdź członkostwo w pliku wzorcowym. Uwaga na regex:
   wzorzec połykający "ogon" wywołania zjada sąsiednie wywołania w wyrażeniach
   warunkowych (`cond ? Loc.T(...) : Loc.T(...)`) — dopasowuj TYLKO do klucza,
   nic za nim.
2. **Klucze składane** (`"prefix_" + enum`) enumeruj osobno ze źródła danych
   (u nas: unlockId z assetów SO).
3. **Sonda w prawdziwym buildzie**: 15 słowników ładuje się z Resources? Zbiory
   kluczy identyczne? Próbka nowych kluczy niepusta w każdym języku? (Pakowanie
   Resources w buildzie to inna ścieżka niż Edytor.)
4. Przy dodawaniu nowego systemu: klucze do słowników dodawaj w TYM SAMYM commicie
   co wywołania Loc.T — inaczej fallback zamaskuje dług na miesiące.

## Powiązane
Tłumaczenia 14 języków robione fan-outem agentów per język z pakietem terminologii
zakotwiczonym w ISTNIEJĄCYM pliku danego języka (wzorce nazw klonowane znak w znak)
+ niezależny recenzent per język — recenzenci realnie łapali kalki i fałszywych
przyjaciół (de "Tier"=zwierzę, en "Commode", tr "Komodin").
