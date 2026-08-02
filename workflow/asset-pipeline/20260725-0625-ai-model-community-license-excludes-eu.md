---
title: Nie stawiaj pipeline'u assetow na modelu AI z licencja "Community", nie czytajac pierwszej linii LICENSE
type: anti-pattern
status: verified
confidence: high
verified: '2026-08-01'
date: '2026-07-25'
project: Kerf - Sawmill Tycoon
tags:
- ai-assets
- licencje
- hunyuan3d
- pipeline
- steam
- prawo
applies_to: []
source: 'https://huggingface.co/tencent/Hunyuan3D-2/raw/main/LICENSE'
promoted: '2026-07-30'
---

# Nie stawiaj pipeline'u assetow na modelu AI z licencja "Community", nie czytajac pierwszej linii LICENSE

## The trap
Lokalny generator 3D (Tencent Hunyuan3D-2 / 2.1) stawia sie w godzine, chodzi na wlasnym GPU,
"nie wysyla nic do chmury", a wagi sa publiczne na HuggingFace. Wyglada jak darmowe,
bezpieczne narzedzie do produkcji assetow do gry komercyjnej.

## Why it fails
Licencje z rodziny "Tencent Hunyuan ... Community License" maja klauzule TERYTORIUM.
Plik `LICENSE`, linia 3, wielkimi literami:

> THIS LICENSE AGREEMENT DOES NOT APPLY IN THE EUROPEAN UNION, UNITED KINGDOM AND SOUTH KOREA

"Territory" = caly swiat MINUS Unia Europejska, Wielka Brytania, Korea Poludniowa,
a licencja jest udzielona "for the Territory only". Sekcja 5.c zabrania uzywania,
kopiowania, modyfikowania, rozpowszechniania i POKAZYWANIA nie tylko modelu,
ale takze jego wynikow ("Output or results") poza Territory.

Dla studia w UE oznacza to, ze poza licencja jest zarowno samo uruchamianie modelu,
jak i wrzucenie wygenerowanej siatki do gry sprzedawanej w UE. Dotyczy to 2.0 i 2.1
identycznie (sprawdzone w obu plikach LICENSE). Klauzula "1 mln aktywnych uzytkownikow
miesiecznie" z sekcji 4 jest myląca: mala skala nie ratuje, bo blokada jest terytorialna,
nie ilosciowa.

## Symptoms
- "Community License" / "Non-Commercial License" w naglowku pliku zrodlowego lub LICENSE.
- Slowo "Territory" w definicjach (sekcja 1) - zawsze sprawdz, co wyklucza.
- Repozytorium modelu bez pliku LICENSE w lokalnym cache HuggingFace
  (warunki trzeba doczytac w repo, nie zakladac, ze sa te same co kodu).

## Correct approach
1. **Przed instalacja** czytaj `LICENSE` (naglowek + definicje "Territory" + sekcje o Output)
   i osobno licencje WAG modelu, bo bywa inna niz licencja kodu.
2. Do assetow komercyjnych w UE bierz modele z licencja permisywna (np. MIT/Apache-2.0);
   licencje "community" traktuj jako czerwona flage do weryfikacji, nie jako zielone swiatlo.
3. Jesli licencja wyklucza terytorium: generator moze zyc na dysku jako zabawka, ale
   NIE wchodzi do pipeline'u produkcyjnego - zadnych blockoutow "tylko do referencji",
   bo zakaz obejmuje takze Output.
4. Zapisz decyzje (ADR), zeby po pol roku ktos nie postawil tego samego pipeline'u drugi raz.

Uwaga: to odczyt pliku licencji, nie porada prawna. Przy watpliwosciach - prawnik.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260801-1140-licencja-modelu-ai-to-trzy-osobne-dokumenty|Licencja modelu AI to trzy osobne dokumenty i wystarczy, ze jeden zabroni]] - wspolne: prawo, licencje
- [[20260801-1245-regulamin-uslugi-a-licencja-wag-to-dwa-rozne-swiaty|Regulamin uslugi w chmurze i licencja pobieranych wag to dwa rozne dokumenty - sprawdzaj oba]] - wspolne: prawo, licencje
- [[20260801-0826-trellis-2-generator-3d-bez-blokady-ue|TRELLIS.2 jako generator 3D bez blokady licencyjnej w UE]] - wspolne: licencje, pipeline
<!-- /POWIAZANE:auto -->
