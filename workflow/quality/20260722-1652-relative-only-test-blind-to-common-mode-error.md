---
title: Test porownujacy instancje MIEDZY SOBA jest slepy na blad wspolny (common-mode)
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-22'
project: Kerf - Sawmill Tycoon
tags:
- testing
- smoke-test
- build-gate
- false-green
- measurement
- unity
applies_to:
- unity
- any-automated-test
source: ''
severity: high
time_lost: ~2h (bug zyl w buildzie mimo zielonej sondy)
promoted: '2026-07-30'
---

# Test porownujacy instancje MIEDZY SOBA jest slepy na blad wspolny (common-mode)

## Problem
Sonda buildu miala sekcje "Stopy NPC": mierzyla wysokosc kosci stop kazdego z 12 modeli
i sprawdzala, czy kazdy miesci sie w 2 cm od modelu-wzorca. Przez tydzien swiecila
12/12 PASS. Gracz mimo to widzial NPC wtopionych w podloge. Pomiar bezwzgledny (podeszwa
vs grunt) pokazal: worker -9,1 cm, sprzedawca -7,3 cm, klient -1,5..-6,5 cm. Wszyscy byli
wtopieni RÓWNO, wiec test "kazdy vs wzorzec" nie mial jak tego zobaczyc - wzorzec tez byl
wtopiony.

## Root cause
Test mierzyl WZGLEDNA zgodnosc wewnatrz populacji, a szukany blad byl PRZESUNIECIEM CALEJ
populacji wzgledem swiata zewnetrznego. Sprawdzian "czy elementy zgadzaja sie ze soba" i
sprawdzian "czy elementy zgadzaja sie z rzeczywistoscia" to dwa rozne twierdzenia; pierwszy
nie implikuje drugiego. Klasyczny common-mode error: wspolne zaburzenie znosi sie w roznicy.

## Solution
Do kazdego testu wzglednego dolozyc PRZYNAJMNIEJ JEDEN pomiar zakotwiczony w niezaleznym
zrodle prawdy, ktore nie jest czescia mierzonego systemu. Tutaj: grunt z silnika fizyki
(raycast w collider podlogi) kontra podeszwa z zeskinowanej siatki (BakeMesh na realnych
wierzcholkach). Zadna z tych liczb nie pochodzila z matematyki offsetow, ktora byla
podejrzana - dlatego check mogl obalic wlasny system.
Weryfikacja, ze check dziala: dzwignia wylaczajaca naprawe (flaga uruchomieniowa) musi
wywolac FAIL z liczbami zgodnymi z tym, co raportowal czlowiek.

## What didn't work
- Zaufanie zielonemu wynikowi sekcji o pasujacej nazwie ("Stopy" brzmi jak "stopy sa OK",
  a znaczylo "stopy modeli zgadzaja sie ze soba").
- Szukanie przyczyny wylacznie w jednej stalej kompensacyjnej - stala tlumaczyla ~6 cm z 9 cm,
  reszta siedziala w geometrii modeli. Bez pomiaru bezwzglednego podzial byl nie do ustalenia.

## Transferability
Dotyczy kazdego zestawu testow, nie tylko Unity: porownania A vs B (snapshot testy, golden
files, "wszystkie warianty spojne", kalibracja czujnikow, porownania wydajnosci wzgledem
baseline) sa strukturalnie slepe na dryf calego zbioru. Pytanie kontrolne przy pisaniu testu:
"jesli WSZYSTKIE mierzone obiekty popsuja sie identycznie, czy ten test zaswieci na czerwono?".
Jesli nie - brakuje kotwicy w zewnetrznym zrodle prawdy.

## Related
- [[gate-must-have-provable-failure-mode]]
- 20260722-1652-npc-foot-grounding-raycast-vs-navmesh-baseoffset.md

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260725-1845-silhouette-measurement-catches-shadow-and-vignette|Pomiar proporcji z renderu klamie trzy razy, zanim zacznie mowic prawde]] - wspolne: false-green, measurement
- [[20260721-1340-stale-artifact-from-crashed-test-run|Crashed test run leaves the PREVIOUS report on disk and reads as "my code never got built"]] - wspolne: build-gate, testing
- [[20260720-1308-pula-jednoelementowa-udaje-pelne-pokrycie|Test losujacy jeden element z puli o rozmiarze 1 udaje pelne pokrycie]] - wspolne: smoke-test, testing
- [[20260710-2250-unity-autonomous-smoke-runner-flag-file|Autonomiczny runner smoke testów w Unity: plik-flaga + plik wyników]] - wspolne: smoke-test, testing
- [[20260714-1245-test-bez-trybu-porazki|Anty-wzorzec: test, ktory nie ma jak zawiesc (silnik "naprawia" mierzona wielkosc)]] - wspolne: false-green, testing
- [[20260727-1535-gates-must-not-identify-parts-by-world-coordinate|A geometry gate that identifies body parts by raw world coordinate is a gate on credit]] - wspolne: false-green, testing
<!-- /POWIAZANE:auto -->
