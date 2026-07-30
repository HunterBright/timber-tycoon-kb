---
title: Świeżość builda Unity sprawdzaj po DLL z kodem gry, nie po .exe
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-19'
project: Kerf - Sawmill Tycoon
tags:
- unity
- build
- batchmode
- verification
- windows
- powershell
applies_to: []
source: ''
suggested-category: engine/lessons
---

# Świeżość builda Unity sprawdzaj po DLL z kodem gry, nie po .exe

## Problem
Automatyczna bramka "czy build się naprawdę wykonał" porównywała znacznik czasu
`Timber_Tycoon.exe` z godziną startu builda. Build zakończył się sukcesem
(raport: Succeeded, 0 błędów), a check i tak krzyczał BUILD_STALE - bo plik .exe
miał datę sprzed dwóch dni.

## Przyczyna
W buildzie Windows .exe to tylko uniwersalny "starter" playera - Unity nie
przepisuje go, jeśli się nie zmienił. Kod gry ląduje w
`<Build>_Data/Managed/<AsmDef>.Runtime.dll` (albo w GameAssembly.dll przy IL2CPP)
i to TEN plik dostaje świeży znacznik czasu przy każdym buildzie.

## Rozwiązanie
Test świeżości builda: `LastWriteTime` biblioteki z kodem gry
(np. `Builds/Win64/Timber_Tycoon_Data/Managed/TimberTycoon.Runtime.dll`) > czas
startu builda. Dodatkowo czytaj raport własnego build-skryptu (result/bledy).

## Bonus - czekanie na batchmode
`& Unity.exe -batchmode ...` w PowerShell wraca NATYCHMIAST (aplikacja GUI).
Zwalidowany wzorzec: `Start-Process -FilePath Unity.exe -ArgumentList ... -PassThru -Wait`
czeka poprawnie do końca builda i daje `ExitCode`.

## Transferowalność
Dowolny projekt Unity z bramką build-verification na Windows.
