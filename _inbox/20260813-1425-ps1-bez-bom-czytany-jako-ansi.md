---
title: Skrypt .ps1 bez BOM jest czytany jako ANSI i wywala się na polskich znakach
type: lesson
status: draft
confidence: high
verified: '2026-08-13'
date: 2026-08-13
project: TelegramBridge
tags:
- powershell
- kodowanie
- utf-8
- bom
- windows
- narzedzia
applies_to: []
source: 'Faza 4 Mostka: install-task.ps1 i start.ps1 nie parsowały się'
severity: high
suggested-category: tooling/lessons
time_lost: '~20 min'
---

# Skrypt .ps1 bez BOM jest czytany jako ANSI i wywala się na polskich znakach

## Problem

`install-task.ps1` (zapisany narzędziem edytującym pliki jako UTF-8 **bez BOM**) nie dawał się
uruchomić. Komunikaty parsera wskazywały na linie, które były całkowicie poprawne:

```
LINIA 63 kol 69: Missing argument in parameter list.
LINIA 81 kol 55: The '<' operator is reserved for future use.
LINIA 88 kol 59: The string is missing the terminator: ".
LINIA 48 kol 75: Missing closing '}' in statement block or type definition.
```

Ta sama linia 63 wklejona do konsoli parsowała się **bez błędu**. To był sygnał, że problem nie
leży w składni.

Prawdziwą przyczynę widać dopiero, gdy się zobaczy, jak PowerShell czyta plik:

```
"zadanie juĹĽ istnieje â€” nadpisujÄ™"     ← tak PowerShell widzi "zadanie już istnieje — nadpisuję"
```

## Root cause

**Windows PowerShell 5.1 (powershell.exe) zakłada, że plik `.ps1` bez BOM jest w kodowaniu ANSI
strony kodowej systemu** (u nas windows-1250), nie w UTF-8.

Każdy polski znak zapisany w UTF-8 to 2 bajty, więc po zdekodowaniu jako ANSI zamienia się w dwa
przypadkowe znaki. Przy myślniku `—` (U+2014, 3 bajty) robi się `â€"` — a w tym `"` to **prawdziwy
cudzysłów prosty**, który rozjeżdża parowanie stringów w całym pliku od tego miejsca.

Stąd kaskada: parser zgłasza błędy dziesiątki linii dalej, w miejscach zupełnie niewinnych.
Ozdobniki typu `─────` w komentarzach są równie groźne jak polskie znaki w kodzie.

⚠️ To dotyczy **także komentarzy** — mojibake w komentarzu potrafi wnieść cudzysłów do kodu.

⚠️ PowerShell 7 (`pwsh.exe`) domyślnie zakłada UTF-8 i **tego błędu nie ma** — czyli skrypt może
działać przy ręcznym teście w nowszej konsoli, a paść pod `powershell.exe` z Harmonogramu zadań.

## Solution

Zapisywać każdy `.ps1` zawierający znaki spoza ASCII jako **UTF-8 z BOM**:

```powershell
$tresc = [System.IO.File]::ReadAllText($plik, [System.Text.Encoding]::UTF8)
$utf8bom = New-Object System.Text.UTF8Encoding($true)
[System.IO.File]::WriteAllText($plik, $tresc, $utf8bom)
```

Kontrola — pierwsze trzy bajty mają być `239,187,191`:

```powershell
[System.IO.File]::ReadAllBytes($plik)[0..2] -join ','
```

**Diagnostyka zamiast zgadywania** — parser AST podaje wszystkie błędy naraz, a nie tylko pierwszy:

```powershell
$errs=$null; $toks=$null
[System.Management.Automation.Language.Parser]::ParseFile($plik,[ref]$toks,[ref]$errs) | Out-Null
$errs | ForEach-Object { "linia {0}: {1}" -f $_.Extent.StartLineNumber, $_.Message }
```

Gdy komunikat wskazuje linię, która wygląda poprawnie — **wypisz tę linię przez `Get-Content`
i sprawdź, czy polskie znaki nie są krzakami**. To rozstrzyga w 5 sekund.

## What didn't work

- **Czytanie komunikatu parsera dosłownie.** Wskazywał linię 63, potem 81, potem 88 — wszystkie
  poprawne. Zmieniałem w nich składnię (interpolację `"${env:USERNAME}:(R,W)"` na zmienne, `<->`
  na `-`) i błędy tylko przesuwały się dalej. Każda taka „poprawka" była leczeniem objawu.
- **Test linii w izolacji.** Wklejona do konsoli parsowała się czysto — co *powinno* było od razu
  skierować mnie na kodowanie pliku, a nie na składnię. Rozbieżność „działa w konsoli, nie działa
  w pliku" jest w PowerShellu prawie zawsze kodowaniem.
- **Sprawdzanie parzystości cudzysłowów w pliku** (skryptem w Node, czytającym plik jako UTF-8).
  Pokazało zero problemów — bo w UTF-8 plik JEST poprawny. Narzędzie diagnostyczne musi czytać
  plik **tak jak czyta go ofiara**, nie tak, jak jest zapisany.

## Transferability

Dotyczy każdego projektu na Windowsie, w którym generowane albo edytowane są skrypty `.ps1`
z polskim tekstem — instalatory, skrypty buildów, narzędzia do Unity, automatyzacje Blendera,
zadania Harmonogramu. Ryzyko jest **systematyczne**, bo większość edytorów i narzędzi AI zapisuje
domyślnie UTF-8 bez BOM, a `powershell.exe` (nadal domyślny w Harmonogramie zadań) wymaga BOM.

Najgroźniejszy wariant: skrypt przetestowany ręcznie w `pwsh` przechodzi, a pod zadaniem
z Harmonogramu (`powershell.exe`) pada — i to dopiero po restarcie komputera.

**Reguła:** `.ps1` z polskimi znakami → UTF-8 z BOM, zawsze. Alternatywnie: trzymać `.ps1`
w czystym ASCII (bez ogonków i ozdobników), co całkowicie usuwa klasę problemu.

## Related
- [[20260813-1120-powershell-python-granica-kodowania]] — ta sama rodzina: granice kodowania
  między PowerShellem a resztą świata
- [[20260810-1500-powershell-npx-scoped-package-splatting]] — inne pułapki składni PowerShella
- [[20260801-0945-funkcja-powershell-zwraca-wszystko-wiec-bramka-zawsze-przechodzi]]
