---
title: Petla "uruchom gre - zobacz wynik" w Unity przez most MCP
type: pattern
status: draft
confidence: medium
verified: 2026-08-14
tags: [unity, mcp, automatyzacja, petla-testera, claude-code, tryb-gry]
date: 2026-08-14
project: GameDevOS
suggested-category: engine/patterns
source: pomiar na zywym edytorze Unity 6000.5.1f1 (Timber_Tycoon), 14.08.2026
applies_to: [unity, claude-code, unity-editor-mcp]
---

# Petla "uruchom gre - zobacz wynik" w Unity przez most MCP

## Kiedy stosować
Gdy agent ma sam sprawdzić skutek swojej zmiany w grze, zamiast pytać człowieka
"uruchom i powiedz, co widzisz". Dotyczy każdego projektu Unity z podpiętym
mostem MCP do edytora.

## Kroki
1. Sprawdź stan edytora. Nie wchodź w tryb gry, gdy trwa kompilacja albo
   przeładowanie kodu - poczekaj i sprawdź ponownie.
2. Sprawdź, czy otwarta scena ma ścieżkę. **Pusta scena bez nazwy to pułapka:**
   systemy startujące automatem dosypią się same, nie znajdą swoich zależności
   i zaleją konsolę błędami, które nic nie znaczą.
3. Zapamiętaj kursor konsoli PRZED uruchomieniem. Dalej czytaj tylko wpisy
   nowsze niż ten kursor.
4. Wejdź w tryb gry. **Kolejne wywołanie zwróci błąd połączenia** - to normalne,
   Unity przeładowuje kod i most na kilka sekund pada. Ponów, nie panikuj.
5. Daj grze pożyć tyle, ile trzeba, żeby zobaczyć badane zjawisko.
6. Zrób zrzut z kamery gry **bez zapisu do pliku** - obraz ma trafić do rozmowy,
   nie do repozytorium.
7. Przeczytaj błędy od kursora i **zgrupuj je po treści**.
8. Zatrzymaj tryb gry zawsze, także po niepowodzeniu wcześniejszego kroku.

## Dlaczego to działa
Każdy krok zamyka jedną drogę do fałszywego wniosku: kompilacja - agent gra
w starym kodzie; pusta scena - agent widzi awarię, której nie ma; brak kursora -
agent melduje błędy sprzed tygodnia; brak grupowania - agent melduje 300
problemów zamiast trzech; brak zatrzymania - człowiek traci swoje zmiany przy
wyjściu z trybu gry.

## Koszty i kompromisy
Petla wymaga otwartego edytora z wczytaną właściwą sceną, więc nie zastąpi
uruchomienia wsadowego (bez okienek) w automacie nocnym. Agent celowo nie
otwiera sceny sam - to oszczędza jedno kliknięcie kosztem grzebania w widoku
człowieka, a ta zamiana się nie opłaca.

## Warianty
Gdy chodzi o wygląd, a nie o błędy, potrzebny jest zrzut sprzed zmiany jako
punkt odniesienia - inaczej agent ocenia obraz bez skali. Gdy chodzi o regresję,
zamiast zrzutu lepszy jest zestaw testów w trybie gry.

## Dowód, że zadziałało u nas
14.08.2026, Timber_Tycoon: pełny ciąg przejechany bez udziału człowieka. Tryb
gry wszedł, zrzut z kamery przyszedł do rozmowy, konsola dała 3 różne błędy
z ponad 300 wpisów w 35 sekund, edytor wrócił do stanu zatrzymanego.
Spakowane w komendę `/zagraj` (`%USERPROFILE%\.claude\commands\zagraj.md`).

## Powiązane
- [[]]
