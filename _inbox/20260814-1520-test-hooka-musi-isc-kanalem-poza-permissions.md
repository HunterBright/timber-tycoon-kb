---
title: Test hooka PreToolUse musi iść kanałem, którego permissions.deny NIE pokrywa
type: lesson
status: draft
confidence: high
verified: '2026-08-14'
date: 2026-08-14
project: Another Quest
tags:
- claude-code
- hooks
- permissions
- testowanie
- infra
applies_to: []
source: 'sesja testów po pierwszym starcie z D:\Unity\AnotherQuest, _Handoff/WYNIKI_TESTOW_2026-08-14.md'
severity: high
suggested-category: workflow/lessons
time_lost: ''
---

# Test hooka PreToolUse musi iść kanałem, którego `permissions.deny` NIE pokrywa

## Problem

Warstwa hooków `PreToolUse` w Claude Code była **martwa przez wiele dni** i nikt tego nie zauważył,
mimo że „testy" wypadały pozytywnie. Powód: testowano ją próbą `Edit` na chronionym pliku — a tę samą
ścieżkę pokrywała **także** reguła `permissions.deny`. Próba odbijała się od warstwy uprawnień,
komunikat wyglądał na sukces, a hook w tym czasie kończył się cicho kodem 0 („przepuść").

Dwa mechanizmy chroniące tę samą ścieżkę = **test nie mierzy tego, co myślisz, że mierzy**.
Przy martwym hooku wynik jest bit w bit identyczny jak przy działającym.

## Root cause

`permissions.deny` i hooki to dwie niezależne warstwy o **częściowo pokrywających się zakresach**.
Gdy obie pokrywają cel testu, ta wyżej maskuje ciszę tej niżej. Testowanie warstwy przez cel objęty
redundancją to metodologiczny odpowiednik sprawdzania hamulca ręcznego na płaskim parkingu.

Do tego dochodzi druga pułapka: hook zaprojektowany jako **fail-open** (żeby awaria transportu nie
kasowała pracy) jest z zewnątrz **nieodróżnialny od hooka nieistniejącego**. Milczenie jest jego
poprawną odpowiedzią w większości przypadków.

## Solution

**Wybierz cel testu leżący w różnicy zbiorów, nie w części wspólnej.**

1. Wypisz, co pokrywa `permissions.deny` (reguły ścieżkowe: `Edit(...)`, `Bash(...)`).
2. Wypisz, co pokrywa matcher hooka.
3. Testuj **wyłącznie** na celu z zakresu hooka minus zakres permissions.

W Claude Code najczystszym takim celem są **narzędzia MCP**: reguły ścieżkowe `Edit(ścieżka)`
**nie dotyczą narzędzi MCP**, więc jeśli wywołanie `mcp__serwer__zapisz` na chronionym rozszerzeniu
się odbije — odbicie mogło pochodzić wyłącznie z hooka. To test o zerowej dwuznaczności.

**Rozpoznawaj źródło odmowy po treści komunikatu.** Warto, żeby hook przedstawiał się prefiksem
(np. `BLOKADA (hook <nazwa>):`). Bez tego atrybucja jest zgadywaniem. `permissions.deny` w Claude Code
odpowiada zawsze tym samym zdaniem: `File is in a directory that is denied by your permission settings.`

**Zawsze dokładaj kontrolę negatywną.** Blokada, która blokuje wszystko, jest awarią, nie sukcesem —
i wygląda w raporcie tak samo dobrze jak blokada poprawna.

**Testuj realnym wywołaniem, nie `dry_run`.** Ale wybierz cel tak, żeby porażka testu (= hook martwy)
była nieszkodliwa: nazwa pliku jawnie testowa, w miejscu, które łatwo znaleźć i skasować. Przy martwym
hooku ten zapis naprawdę się wykona.

## What didn't work

- **Test przez `Edit` na chronionym rozszerzeniu** — przykryty przez `permissions.deny`, zero mocy dowodowej.
- **Test hooka `UserPromptSubmit` przez prompt bez słowa-klucza** — hook milczy poprawnie, wynik zerowy.
  Hook fail-open da się sprawdzić **wyłącznie** promptem, który MUSI go odpalić.
- **Uznanie „odbiło się" za dowód** bez sprawdzenia, **co** odbiło. Fakt odmowy jest tańszy niż jej atrybucja
  i dlatego kusi — ale to atrybucja niesie informację.

## Transferability

Dotyczy każdego projektu z Claude Code i każdej warstwy obrony w głąb (nie tylko hooków):
firewall + iptables, walidacja klienta + serwera, CI gate + pre-commit hook. Zasada jest jedna:
**warstwę testuje się na celu, którego nie chroni żadna inna warstwa.** Redundancja ochrony jest
dobra w produkcji i trująca w teście.

Drugi transferowalny wniosek: **fail-open i „nieobecny" są nieodróżnialne z zewnątrz**. Każdy komponent
fail-open potrzebuje osobnego, jawnego testu pozytywnego — inaczej jego awaria jest cicha z definicji.

## Related

- [[powershell-stdin-cp852-vs-utf8]] — druga przyczyna, dla której ten sam hook był martwy
