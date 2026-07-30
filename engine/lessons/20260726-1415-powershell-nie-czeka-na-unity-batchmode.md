---
title: PowerShell nie czeka na Unity.exe ani na exe gry - kontrola swiezosci builda strzela za wczesnie
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-26'
project: Kerf - Sawmill Tycoon
tags:
- unity
- build
- batchmode
- powershell
- ci
- automation
- false-green
applies_to:
- unity
- windows
- powershell
source: ''
severity: high
time_lost: '~10 min (ale klasa bledu: falszywy zielony build)'
promoted: '2026-07-30'
---

# PowerShell nie czeka na Unity.exe ani na exe gry - kontrola swiezosci builda strzela za wczesnie

## Problem

Skrypt budujacy w PowerShellu:

```powershell
& $unity -batchmode -quit -projectPath $root -executeMethod Build.BuildWindows -logFile $log
$code = $LASTEXITCODE
# ...natychmiast po tym: sprawdzenie swiezosci
"Runtime.dll w buildzie: " + (Get-Item $dllWBuildzie).LastWriteTime
```

Wynik przebiegu:

```
START BUILD 14:04:04
KONIEC BUILD 14:04:04  exitcode=          <-- pusty kod wyjscia, 0 sekund
Runtime.dll w buildzie: 07/25/2026 22:16  <-- WCZORAJSZY kod
```

Wyglada jak "build nie wystartowal". W rzeczywistosci build ruszyl i po 26 s
zakonczyl sie sukcesem (`Build Finished, Result: Success` w logu Unity).
Skrypt zwyczajnie **nie poczekal** i zmierzyl plik, ktorego Unity jeszcze nie
zdazylo nadpisac.

Gdyby kontrola swiezosci byla odwrotnie sformulowana (np. "czy plik istnieje"),
skrypt zameldowalby ZIELONE na wczorajszym kodzie.

## Root cause

`Unity.exe` oraz zbudowany exe gry sa aplikacjami podsystemu **GUI** (Windows
subsystem: WINDOWS, nie CONSOLE). Operator `&` w PowerShellu (i `Start-Process`
bez `-Wait`) czeka wylacznie na programy konsolowe. Dla programu okienkowego
wraca natychmiast po uruchomieniu procesu, nie po jego zakonczeniu. `-batchmode`
tego nie zmienia - to flaga zachowania Unity, nie typ podsystemu binarki.
Dlatego `$LASTEXITCODE` zostaje **niezainicjowany** (nie 0, tylko pusty) - to
najlepszy sygnal, ze polecenie w ogole nie bylo czekane.

## Solution

Czekac jawnie i na procesie, nie na operatorze:

```powershell
$p = Start-Process -FilePath $unity -ArgumentList $args -PassThru
Wait-Process -Id $p.Id -Timeout 1800    # limit, zeby zawis nie wisial wiecznie
$code = $p.ExitCode
```

Dwie zasady, ktore ratuja niezaleznie od powyzszego:

1. **Swiezosc builda mierz po pliku z KODEM, nie po .exe.** W buildzie Unity
   `.exe` jest launcherem i czesto NIE zmienia daty miedzy buildami (tu:
   `Kerf.exe` z 24.07, a swiezo zbudowana `Kerf_Data\Managed\*.Runtime.dll`
   z 26.07 14:04:39). Porownuj date DLL-a z data zapisu pliku zrodlowego.
2. **Kod wyjscia sondy/gry jest niewiarygodny** - exe potrafi sypnac sie przy
   zamykaniu i zwrocic bzdure. Werdykt czytaj z PLIKU raportu, a raport
   **usun przed uruchomieniem**, zeby przerwany przebieg nie zostawil starego
   wyniku, ktory przeczytasz jako nowy.

## What didn't work

- `& $exe ...` - nie czeka (sedno problemu).
- Zaufanie `$LASTEXITCODE` - dla programu okienkowego zostaje pusty, a puste
  w porownaniach liczbowych latwo przechodzi jako "brak bledu".
- Zaufanie dacie `.exe` jako dowodowi, ze build sie przebudowal.

## Transferability

Dotyczy kazdego projektu Unity budowanego z wiersza polecen na Windowsie, a
szerzej: kazdej automatyzacji, ktora odpala program okienkowy i zaraz potem
sprawdza jego skutki (instalatory, eksportery, Blender w trybie okienkowym).
Wzorzec "uruchom i natychmiast zmierz plik wyjsciowy" jest bledny zawsze, gdy
uruchamiany program nie jest konsolowy.

Klasa bledu jest gorsza niz zwykla awaria: daje **falszywy zielony**. Skrypt
melduje sukces, mierzac stan sprzed swojej wlasnej pracy.

## Related
- [[build-is-the-only-truth-editor-lies]]
- [[gate-must-have-provable-failure-mode]]
