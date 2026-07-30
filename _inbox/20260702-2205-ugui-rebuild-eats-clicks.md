---
title: Rebuild-on-event w uGUI zjada kliki + ScrollRect bez Graphica nie scrolluje
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-02'
project: Timber_Tycoon
tags:
- unity
- ugui
- ui
- rebuild
- events
- scrollrect
applies_to: []
source: ''
severity: minor
suggested-category: engine/lessons
---

# Rebuild-on-event w uGUI zjada kliki + ScrollRect bez Graphica nie scrolluje

## Problem 1: destroy-and-rebuild niszczy przycisk między pointer-down a pointer-up
Wzorzec „event ustawia dirty → Update przebudowuje całą listę (Destroy + Create)" jest wygodny, ale gdy eventy lecą Z TŁA (u nas: NPC-produkcja zmienia magazyn/kasę co parę sekund, a okno subskrybuje OnStorageChanged/OnMoneyChanged), przebudowa potrafi trafić między wciśnięcie a puszczenie LMB — `Button.onClick` odpala się na pointer-UP na TYM SAMYM obiekcie, więc klik przepada bez śladu.

**Fix (tani):** w Update odraczaj przebudowę, dopóki gracz trzyma LMB, + minimalny odstęp koalescencji:
`if (dirty && !Input.GetMouseButton(0) && Time.unscaledTime - lastRebuild >= 0.25f) { dirty=false; Rebuild(); }`

## Problem 2: ScrollRect wymaga raycastowalnego Graphica pod kursorem
ScrollRect NIE łapie kółka myszy/dragu, jeśli w jego poddrzewie pod kursorem nie ma żadnego Graphica z `raycastTarget=true` (typowe, gdy wiersze listy mają `raycastTarget=false` dla wydajności, a viewport ma tylko RectMask2D). **Fix:** niewidoczny `Image` na viewporcie (np. `color = (0,0,0,0.01f)`).

## Transfer
Oba dotyczą każdego uGUI budowanego w kodzie z live-refresh. Checklist do nowych okien: (1) czy eventy tła mogą przebudować UI w trakcie interakcji? (2) czy każdy ScrollRect ma raycast-target pod kursorem?
