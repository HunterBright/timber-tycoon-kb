---
title: Windows PowerShell 5.1 czyta `.ps1` jako ANSI — literały Unicode w skrypcie się sypią
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-06-24'
project: Kerf - Sawmill Tycoon
tags:
- powershell
- encoding
- utf8
- unicode
- scripting
- localization
- batch-edit
applies_to: []
source: ''
suggested-category: tooling/lessons
---

# Windows PowerShell 5.1 czyta `.ps1` jako ANSI — literały Unicode w skrypcie się sypią

## Problem / kontekst
Skrypt do hurtowej edycji 14 plików lokalizacji miał tablicę tłumaczeń z japońskim/koreańskim/
chińskim wprost w kodzie (`ja="駐車スペース"`). Plik `.ps1` zapisany jako UTF-8 BEZ BOM. Windows
PowerShell 5.1 uruchamiając skrypt zinterpretował go w systemowej stronie kodowej (Windows-1252),
przez co znaki CJK zamieniły się w mojibake i wywaliły parser („Unexpected token", „hash literal
incomplete"). To NIE błąd logiki — to błąd kodowania odczytu samego skryptu.

## Reguła
Trzymaj kod skryptu w **czystym ASCII**, a wszystkie wartości Unicode (tłumaczenia, ścieżki ze
znakami diakrytycznymi) w OSOBNYM pliku danych (np. `*.json`) i wczytuj go jawnie jako UTF-8:
`[System.IO.File]::ReadAllText($path, [System.Text.Encoding]::UTF8) | ConvertFrom-Json`.
Zapis wynikowych plików: `[System.IO.File]::WriteAllText($p, $txt, (New-Object System.Text.UTF8Encoding($false)))`
— UTF-8 bez BOM (nie używać `Out-File -Encoding utf8`, bo w 5.1 dorzuca BOM).

## Dlaczego to działa
- `.ps1` w ASCII → PS 5.1 nie ma czego źle dekodować, parser zadowolony.
- Dane czytane przez `Encoding.UTF8` → poprawne znaki w pamięci niezależnie od ustawień konsoli.
- `[IO.File]::ReadAllText`/`WriteAllText` omijają domyślne (UTF-16) kodowanie cmdletów PS 5.1.

## Dodatkowo (przy hurtowej edycji JSON)
- Edycja TEKSTOWA (regex insert/replace) zachowuje ręczne formatowanie pliku; `ConvertTo-Json`
  przeformatowałby cały plik (utrata grup/wcięć, escapowanie \uXXXX) → ogromny diff.
- Rób operacje IDEMPOTENTNYMI (sprawdź czy klucz już jest przed wstawieniem) — bezpieczny re-run.
- Po edycji zweryfikuj parytet/poprawność: policz wystąpienia kluczy i `ConvertFrom-Json` w pętli.
