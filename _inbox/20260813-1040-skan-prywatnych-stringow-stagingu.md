---
title: Skan prywatnych stringow (ASCII+UTF-16) jako staly krok stagingu wydania
type: pattern
status: draft
confidence: high
verified: ''
date: 2026-08-13
project: Kerf - Sawmill Tycoon
tags:
- release
- privacy
- security
- steam
- powershell
suggested-category: pipeline/patterns
---

# Skan prywatnych stringow (ASCII+UTF-16) jako staly krok stagingu wydania

## Kontekst
Przed wydaniem gry kazdy plik, ktory dostanie gracz, trzeba przeskanowac pod katem danych
dewelopera: nick, mail, sciezki dysku, tokeny, nazwy kanalow. Jednorazowy audyt NIE wystarcza -
kazdy rebuild uniewaznia wynik, wiec skan musi byc czescia skryptu stagingu (twardy blad =
staging nie powstaje).

## Wzorzec
1. Lista wzorcow per projekt: nick(i) dewelopera, prefiks maila, '@gmail', sciezki
   ('D:\Unity', 'C:\Users'), nazwy vaultow/narzedzi ('GameDevOS', 'Obsidian', '_Handoff'),
   'oauth:', konkretne identyfikatory znalezione w przeszlosci (room-id itp.).
2. KAZDY plik stagingu czytac jako bajty i skanowac TRZY reprezentacje:
   ASCII (serializacja Unity, YAML, JSON), UTF-16 od bajtu 0 ORAZ UTF-16 od bajtu 1.
   Skan samego ASCII przepusci wszystko, co siedzi w kodzie gry, a UTF-16 liczony
   tylko od zera ma SLEPA STREFE: stringi w stercie #US DLL-ek Mono nie maja gwarancji
   parzystego wyrownania (dowod 2026-08-13: literal widoczny dopiero z przesunieciem 1).
2b. Skaner MUSI miec samotest-dzwignie odpalana przy kazdym uzyciu (wzorzec wstrzykniety
   na nieparzystym offsecie ma byc zlapany, inaczej twardy blad), a sciezka porazki
   (throw) MUSI sprzatac niedoszla kopie ze stagingu - przerwany staging zostawil raz
   476 MB brudnego builda o krok od uploadu. Oba mechanizmy udowodnic atrapa.
3. Dozwolone wyjatki przez KONTEKST trafienia (nie przez plik): sciezka pdb w naglowku RSDS
   lokalnie kompilowanych DLL (regex 'Library\\Bee|\.pdb') to standard .NET bez nazwiska.
4. Kazde inne trafienie = throw z nazwa pliku, wzorcem i ~200 znakami kontekstu.

Implementacja referencyjna: `_Handoff/Steam/stage_steam_content.ps1` w Kerf (funkcja
Assert-NoPrivateStrings).

## Pulapki
- Kod gry moze NIE siedziec w Assembly-CSharp.dll (asmdef -> wlasna nazwa DLL,
  u nas TimberTycoon.Runtime.dll; Assembly-CSharp to byl pusty stub 8.7 KB).
- Build macOS niesie te same DLL-e i globalgamemanagers - skanowac obie platformy.
- Dane testowe w sondach/testach kompilowanych do builda tez wychodza do graczy
  (u nas: prawdziwy kanal Twitch i room-id w probkach parsera IRC).

## Related
- [[20260813-1030-unity-cloud-id-wycieka-do-builda]]
