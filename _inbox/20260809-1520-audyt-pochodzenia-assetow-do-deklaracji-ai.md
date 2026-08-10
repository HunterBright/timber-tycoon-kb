---
title: Audyt pochodzenia assetów pod deklarację AI - licz z manifestu builda, nie z katalogu Assets
type: lesson
status: draft
confidence: high
verified: '2026-08-09'
date: 2026-08-09
project: Kerf - Sawmill Tycoon
tags:
- unity
- provenance
- steam
- compliance
- ai-disclosure
- forensics
applies_to: []
source: 'sesja 2026-08-09, aktualizacja AI Generated Content Disclosure dla AppID 5010520'
severity: high
suggested-category: pipeline/lessons
time_lost: '~2 h śledztwa, z czego ~40 min na trzy fałszywe alarmy'
---

# Audyt pochodzenia assetów pod deklarację AI - licz z manifestu builda, nie z katalogu Assets

## Problem

Trzeba było zaktualizować obowiązkową deklarację treści generowanych przez AI na stronie sklepu.
Deklaracja sprzed miesiąca wymieniała jako treść AI modele postaci, których w grze **już nie było**
(zostały wymienione), a pomijała góry, krzewy i animacje, które **w grze były**. Katalog `Assets/`
zawierał jedno i drugie, więc odpowiedź „co jest w grze" wyczytana z katalogu byłaby fałszywa
w obie strony naraz.

## Root cause

Dwa niezależne mechanizmy:

1. **Katalog projektu to archiwum, nie spis treści gry.** Leżą w nim porzucone wersje, kopie
   zapasowe, katalogi testowe. Unity pakuje do builda tylko to, do czego prowadzi odwołanie ze
   sceny lub z `Resources`. Rozjazd był ogromny: 43 pliki ze znacznikiem generatora w `Assets/`,
   z czego do gry wchodziło 12.
2. **Wyszukiwanie krótkich słów w plikach binarnych, bez rozróżniania wielkości liter, produkuje
   śmieci.** Trzy z czterech „znalezisk" były przypadkowymi bajtami.

## Solution

**Źródło prawdy = sekcja „Used Assets and files from the Resources folder" z logu builda.**
Unity wypisuje ją przy każdym buildzie razem z rozmiarami. To jest lista tego, co gracz dostaje.

Procedura, która się sprawdziła:

1. Wytnij listę z logu builda do osobnego pliku, rozdziel na modele / tekstury / audio.
2. Skanuj **tylko te pliki** pod kątem znaczników generatora.
3. Każde trafienie obejrzyj **dosłownie**, z kontekstem po obu stronach - nie ufaj samej liczbie.
4. Dla każdej kategorii treści zapisz wiersz: kategoria → czym zrobione → **plik jako dowód**.

Znaczniki, które realnie działają:

| Format | Gdzie szukać | Co zdradza |
|---|---|---|
| FBX (binarny) | nazwy węzłów (`tripo_mesh_*`), pola `Original\|ApplicationName` | generator albo program eksportujący |
| GLB | JSON w nagłówku: `"generator"`, nazwy materiałów i siatek | jw. |
| PNG | bloki tekstowe `tEXt` / `iTXt` | ComfyUI pisze `prompt`/`workflow`, A1111 `parameters`, Blender `File`/`Date`/`Time` |
| MP3 | ramka `GEOB` w tagu ID3 | **manifest C2PA** - podpisany certyfikatem wystawcy, nie do podrobienia i nie do przeoczenia |

Manifest C2PA okazał się najmocniejszym dowodem w całym audycie: rozstrzygnął pochodzenie muzyki
wbrew temu, co mówił nasz własny rejestr pochodzenia.

## What didn't work

- **`rg -a -i` z krótkimi wzorcami po plikach binarnych.** Fałszywe alarmy, każdy wyglądał wiarygodnie:
  - `meshy` → w binarnym FBX token `Mesh` skleja się z kolejnym bajtem (`MeshY`, `Meshy`, `Mesht`).
    Trafiło 22 pliki, w tym drzewa i meble. Zero z nich to generator.
  - `qwen` → ciąg `JqwENJ` w skompresowanych danych PNG.
  - `udio` (generator muzyki) → słowo „**A**udio" w opisie kanałów pliku WAV.
- **Poleganie na samym znaczniku generatora.** Znacznik ginie przy pierwszym ponownym eksporcie.
  Zadziałał dopiero **odcisk palca wersji programu**: nasz potok stoi na Blenderze 5.0-5.2,
  a surowe modele postaci miały w nagłówku `4.2.3 LTS` - czyli wyszły z cudzej maszyny
  (konwertera usługi generującej). Ta metoda wykryła pochodzenie tam, gdzie skan nazw milczał.
  Warto przeskanować tak **cały** manifest wydania i zobaczyć rozkład wersji: wszystko spoza
  Twoich wersji jest z zewnątrz i wymaga wyjaśnienia.
- **Rejestr pochodzenia prowadzony z pamięci.** Nasz własny dokument przypisywał muzykę
  niewłaściwemu narzędziu i twierdził, że w grze siedzi 13 modeli, których tam nie ma. Dokument
  bez dowodu na plik jest hipotezą.
- **Zaufanie własnemu wpisowi „sprawdzone, nic nie ma".** Rejestr zamykał sprawę generatora
  wykluczonego licencyjnie zdaniem „ani jeden plik nie zawiera znacznika" - prawdziwym w dniu
  pomiaru. Trzy dni później z tego generatora powstała **cała obsada gry**, a ponowny skan
  znaczników tego nie wykrył, bo eksport przez Blender znacznik kasuje. Błąd wyszedł dopiero,
  gdy człowiek powiedział „to nieprawda, modele robił generator X".

## Pułapka: manifest wydania też się starzeje

Skany robiłem na buildzie z rana. W trakcie tego samego dnia pracy powstał nowy build z wdrożoną
zmianą animacji - i gotowe już zdanie deklaracji („część animacji nadal pochodzi z biblioteki
motion capture") stało się nieprawdą, zanim ktokolwiek je przeczytał. Wyszło tylko dlatego, że
przy innym sprawdzeniu rzucił się w oczy zmieniony znacznik czasu pliku z logiem.

**Zasada:** przy zadaniu opartym na artefakcie wydania sprawdzaj znacznik czasu tego artefaktu
**dwa razy - na starcie i tuż przed oddaniem pracy**. Gdy artefakt się zmienił, nie skanuj
wszystkiego od zera: **porównaj stary manifest z nowym**. Różnica jest zwykle kilkupozycyjna
i od razu widać, czy dotyka wniosków.

## Transferability

Dotyczy każdego projektu, który wchodzi na sklep z obowiązkiem deklaracji AI (Steam od 2024,
podobne wymogi wchodzą gdzie indziej), i każdego audytu licencyjnego w dowolnym silniku.
Sedno jest niezależne od Unity: **deklarujesz to, co trafia do odbiorcy, więc licz z artefaktu
wydania, nie z warsztatu.**

Najważniejsza i najłatwiejsza do przeoczenia: **wpis „sprawdzone, nic nie ma" ma datę ważności.**
Rejestr pochodzenia jest zdjęciem stanu z dnia pomiaru, a nie gwarancją na przyszłość. Jeżeli
narzędzie zostało kiedyś odrzucone z powodu licencji, moment jego ponownego użycia trzeba złapać
**przy użyciu** - po fakcie skan tego nie wyłapie, bo ponowny eksport przez Blender kasuje znacznik
generatora. Skan dowodzi więc, że coś JEST wygenerowane - nie dowodzi, że coś NIE JEST.
Dlatego tekst deklaracji pisz listą otwartą („na przykład..."), a nie zamkniętym wyliczeniem, i trzymaj
osobno pytania bez rozstrzygnięcia. Zamknięta lista przy nieudowadnialnej negatywnej tezie to
zobowiązanie, którego nie da się dotrzymać.

## Related
- `Docs/_provenance/REJESTR.md` (rejestr pochodzenia - poprawiony tą sesją)
- `_Handoff/STEAM_ANKIETA_AI_2026-08-09.md` (deliverable: tekst + tabela dowodowa + fałszywe alarmy)
