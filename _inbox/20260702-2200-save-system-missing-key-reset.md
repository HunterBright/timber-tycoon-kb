---
title: Nowy ISaveable + stary save = przeciek żywego stanu (reset przy braku klucza)
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-02'
project: Timber_Tycoon
tags:
- unity
- save-system
- isaveable
- backward-compat
- load
applies_to: []
source: ''
severity: major
suggested-category: engine/lessons
---

# Nowy ISaveable + stary save = przeciek żywego stanu (reset przy braku klucza)

## Problem
Keyed save system (słownik klucz→JSON) woła `LoadSaveData` tylko dla kluczy OBECNYCH w pliku. Gdy dodajesz NOWY system ISaveable (np. kadry NPC) i gracz wczyta STARY save (sprzed feature'a), nowy system nie dostaje żadnego wywołania — jego ŻYWY stan bieżącej sesji przecieka do wczytanej gry. W Timber Tycoon: zatrudniona w sesji załoga przetrwała wczytanie starego save'a z cofniętą kasą (darmowa załoga), a `lastPaidDay` z przyszłości tłumił payroll przez wiele dni.

## Root cause
Load-in-place (bez przeładowania sceny) + brak ścieżki „reset do defaults" dla saveable'a, którego klucza nie ma w pliku. Warning w logu ≠ reset.

## Fix / wzorzec
Save manager emituje sygnał po ZAKOŃCZONYM load (np. `public static event Action OnAfterLoad` obok istniejącego `OnBeforeSave`) + trzyma cache wczytanych kluczy (`TryGetLoadedData(key, out json)`). Każdy nowy system subskrybuje i przy braku własnego klucza robi PEŁNY reset do stanu świeżej gry + emituje swoje eventy zmiany (żeby zależne systemy — u nas spawnery agentów — posprzątały świat).

## Transfer
Dotyczy każdego projektu Unity z keyed save + load-in-place. Reguła: **dodajesz ISaveable po premierze pierwszych save'ów → OBOWIĄZKOWO ścieżka resetu przy braku klucza.** Test: hire/zmień stan → wczytaj save sprzed feature'a → stan musi wrócić do defaults.
