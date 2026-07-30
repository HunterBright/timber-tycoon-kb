---
title: Dowod wizualny z Unity batchmode (bez otwierania Edytora)
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-07-18'
project: Kerf - Sawmill Tycoon
tags:
- unity
- batchmode
- screenshot
- verification
- headless
- editor-scripts
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Dowod wizualny z Unity batchmode (bez otwierania Edytora)

## Problem
Zmiany wizualne w scenie (VFX, materialy, pozycje) robione skryptami edytorowymi w batchmode -
jak ZOBACZYC wynik bez recznego otwierania Edytora i bez czekania na playtest?

## Wzorzec
Metoda edytorowa odpalana po setupie sceny (w tym samym -executeMethod):
1. `ps.Simulate(5f, true, true)` na kazdym ParticleSystem (inaczej zrzut jest pusty - czasteczki startuja od zera),
2. tymczasowa kamera (`new GameObject` + Camera), pozycje liczone z bounds celu (4 strony swiata + zblizenia),
3. `cam.targetTexture = RenderTexture` -> `cam.Render()` -> `ReadPixels` -> `EncodeToPNG` -> plik w _Handoff/,
4. sprzatanie w `finally` (DestroyImmediate kamery/RT/tekstury), zrzuty PO zapisie sceny (nic tymczasowego nie wpada do pliku sceny),
5. do logu wypisac `particleCount` / inne liczniki - odroznia "nie ma efektu" od "efekt jest, ale niewidoczny".

Dziala z URP w -batchmode (bez -nographics!). Model czyta PNG i sam ocenia wynik przed oddaniem
czlowiekowi; czlowiek dostaje gotowe zrzuty zamiast "wierz mi, dziala".

## Uwagi
- Unity.exe na Windows wraca NATYCHMIAST - uruchamiac przez Start-Process -Wait i sprawdzac exit code.
- Kazdy przebieg to pelny start Unity (~90 s) - laczyc kroki fazy w jeden -executeMethod.
