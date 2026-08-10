---
title: Steam - co da się zmienić na stronie sklepu programowo, a co się zatrzaskuje po publikacji
type: lesson
status: draft
confidence: high
verified: '2026-08-09'
date: 2026-08-09
project: Kerf - Sawmill Tycoon
tags:
- steam
- steamworks
- publishing
- localization
- automation
severity: medium
suggested-category: publishing/lessons
time_lost: '~1 h na ustalenie, że blokada nie dotyczy tłumaczeń tylko całego formularza'
---

# Steam - co da się zmienić na stronie sklepu programowo, a co się zatrzaskuje po publikacji

## Problem

Trzeba było zmienić jedno zdanie w opisie gry (żyje w 14 językach) i dopisać deklarację AI
w kolejnych 12 językach. Panel Steamworks bywa nieprzewidywalny w automatyzacji: część widżetów
przyjmuje sterowanie, część nie.

## Co DZIAŁA

**Opis sklepu w wielu językach: przez plik JSON.** Zakładka Lokalizacja → „Pobierz lokalizację"
(format JSON, wszystkie języki) → podmiana skryptem → „Prześlij lokalizację" → zakładka Opublikuj.
Cała droga daje się zautomatyzować.

Trzy rzeczy, które zaskakują na tej drodze:
- **Pobrany plik ląduje jako `.tmp` z losową nazwą** i tak zostaje (przeglądarka nie domyka nazwy).
  Nie szukaj ładnej nazwy - ten `.tmp` jest kompletnym, poprawnym JSON-em.
- **Wgrywarka lokalizacji przyjmuje pliki tylko z katalogu projektu**, nie z katalogu pobrań.
  (Ta sama strona ma wgrywarkę grafik, która nie działa programowo w ogóle - nie mylić ich.)
- **Podgląd różnic po stronie Steama pokazuje jako zmienione również akapity identyczne** -
  to normalizacja białych znaków przy round-tripie. Nie panikuj i nie cofaj: miarodajny jest diff
  Twoich własnych plików przed wgraniem.

**Bramka, która się opłaciła:** skrypt podmieniający wymagał, żeby każdy wzorzec trafił
**dokładnie raz** w danym języku; przy jakimkolwiek innym wyniku przerywał i nic nie zapisywał.
Wyłapałby literówkę w którymkolwiek z 14 wariantów językowych.

## Czego NIE DA SIĘ zrobić (anty-wzorzec)

**Po opublikowaniu ankiety treści formularz przestaje cokolwiek zapisywać.** Przycisk zapisu
reaguje, znacznik „ostatnio zaktualizowana" się podbija, ale treść jest odrzucana w ciszy.

Sprawdzone cztery drogi wpisywania (ukryte pola formularza, własne zdarzenia widżetu, narzędzie
formularzowe, prawdziwa klawiatura) - każda z tym samym skutkiem.

**Rozstrzygnął dopiero dowód negatywny:** wpisanie czegokolwiek w prywatne pole „uwagi dla zespołu
recenzentów" (nie tłumaczenie, nie treść publiczna) też się nie zapisało. Gdyby problem dotyczył
tłumaczeń, to pole by weszło. Nie weszło, więc blokada obejmuje CAŁY formularz.

Wniosek operacyjny: **wpisz wszystkie języki ZANIM klikniesz publikację ankiety.** Po publikacji
zostaje czekanie na nową wersję roboczą albo kontakt z pomocą.

## Transferability

Dotyczy każdego wydania na Steamie. Sedno metodyczne jest szersze: **gdy automatyzacja panelu
cicho nie działa, znajdź kontrolny przypadek poza podejrzewaną kategorią.** Testowałem cztery
sposoby wpisywania tej samej rzeczy i wszystkie zawiodły tak samo, co niczego nie rozstrzygało.
Dopiero test na polu z zupełnie innej kategorii pokazał, że zmienna, którą podejrzewałem
(tłumaczenia), nie ma z tym nic wspólnego.

## Related
- `_Handoff/STEAM_ANKIETA_AI_2026-08-09.md` (deklaracja AI: treść, dowody, przebiegi kontrolne)
- `_Handoff/STEAM_ANKIETA_AI_TLUMACZENIA.md` (12 gotowych tłumaczeń czekających na odblokowanie)
- `_Handoff/steam_loc/` (skrypt podmiany opisu + oryginał jako ścieżka odwrotu)

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260724-1907-steam-release-timeline-two-reviews-hard-clock|Wydanie gry na Steam: sekwencyjne recenzje + twardy zegar Coming Soon]] - wspolne: publishing, steamworks, steam
- [[20260809-1120-steam-obraz-w-opisie-bez-rozszerzenia|20260809-1120-steam-obraz-w-opisie-bez-rozszerzenia]] - wspolne: steamworks, steam
- [[20260612-1815-headless-tmp-sdf-font-generation|Headless TMP setup: import Essentials + generate SDF font asset from editor script]] - wspolne: localization, automation
- [[20260724-1545-unity-photoshoot-mode-cmdline|Tryb "fotograf" w buildzie: marketingowe screenshoty bez Edytora]] - wspolne: steam, automation
<!-- /POWIAZANE:auto -->
