---
title: Świeżość builda Unity sprawdzaj po DLL z kodem gry, nie po .exe
type: lesson
status: active
confidence: medium
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
promoted: '2026-07-30'
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

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260726-1415-powershell-nie-czeka-na-unity-batchmode|PowerShell nie czeka na Unity.exe ani na exe gry - kontrola swiezosci builda strzela za wczesnie]] - wspolne: batchmode, powershell, build
- [[20260714-2245-unity-batchmode-returns-before-build-finishes|Unity w trybie wsadowym WRACA, zanim build się skończy - i sonda daje fałszywe zielone światło]] - wspolne: batchmode, verification, build
- [[20260713-1830-runtime-meshcollider-needs-readwrite-and-editor-cannot-prove-it|Runtime MeshCollider wymaga Read/Write Enabled - a Edytor NIE JEST w stanie tego udowodnić]] - wspolne: verification, build
- [[20260718-0805-headless-visual-proof-batchmode|Dowod wizualny z Unity batchmode (bez otwierania Edytora)]] - wspolne: batchmode, verification
- [[20260713-1900-build-early-never-built-project-hides-editor-only-bugs|Projekt, który nigdy nie był budowany, hoduje całą klasę uśpionych błędów]] - wspolne: verification, build
<!-- /POWIAZANE:auto -->
