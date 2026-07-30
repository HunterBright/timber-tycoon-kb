---
title: Rzut prostokątny w ujęciu porównawczym potrafi pokazać kilka obiektów nałożonych na siebie i wyglądać jak jeden poprawny obiekt
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-20'
project: Kerf - Sawmill Tycoon
tags:
- blender
- render
- review-artifact
- orthographic
- occlusion
- false-evidence
applies_to:
- blender
- comparison-renders
- asset-review-pipelines
source: ''
severity: high
time_lost: 1 cykl renderu + 1 sesja z niezauważonym błędem
promoted: '2026-07-30'
---

# Rzut prostokątny w ujęciu porównawczym potrafi pokazać kilka obiektów nałożonych na siebie i wyglądać jak jeden poprawny obiekt

## Problem

Skrypt budował plik przeglądowy z kilkoma wariantami modelu (auta) ustawionymi obok
siebie wzdłuż osi Y, i renderował "widok z boku" każdego z nich osobno: kamera
ortograficzna w `(0, -14, z)`, cel w `(0, y_wariantu, z)`.

Wszystkie warianty leżały na **tej samej linii wzroku**. Najbliższy zasłaniał resztę,
więc każde z "osobnych" ujęć pokazywało w rzeczywistości ten sam model - albo, przy
różnych sylwetkach, ich **nałożone obrysy** czytające się jak jeden dziwny pojazd.

Kluczowe: obraz nie wyglądał na uszkodzony. Wyglądał jak model. Miał sensowne szyby,
słupki i koła. Dopiero porównanie z liczbami z raportu budowy (długość 3.715 m wobec
sylwetki wyglądającej na ~4.4 m) zdradziło, że to nie jest ten obiekt.

Ten sam skrypt w poprzedniej sesji produkował ujęcie podpisane "kombi", które
pokazywało sedana. Nikt tego nie zauważył - ocena została wydana na podstawie ujęć 3/4,
które akurat były poprawne.

## Root cause

Rzut prostokątny (ortho) usuwa perspektywę, czyli **jedyną wskazówkę głębi**, która
zdradziłaby, że w kadrze stoi kilka obiektów jeden za drugim. W rzucie perspektywicznym
dalszy obiekt byłby mniejszy i wystawałby zza bliższego; w ortho oba mają identyczną
skalę i idealnie się pokrywają.

Do tego dochodzi błąd geometryczny w samym ustawieniu: skierowanie kamery na inny cel
wzdłuż osi, na której obiekty są rozstawione, **nie przesuwa kamery w bok** - zmienia
tylko kąt o ułamek stopnia. Intencja "pokaż drugi obiekt" nie ma odzwierciedlenia
w tym, co robi kod.

Ortho + rozstawienie wzdłuż osi patrzenia = cicha awaria, która produkuje
wiarygodnie wyglądający artefakt dowodowy.

## Solution

Przy renderowaniu ujęcia pojedynczego obiektu ze sceny zbiorczej **jawnie ukryj
pozostałe**, zamiast liczyć na to, że kadr je wytnie:

```python
for idx, key in enumerate(VARIANTS):
    for other in VARIANTS:
        hide = other != key
        roots[other].hide_render = hide
        for ch in roots[other].children:
            ch.hide_render = hide          # dzieci NIE dziedziczą hide_render
    shoot(...)
```

Dwie pułapki w samym rozwiązaniu:
- `hide_render` **nie jest dziedziczone** przez dzieci obiektu - trzeba przejść
  hierarchię, inaczej znikną nadwozia, a koła zostaną wiszące w powietrzu.
- Po pętli trzeba **odsłonić wszystko** przed ujęciami grupowymi, inaczej ostatnie
  ukrycie wycieknie do kolejnych renderów.

Weryfikacja: zawsze porównaj wymiar z obrazu z wymiarem z raportu budowy. Jeżeli
pipeline liczy gabaryty, to jest to darmowy test poprawności artefaktu dowodowego.

## What didn't work

- **Ocena "na oko" samego renderu.** Nałożone bryły dają obraz, który wygląda jak
  poprawny model. Nie ma w nim niczego, co krzyczy "błąd".
- **Założenie, że skierowanie kamery na cel wystarczy.** W ortho pozycja kamery wzdłuż
  osi patrzenia nie ma znaczenia dla tego, CO widać - liczy się tylko kierunek i to,
  co stoi na drodze.
- **Zaufanie do skryptu, który raz przeszedł akceptację.** Błąd istniał od poprzedniej
  sesji i przeżył ocenę człowieka, bo ten oceniał inne kadry z tego samego zestawu.

## Transferability

Dotyczy każdego pipeline'u, który generuje **artefakty dowodowe** dla człowieka:
rendery porównawcze w Blenderze, zrzuty widoku w Unity, kadry z headless renderera,
kontaktówki wariantów assetów.

Ogólna zasada: **artefakt dowodowy, który przy awarii nadal wygląda wiarygodnie, jest
groźniejszy niż brak artefaktu.** Brak renderu widać od razu; render pokazujący nie ten
obiekt co trzeba zostaje zatwierdzony. Każdy generator takich artefaktów potrzebuje
niezależnej kontroli, że pokazuje to, co obiecuje w nazwie pliku - najtaniej przez
porównanie z liczbą, którą pipeline i tak już liczy.

Szczególnie dotyczy rzutu prostokątnego, który jest naturalnym wyborem do porównywania
proporcji (te same skale, można mierzyć linijką) i jednocześnie usuwa wskazówkę,
która zdradziłaby nakładanie.

## Related
- [[fbx-binary-overwrite-corrupts-bindposes]]
- Zasada projektowa Timber Tycoon: "sonda musi umieć zawieść" - ten sam problem
  na poziomie testów, tu przeniesiony na poziom materiału dowodowego dla człowieka
