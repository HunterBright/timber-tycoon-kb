---
title: Compress-Archive (PS 5.1) do paczek dla macOS
type: anti-pattern
status: draft
confidence: low
verified: ''
date: '2026-07-23'
project: Kerf - Sawmill Tycoon
tags:
- powershell
- zip
- macos
- packaging
- compress-archive
- bsdtar
applies_to: []
source: ''
suggested-category: tooling/anti-patterns
---

# Compress-Archive (PS 5.1) do paczek dla macOS

## Anty-wzorzec

Pakowanie buildu dla macOS (albo czegokolwiek rozpakowywanego na Linux/macOS)
przez `Compress-Archive` w Windows PowerShell 5.1.

## Dlaczego nie dziala

`Compress-Archive` na PS 5.1 (.NET Framework) zapisuje sciezki wpisow zipa
z BACKSLASHAMI (`Kerf.app\Contents\MacOS\...`). Windows rozpakuje taki zip
poprawnie, ale macOS/Linux traktuja `\` jako czesc NAZWY pliku - zamiast
struktury folderow powstaja plaskie pliki o nazwach z backslashami.
Dla bundla `.app` oznacza to martwa aplikacje, a blad wychodzi dopiero
u odbiorcy na Macu.

## Co robic zamiast tego

Na kazdym Windows 10/11 lezy `C:\Windows\System32\tar.exe` (bsdtar) - pisze
poprawne ukosniki `/`:

```
tar.exe -a -cf paczka.zip -C folder_staging NazwaFolderu
```

Weryfikacja po spakowaniu: `tar -tf paczka.zip` i asercja, ze zaden wpis
nie zawiera `\`.

Uwaga dodatkowa: ZADEN zip robiony na Windows nie przenosi unixowych bitow
wykonywalnosci - odbiorca na Macu i tak potrzebuje `chmod +x` na binarce
(plus `xattr -dr com.apple.quarantine` na kwarantanne Gatekeepera).
