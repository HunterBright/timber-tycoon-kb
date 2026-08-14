---
title: Unity wypala organizationId (nick dewelopera) do globalgamemanagers kazdego builda
type: lesson
status: draft
confidence: high
verified: ''
date: 2026-08-13
project: Kerf - Sawmill Tycoon
tags:
- unity
- privacy
- build
- steam
- release
applies_to: []
source: audyt przedwydaniowy 2026-08-13
severity: high
suggested-category: engine/lessons
time_lost: ''
---

# Unity wypala organizationId (nick dewelopera) do globalgamemanagers kazdego builda

## Problem
Skan binarny builda przed wydaniem na Steam znalazl nick dewelopera ("mrhunterbright",
rowny prefiksowi prywatnego maila) w pliku `Kerf_Data/globalgamemanagers` - w KAZDEJ kopii
gry, widoczny kazdym narzedziem typu strings/hex.

## Root cause
Unity serializuje do `globalgamemanagers` pola z ProjectSettings zwiazane z uslugami chmury:
`organizationId`, `cloudProjectId`, `projectName`. `organizationId` to zwykle nick konta
Unity ID dewelopera. Trafia do builda NIEZALEZNIE od tego, czy uslugi chmury sa wlaczone
(u nas wszystkie mialy m_Enabled: 0 i pole i tak wychodzilo).

## Solution
1. Jesli projekt nie uzywa uslug chmury Unity: wyczyscic w `ProjectSettings/ProjectSettings.asset`
   pola `organizationId`, `cloudProjectId`, `projectName` (edycja tekstowa YAML dziala) i przebudowac.
2. Po rebuildzie POWTORZYC skan binarny `globalgamemanagers` - i wpisac go na stale do skryptu
   stagingu wydania (patrz wpis o skanie ASCII+UTF-16).
3. Pamietac o WSZYSTKICH platformach: build macOS niesie ten sam plik
   (`Kerf.app/Contents/Resources/Data/globalgamemanagers`).

## What didn't work
n/d - znalezione skanem przed wydaniem, nie po.

## Transferability
Dotyczy KAZDEGO projektu Unity budowanego z konta osobistego: nick konta Unity ID laduje
w plikach gry. Sprawdzac przed pierwszym publicznym buildem kazdego projektu.

## Related
- [[20260813-1040-skan-prywatnych-stringow-stagingu]]
- [[20260813-1050-dev-toggle-w-so-zapieczony-w-buildzie]]
