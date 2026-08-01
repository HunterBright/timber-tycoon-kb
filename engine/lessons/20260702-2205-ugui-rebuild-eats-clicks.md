---
title: Rebuild-on-event w uGUI zjada kliki + ScrollRect bez Graphica nie scrolluje
type: lesson
status: active
confidence: medium
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
promoted: '2026-07-30'
---

# Rebuild-on-event w uGUI zjada kliki + ScrollRect bez Graphica nie scrolluje

## Problem 1: destroy-and-rebuild niszczy przycisk między pointer-down a pointer-up
Wzorzec „event ustawia dirty → Update przebudowuje całą listę (Destroy + Create)" jest wygodny, ale gdy eventy lecą Z TŁA (u nas: NPC-produkcja zmienia magazyn/kasę co parę sekund, a okno subskrybuje OnStorageChanged/OnMoneyChanged), przebudowa potrafi trafić między wciśnięcie a puszczenie LMB - `Button.onClick` odpala się na pointer-UP na TYM SAMYM obiekcie, więc klik przepada bez śladu.

**Fix (tani):** w Update odraczaj przebudowę, dopóki gracz trzyma LMB, + minimalny odstęp koalescencji:
`if (dirty && !Input.GetMouseButton(0) && Time.unscaledTime - lastRebuild >= 0.25f) { dirty=false; Rebuild(); }`

## Problem 2: ScrollRect wymaga raycastowalnego Graphica pod kursorem
ScrollRect NIE łapie kółka myszy/dragu, jeśli w jego poddrzewie pod kursorem nie ma żadnego Graphica z `raycastTarget=true` (typowe, gdy wiersze listy mają `raycastTarget=false` dla wydajności, a viewport ma tylko RectMask2D). **Fix:** niewidoczny `Image` na viewporcie (np. `color = (0,0,0,0.01f)`).

## Transfer
Oba dotyczą każdego uGUI budowanego w kodzie z live-refresh. Checklist do nowych okien: (1) czy eventy tła mogą przebudować UI w trakcie interakcji? (2) czy każdy ScrollRect ma raycast-target pod kursorem?

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260723-1215-ugui-mask-clear-image-invisible|uGUI: Mask z obrazkiem Color.clear = NIEWIDZIALNA zawartosc (a transformy klamia, ze wszystko gra)]] - wspolne: scrollrect, ugui
- [[20260613-0625-9slice-ppu-must-scale-to-target-rect-not-stay-100|A large 9-slice sprite at PixelsPerUnit=100 breaks because its fixed corners exceed the panel]] - wspolne: ugui, ui
- [[statistics-manager-pattern|StatisticsManager Pattern]] - wspolne: events, ui
- [[20260607-2016-ugui-filled-image-needs-sprite|A UGUI `Image` with `type = Filled` but no sprite ignores `fillAmount`]] - wspolne: ugui, ui
- [[20260613-0610-dim-scrim-must-not-reuse-9slice-panel-factory|Don't build a full-screen dim/scrim by reusing your skinnable panel factory]] - wspolne: ugui, ui
<!-- /POWIAZANE:auto -->
