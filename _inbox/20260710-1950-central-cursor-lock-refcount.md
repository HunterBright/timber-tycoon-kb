---
type: pattern
project: Timber Tycoon
suggested-category: engine/patterns
tags: [unity, cursor, ui, input, esc, refcount]
date: 2026-07-10
status: draft
---

# Centralny zarządca kursora z refcountem właścicieli (zamiast rozproszonych Cursor.lockState)

## Problem
W grze FPP każde okno/minigra odblokowuje kursor (`Cursor.lockState = None`) przy otwarciu
i blokuje przy zamknięciu. Gdy robi to 20 klas niezależnie:
1. **Last-writer-wins**: zamknięcie okna A blokuje kursor, mimo że okno B wciąż otwarte.
2. **Podwójna obsługa ESC**: dwa Update'y łapią ten sam ESC w jednej klatce (kolejność
   niedeterministyczna) - jeden ESC zamyka dialog I otwiera pauzę.
3. **Quirk edytora/Windows**: re-lock ustawiony w TEJ SAMEJ klatce co ESC bywa ignorowany -
   kursor zostaje widoczny do następnego kliknięcia w ekran.

## Wzorzec
Statyczna klasa `GameCursor`:
- `Unlock(object owner)` - dodaje właściciela do HashSet, odblokowuje.
- `Release(object owner)` - zdejmuje właściciela; gdy zbiór pusty, blokuje kursor
  **w następnej klatce** (ukryty runner MonoBehaviour, DontDestroyOnLoad) - to omija quirk (3).
- `ApplyDefaultLock()` - natychmiastowa blokada tylko gdy zbiór pusty (start gry, wsiadanie do auta).
- Prune "fake-null": właściciele będący zniszczonymi komponentami Unity usuwani przy każdej operacji.
- Refcount naturalnie załatwia (1): dialog nad pauzą może się zamknąć, kursor zostaje, bo pauza
  wciąż trzyma. Ręczne "if (PauseMenu.IsOpen) return" znikają.
- ESC obsługiwać w JEDNYM miejscu (łańcuch priorytetów), nie per-okno - to załatwia (2).

## Kiedy stosować
Każda gra FPP/TPP z więcej niż 2-3 oknami UI, które ruszają kursorem. Migracja jest mechaniczna:
para `Cursor.lockState/visible` -> jedno wywołanie.
