---
title: Granica PowerShell - Python: BOM i ANSI psuja pliki w obie strony
type: lesson
status: draft
confidence: high
verified: '2026-08-13'
date: 2026-08-13
project: GameDevOS
tags:
- powershell
- python
- windows
- kodowanie
- automatyzacja
applies_to: []
source: 'D:\GameDevOS\tools\zamek.ps1, D:\GameDevOS\tools\bramka_raportu.py'
severity: high
suggested-category: workflow/lessons
time_lost: 'dwa osobne bledy w jednej sesji, oba znalezione przed uruchomieniem'
---

# Granica PowerShell - Python: BOM i ANSI psuja pliki w obie strony

## Problem

Automat na Windowsie, w ktorym skrypty PowerShella zapisuja pliki czytane
pozniej przez Pythona. Dwa osobne bledy tego samego dnia, oba ciche:

1. **Skrypt zapisal plik, bramka w Pythonie go odrzucila.** `Out-File -Encoding utf8`
   w PowerShellu 5.1 dokleja na poczatku znacznik BOM. Python czytajacy plik
   jako `utf-8` widzi go jako pierwszy znak tresci, wiec sprawdzenie
   „czy plik zaczyna sie od `---`" zawodzi. Bramka odrzucalaby **kazdy** raport
   za rzekomy brak naglowka - i to raport, ktory sama chwile wczesniej poprawila.

2. **Odczyt i zapis tego samego pliku zniszczyl polskie znaki.**
   `(Get-Content plik) -replace ... | Set-Content -Encoding utf8`: `Get-Content`
   bez `-Encoding` czyta w PowerShellu 5.1 jako ANSI, wiec znaki UTF-8 zostaja
   zinterpretowane bajt po bajcie i zapisane ponownie jako UTF-8. Efekt to
   podwojne kodowanie (`„` zamienia sie w `â€ž`).

Trzeci wariant tej samej granicy jest najgorszy: **litera z ogonkiem w skrypcie
`.ps1` bez BOM**. PowerShell czyta taki plik jako ANSI, znak rozpada sie na dwa
i jesli trafil do tekstu w apostrofach, **skrypt przestaje sie parsowac**.
Jedna litera w wyrazeniu regularnym wywrocila caly plik.

## Root cause

PowerShell 5.1 i Python maja **odwrotne domyslne zalozenia** co do znacznika BOM:

| | PowerShell 5.1 | Python |
|---|---|---|
| plik BEZ BOM | czyta jako ANSI (strona kodowa systemu) | czyta jako UTF-8 poprawnie |
| plik Z BOM | czyta poprawnie jako UTF-8 | `utf-8` zostawia BOM w tresci |

Nie ma jednego kodowania, ktore zadowoli oba narzedzia domyslnie. Trzeba
zdecydowac osobno dla kazdego kierunku.

## Solution

Trzy reguly, kazda pilnujaca jednej strony granicy:

1. **Skrypty `.ps1` pisz czystym ASCII** - bez polskich ogonkow i typograficznych
   cudzyslowow. Wtedy sposob odczytu przestaje mieć znaczenie. (Alternatywa,
   czyli zapisanie skryptu z BOM, tez dziala, ale ASCII jest odporniejsze:
   przetrwa kazde narzedzie, ktore po drodze BOM zgubi.)
2. **Pliki dla Pythona zapisuj bez BOM**, jawnie:
   `[System.IO.File]::WriteAllText($p, $t, (New-Object System.Text.UTF8Encoding($false)))`
   zamiast `Out-File -Encoding utf8`.
3. **W Pythonie czytaj `encoding="utf-8-sig"`**, nie `"utf-8"`. Dla plikow bez BOM
   wynik jest identyczny, a plik z BOM przestaje wywracac sprawdzenia.
   To jest obrona przed kazdym narzedziem Windows, ktore BOM dolozy w przyszlosci.

Do tego jeden mechaniczny sprawdzian, ktory wykrywa mine, zanim wybuchnie -
plik z bajtami spoza ASCII i bez BOM:

```powershell
Get-ChildItem *.ps1 | ForEach-Object {
    $b = [System.IO.File]::ReadAllBytes($_.FullName)
    $bom = ($b.Length -ge 3 -and $b[0] -eq 0xEF)
    $poza = ($b | Select-Object -Skip $(if($bom){3}else{0}) | Where-Object { $_ -gt 127 }).Count
    if ($poza -and -not $bom) { "MINA: $($_.Name)" }
}
```

## What didn't work

Poleganie na tym, ze skrypt **sie parsuje**. Plik z podwojnie zakodowanym
znakiem w komentarzu parsuje sie bez bledu i wyglada zdrowo; ten sam znak
w tekscie w apostrofach wywraca wszystko. Roznica jest przypadkowa, wiec
„dziala u mnie" nic tu nie dowodzi.

Poleganie na wygladzie w konsoli tez zawodzi w druga strone: `Get-Content`
pokazuje krzaki takze dla plikow, ktore na dysku sa poprawne, bo sam czyta
je jako ANSI. Jedyny wiarygodny pomiar to odczyt bajtow.

## Transferability

Dotyczy kazdego projektu na Windowsie, w ktorym PowerShell i Python (albo
dowolne narzedzie uniksowe) podaja sobie pliki: potoki assetow, generatory
konfiguracji, skrypty budowania, eksport z Blendera do Unity.

Regula ogolna, szersza niz kodowanie: **na granicy dwoch narzedzi nie zakladaj
niczego o domyslnych ustawieniach - podaj je jawnie po obu stronach.** Domyslne
zachowania sa dobierane pod wygode, a nie pod wspolprace, i po obu stronach
granicy bywaja przeciwne.

## Related

- [[20260813-0940-automat-konczy-sie-zielono-choc-nic-nie-wyslal]]
