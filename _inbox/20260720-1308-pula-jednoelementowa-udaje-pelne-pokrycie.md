---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [testing, smoke-test, coverage, flaky-tests, random-sampling]
severity: high
time_lost: "0 h teraz, ale bez tego kazdy przyszly zly eksport modelu przechodzilby w wiekszosci buildow"
date: 2026-07-20
status: draft
applies_to: [unity, ci, smoke-tests]
---

# Test losujacy jeden element z puli o rozmiarze 1 udaje pelne pokrycie

## Problem

Sonda build-smoke sprawdzala auto NPC: nazwy i liczbe kol, promienie, kule kolizji, tarcie,
cienie, lakier. Osiem warunkow, wszystkie zielone od miesiecy, check uznany za solidny.

Sonda robila to tak, ze wolala kod spawnu gry, ktory wybiera model **losowo z puli**, i
sprawdzala powstaly egzemplarz. Pula miala jeden wpis.

W momencie dodania drugiego modelu ten sam kod przestalby byc bramka, a stalby sie loteria:
zly eksport przechodzilby w polowie uruchomien, przy trzech modelach w dwoch na trzy. Zielone
byloby wynikiem WIEKSZOSCIOWYM, a awaria wygladalaby jak losowy szum - najgorszy rodzaj
bledu, bo zniechecajacy do szukania przyczyny.

## Root cause

**Losowanie z jednoelementowego zbioru jest nieodroznialne od pelnego pokrycia.** Dopoki
rozmiar puli wynosil 1, "sprawdz losowy element" i "sprawdz wszystkie elementy" dawaly
identyczny wynik przy kazdym uruchomieniu, wiec nic nie sygnalizowalo, ze test mierzy probke,
a nie calosc.

Poglebiajace to dwie rzeczy:
- Test wolal produkcyjna sciezke spawnu (slusznie - to ona ma byc sprawdzana), a ta zawiera
  losowanie. Wierne odwzorowanie produkcji **wciagnelo do testu niedeterminizm produkcji**.
- Test nigdy nie czytal zrodla puli, wiec nie mial nawet z czym porownac liczby sprawdzonych
  elementow.

## Solution

1. **Rozdzielic "zmontuj" od "wpnij w gre".** Wydzielilismy z kodu spawnu funkcje budujaca
   sam pojazd (`BuildVehicleRig`), przyjmujaca JAWNIE wskazany model. Kod produkcyjny wola ja
   z `null` (= losuj, bez zmiany zachowania), test wola ja per element puli.
   To bylo konieczne, bo sprawdzane rzeczy (kola, kule kolizji, kontroler) **nie istnieja
   w pliku modelu** - powstaja dopiero przy montazu. Sprawdzanie samego assetu udawaloby test.
2. **Identyfikatory checkow z nazwa elementu**: `NPC/kola:<model>` zamiast `NPC/kola`. Bez
   tego trzy modele daja trzy linie o tej samej nazwie i raport nie mowi, ktory zawiodl.
3. **Osobny check na samo POKRYCIE** (`NPC/pula`), ktory liczy wyemitowane bloki i porownuje
   z rozmiarem puli. Ma wlasna czerwona probke (`-carpoolredproof` pomija ostatni element).
   Bez tego "sprawdzilismy wszystko" byloby twierdzeniem bez trybu porazki.
4. **Drogie testy zostaja pojedyncze, ale DETERMINISTYCZNE.** Test przejazdu trwa 47 s;
   uruchamianie go per model rozdymaloby bramke. Zostal jeden, ale na wskazanym elemencie
   (domyslnie najnowszym, czyli najmniej sprawdzonym), z mozliwoscia wyboru flaga.
   Zostawienie go na losowym tylko PRZESUWALOBY loterie, zamiast ja usunac.
5. Suma checkow ma **rosnac z rozmiarem puli** - to uczciwy sygnal. U nas 95 -> 100.

## What didn't work

Pierwszy pomysl: instancjonowac sam plik modelu i sprawdzac go bez montazu. Odrzucony -
wiekszosc warunkow czyta komponenty dodawane w kodzie przy spawnie, wiec taki test
przeszedlby ZAWSZE, nie badajac niczego z listy. Bylby trzecia odmiana tego samego bledu:
zielono, bo nie ma czego zlamac.

## Transferability

Dotyczy kazdego smoke-testu, ktory pobiera element z kolekcji: pule assetow, listy poziomow,
warianty konfiguracji, zestawy lokalizacji, tablice shaderow. Regula ogolna:

**Jesli test losuje probke, jego wiarygodnosc zalezy od rozmiaru zbioru - a rozmiar zbioru
zmienia sie bez dotykania testu.** Test, ktory byl dowodem przy N=1, staje sie loteria przy
N=2, i nikt nie dostaje o tym powiadomienia. Dlatego kolekcje sprawdza sie w PETLI, a nie
przez probke, a jesli petla jest za droga - probka musi byc deterministyczna i jawnie
udokumentowana jako probka.

## Related

- [[walidator-spelniony-przez-konstrukcje]] - falszywa pewnosc po stronie mierzonej wielkosci.
- Zasada projektowa Timber Tycoon: "test wszystkich X = WSZYSTKIE instancje".
