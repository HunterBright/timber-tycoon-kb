---
title: Windows PowerShell czyta stdin w kodowaniu OEM (CP852), a nie UTF-8 - polskie znaki w payloadzie gina
type: lesson
status: draft
confidence: high
verified: '2026-08-14'
date: 2026-08-14
project: AnotherQuest
tags:
- powershell
- encoding
- utf-8
- cp852
- stdin
- claude-code
- hooks
applies_to:
- windows-powershell-5.1
- claude-code-hooks
source: 'zmierzone: [Console]::InputEncoding = ibm852 (CP 852) przy spawnie z Node'
severity: high
suggested-category: workflow/lessons
time_lost: '~40 min'
---

# Windows PowerShell dekoduje stdin kodowaniem OEM, nie UTF-8

## Problem

Hook `UserPromptSubmit` w PowerShellu mial wykrywac pytania informacyjne po slowach-kluczach
(`"co mysl"`, `"jak bys"`). Dzialal dla promptow angielskich, a dla polskich **cicho nie robil nic**.
Czyli byl martwy dokladnie dla tych promptow, dla ktorych go napisano.

Skrypt konczyl sie kodem 0 i pustym wyjsciem - identycznie jak przy poprawnym "brak trafienia".
Zadnego bledu, zadnego sladu.

## Root cause

Proces nadrzedny (Claude Code, Node) podaje payload jako **UTF-8**: `stdin.write(json, "utf8")`.
Windows PowerShell 5.1 dekoduje stdin uzywajac `[Console]::InputEncoding`, ktore domyslnie jest
**kodowaniem OEM konsoli** - na polskim Windowsie **CP852**, nie UTF-8.

Efekt: `[Console]::In.ReadToEnd()` rozbija kazdy znak spoza ASCII na dwa smieciowe. Zmierzone:

    "Co myslisz" (z 's z kreska', U+015B)
    UTF-8 na wejsciu:  0xC5 0x9B
    po dekodowaniu CP852:  U+253C U+0164   ("┼" + "Ť")

Normalizacja ogonkow w skrypcie (`Replace([char]0x015B, 's')`) nie miala wiec czego zamienic -
w lancuchu nie bylo juz zadnego U+015B. `Contains("co mysl")` nie trafial. Kod 0, cisza.

To sa **dwie warstwy jedna na drugiej**: najpierw trzeba poprawnie ODCZYTAC znak, dopiero potem
mozna go znormalizowac. Sama normalizacja bez poprawnego dekodowania jest bezuzyteczna.

## Solution

Nie czytaj stdin jako tekstu. Czytaj **bajty** i dekoduj je jawnie:

    try { [Console]::OutputEncoding = New-Object System.Text.UTF8Encoding $false } catch { }

    $bufor = New-Object System.IO.MemoryStream
    [Console]::OpenStandardInput().CopyTo($bufor)
    $raw = [System.Text.Encoding]::UTF8.GetString($bufor.ToArray())
    if ($raw.Length -gt 0 -and $raw[0] -eq [char]0xFEFF) { $raw = $raw.Substring(1) }

Trzy rzeczy naraz:
1. `OpenStandardInput()` + `UTF8.GetString()` - dekodowanie po naszej stronie, niezalezne od CP konsoli.
2. `[Console]::OutputEncoding` na UTF-8 **bez BOM** - bo proces nadrzedny czyta stdout/stderr jako UTF-8,
   wiec komunikat zwrotny ze sciezka zawierajaca polskie znaki tez musi wyjsc poprawnie.
3. Zdjecie BOM-a z poczatku, jesli sie trafi.

**Nie ustawiaj `[Console]::InputEncoding`** - przy przekierowanym stdin potrafi rzucic wyjatkiem,
a i tak jest za pozno, jesli strumien zostal juz czesciowo odczytany.

## What didn't work

- **Testowanie skryptu recznie w terminalu** - w terminalu kodowanie jest inne niz przy spawnie
  z przekierowanym stdin, wiec skrypt "dzialal" i defekt sie nie pokazywal. To byl glowny powod,
  dla ktorego blad przezyl wczesniejsza weryfikacje "6 payloadow, logika poprawna".
- **Zakladanie, ze skoro angielski dziala, to kodowanie jest OK** - ASCII przechodzi przez kazde
  kodowanie jednobajtowe bez szwanku i maskuje problem.
- **Podejrzenie o liste slow-kluczy** - lista byla poprawna, zawodzil transport.

## Transferability

Dotyczy **kazdego** skryptu PowerShell (5.1) odbierajacego dane na stdin od procesu nie-windowsowego:
hooki Claude Code, git hooks, filtry w potokach, integracje z Node/Pythonem. Wszedzie, gdzie po jednej
stronie jest UTF-8, a po drugiej Windows PowerShell.

PowerShell 7 (pwsh) ma domyslnie UTF-8 i tego problemu nie ma - ale nie mozna na tym polegac,
bo pwsh nie jest zainstalowany domyslnie, a fallback trafia wlasnie na `powershell.exe` 5.1.

**Regula diagnostyczna:** hook, ktory "nie dziala tylko dla polskich znakow", to prawie zawsze
kodowanie stdin, nie logika. Zmierz `[Console]::InputEncoding.CodePage` zanim zaczniesz poprawiac regexy.

## Related

- 20260814-1145-hooki-claude-code-windows-forma-exec.md (ten sam hook, inna warstwa awarii)
