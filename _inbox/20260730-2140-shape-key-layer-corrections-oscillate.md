---
title: Poprawki anty-przebiciowe w danych POJEDYNCZEGO klucza kształtu oscylują
type: anti-pattern
status: draft
confidence: low
verified: ''
date: '2026-07-30'
project: Kerf - Sawmill Tycoon
tags:
- blender
- shape-keys
- clothing
- layers
- sliders
- convergence
- oscillation
applies_to: []
source: ''
suggested-category: blender/lessons
---

# Poprawki anty-przebiciowe w danych POJEDYNCZEGO klucza kształtu oscylują

## Kontekst

Warstwy ubrań proxy (ciało → koszula → kamizelka) z kluczami kształtu
(suwaki sylwetki) kopiowanymi z ciała. Na skrajach suwaków warstwy
rozjeżdżają się o kilka mm (delty interpolowane z różnych trójkątów
źródłowych), więc po zbudowaniu kluczy trzeba per-stan dopchnąć warstwę
zewnętrzną nad wewnętrzną.

## Anti-pattern

Naprawa przez zapis do danych klucza tego stanu:
`klucz.data[i].co += d / waga_stanu` — NIE ZBIEGA przy suwakach
dwustronnych (min < 0 < max):

1. Pozycja w stanie w to `baza + delta*w`. Poprawka delty naprawiająca
   skraj +1 przesuwa skraj -1 w PRZECIWNĄ stronę o tę samą wielkość.
   Sumarycznie na parze skrajów nie da się zyskać - suma poprawek zeruje się.
2. Iterowanie przejść (nawet z tłumieniem 0.8) daje oscylację:
   5335 → 5165 poprawek/przejście, klucze rozniesione o kilkanaście mm.
3. Suwak z małym zakresem (np. max 0.4) MNOŻY szkody: poprawka /0.4 jest
   2.5x większa w delcie, a jej odbicie na drugim skraju jeszcze większe.

## Rozwiązanie

Przesunięcie JEDNOLITE: `baza[i] += d` ORAZ `każdy_klucz.data[i] += d`
(łącznie z Basis). Działa na każdym stanie identycznie, więc nie ma jak
odbić; iterować przejścia aż liczba poprawek spadnie do zera (relacje,
w których nic się nie ruszyło od poprzedniego przejścia, można pomijać).
Koszt: ubranie robi się miejscami odrobinę luźniejsze we WSZYSTKICH
stanach (to samo robi prawdziwa odzież warstwowa).

## Pułapka pomiarowa obok (ta sama sesja)

Solidify tworzy na brzegu ubrania wąski RANT (pasek łączący powłokę
zewnętrzną z wewnętrzną, normalna wzdłuż powierzchni, np. w górę na
krawędzi spodni). Rąbek koszuli wiszący tuż PONIŻEJ krawędzi spodni
mierzony względem rantu wygląda jak zanurzony o kilka mm — fałszywe
poprawki szarpią rąbek w górę. Filtr ścianek źródłowych: zostają tylko
te z `dot(kierunek_od_ciała, normalna) > 0.3` (było `>= 0`).
