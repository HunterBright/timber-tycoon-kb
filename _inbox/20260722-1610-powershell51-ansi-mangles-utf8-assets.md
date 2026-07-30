---
type: anti-pattern
project: Timber Tycoon
suggested-category: workflow/anti-patterns
tags: [powershell, encoding, utf-8, unity, scriptableobject, automation, batch-edit]
date: 2026-07-22
status: draft
---

# Skrypt PowerShell 5.1 psuje polskie znaki przy masowej edycji plików assetów

## Co się stało

Skrypt PowerShell miał podmienić liczby i nazwy w 13 plikach `.asset` Unity (YAML, UTF-8).
Nazwy typu `Regał na zrębki poj. 63` zapisały się jako `RegaĹ‚ na zrÄ™bki`, czyli podwójnie
zakodowane. Unity nie protestuje - po prostu pokazuje graczowi krzaki.

## Dlaczego

Windows PowerShell 5.1 czyta pliki `.ps1` **jako ANSI** (strona kodowa systemu), a nie UTF-8,
o ile plik nie ma BOM. Literał `'Regał'` zapisany w skrypcie w UTF-8 zostaje w pamięci
odczytany jako dwa znaki ANSI, a potem zapisany do pliku wyjściowego ponownie w UTF-8.
Podwójne kodowanie.

Pułapka jest cicha z dwóch powodów: skrypt kończy się bez błędu, a `git diff` na pliku
trzymanym w LFS pokazuje wyłącznie zmianę wskaźnika, więc krzaków w ogóle nie widać.

## Jak robić

**Trzymaj plik `.ps1` w czystym ASCII i składaj znaki spoza ASCII z kodów:**

```powershell
$lStroke = [char]0x142   # l z kreską
$eOgonek = [char]0x119   # e z ogonkiem
$name = "Rega$lStroke na zr${eOgonek}bki poj. $n"
[System.IO.File]::WriteAllText($path, $text, (New-Object System.Text.UTF8Encoding($false)))
```

Alternatywa, gdy format docelowy ją wspiera (YAML Unity wspiera): pisz sekwencje `\uXXXX`
w cudzysłowie, wtedy w pliku lądują same znaki ASCII i kodowanie nie ma jak się zepsuć.

**Zawsze pisz przez `[System.IO.File]::WriteAllText` z jawnym `UTF8Encoding($false)`.**
`Set-Content` domyślnie zapisuje ANSI, a `Out-File` bywa UTF-8 z BOM - obie opcje psują
pliki, które czyta inne narzędzie.

## Kontrola po przebiegu

Odczyt z jawnym kodowaniem, nie na oko w terminalu (terminal ma własną stronę kodową
i sam potrafi skłamać w obie strony):

```powershell
Get-Content $path -Encoding UTF8 | Select-String -Pattern "displayName" -Context 0,1
```

## Sygnał ostrzegawczy

Jeśli plik siedzi w Git LFS, `git diff` pokaże tylko `oid` i `size` - masowa edycja tekstu
jest wtedy NIEWIDOCZNA w przeglądzie zmian. Przy takich plikach kontrola treści po zapisie
jest obowiązkowa, bo recenzja diffa jej nie zastąpi.
