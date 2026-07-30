---
title: Build Unity na macOS z maszyny Windows (bez Maca)
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-07-23'
project: Kerf - Sawmill Tycoon
tags:
- unity
- macos
- build
- cross-compile
- unity-hub
- headless
- gatekeeper
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Build Unity na macOS z maszyny Windows (bez Maca)

## Kontekst

Trzeba wyslac build gry testerowi z Makiem, a caly pipeline stoi na Windows.
Zweryfikowane na Unity 6000.5.1f1, 2026-07-23.

## Wzorzec

1. **Modul bez klikania**: Unity Hub ma tryb bezokienkowy - instalacja
   Mac Build Support jedna komenda (dziala, zweryfikowane):
   ```
   & "D:\Unity\Unity Hub\Unity Hub.exe" -- --headless install-modules --version 6000.5.1f1 --module mac-mono
   ```
2. **Backend musi byc Mono** - IL2CPP dla macOS NIE zbuduje sie z Windows.
   W skrypcie builda ASERTOWAC Mono (czytelny blad), nie mutowac ustawien po cichu.
3. **Architektura Universal (x64+ARM64) przez refleksje** - typ
   `UnityEditor.OSXStandalone.UserBuildSettings.architecture` zyje w assembly
   modulu Mac; twarda referencja psuje kompilacje projektu na maszynie bez modulu.
   Refleksja + Enum.Parse("x64ARM64"); fallback = default (Apple Silicon ratuje Rosetta).
4. **Kolejnosc buildow**: BuildPlayer PRZELACZA aktywny target edytora (dlugi
   reimport przy pierwszym razie) - najpierw Windows + testy/sondy, Mac NA KONCU.
5. **Pakowanie**: tar.exe, nie Compress-Archive (patrz osobny anty-pattern);
   README dla testera z komendami `chmod +x .../Contents/MacOS/<binarka>` oraz
   `xattr -dr com.apple.quarantine <App>.app` + sciezka awaryjna przez
   Ustawienia systemowe -> Prywatnosc i ochrona -> "Otworz mimo to"
   (na Sonoma/Sequoia prawy-klik-Otworz nie wystarcza dla niepodpisanych aplikacji).
6. **Granica wzorca**: builda z Windows NIE DA SIE zweryfikowac lokalnie
   (start, Metal, Gatekeeper) - pierwszy tester z Makiem JEST smoke testem;
   poprosic o raport zaraz po pierwszym uruchomieniu, zanim zainwestuje czas.
