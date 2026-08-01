---
title: 'Wzorzec: pętla serii maszyny z minigrą (batch-machine loop)'
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-07-02'
project: Kerf - Sawmill Tycoon
tags:
- unity
- minigame
- loop
- batch
- count-picker
- camera-lock
- ux
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Wzorzec: pętla serii maszyny z minigrą (batch-machine loop)

## Problem
Maszyna z minigrą per partia: przerabianie N partii nie może wymagać N pełnych sesji
(wejście E → kamera → minigra → wyjście → E...).

## Rozwiązanie (zwalidowane 3x: Chipper, Pelletizer, FertilizerMaker)
1. **Picker liczby partii** przed sesją (wspólny `ChoppingSelectionUI`): max =
   dostępnySurowiec / zużycieNaPartię. Species-agnostic → jeden wpis z gatunkiem None.
2. **Kamera blokowana RAZ na całą serię** (zapis world pos/rot → lerp → pętla partii →
   restore raz na końcu).
3. **Pierwsza partia: zielony guzik = PRZYTRZYMANIE ~1 s** (rozruch maszyny, pasek postępu,
   „trzymaj albo trać" - puszczenie cofa o połowę tempa). **Kolejne partie: pojedynczy klik**
   (maszyna już rozgrzana). Flaga `drumRunning`/`machineRunning` przełącza tryb.
4. **Zużycie surowca per partia W ŚRODKU pętli** (na czerwonym guziku), nie z góry -
   break przy braku surowca/miejsca zostawia resztę kolejki w magazynie.
5. **Jedna ścieżka wyjścia** (`EndSessionCleanup`) dla: koniec serii, break, abort -
   zero rozjazdów stanu.

## Dlaczego tak
Parytet UX między maszynami = gracz uczy się raz. Kamera raz na serię usuwa najbardziej
męczący element (najazd/zjazd). Hold-pierwszy/klik-kolejne czyta się jako "rozruch maszyny".

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260629-1917-diegetic-buttons-frame-minigame|Console buttons FRAME a skill minigame (they're flow-control, not the mechanic)]] - wspolne: camera-lock, minigame
- [[20260713-1430-probe-visibility-by-rotating-rays-not-the-object|Sonda widoczności: obracaj PROMIENIE, nie obiekt]] - wspolne: ux, minigame
<!-- /POWIAZANE:auto -->
