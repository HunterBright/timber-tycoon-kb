---
title: Grafikę pod cudzy interfejs oceniaj w jego ramce, nie w pełnym kadrze
type: pattern
status: draft
confidence: high
verified: ''
date: 2026-08-13
project: Discord_Studio (MGDB Studio)
tags:
- branding
- asset-pipeline
- ux
- podglad
applies_to: []
source: ''
suggested-category: workflow/asset-pipeline
---

# Grafikę pod cudzy interfejs oceniaj w jego ramce, nie w pełnym kadrze

## When to use
Za każdym razem, gdy asset trafia do interfejsu, nad którym nie masz władzy: baner serwera
Discorda, tło zaproszenia, grafika sklepowa Steam, ikona aplikacji, miniatura na YouTube,
okładka w launcherze. Wspólna cecha: **platforma coś na to nałoży, coś przytnie i pokaże
w rozmiarze innym niż plik**.

## Steps
1. Zrób wariant w pełnej rozdzielczości, jak zwykle.
2. Zbuduj arkusz z **podglądem w docelowej ramce**, a nie tylko dużym kadrem:
   - przeskaluj do rzeczywistego rozmiaru wyświetlania (baner Discorda w pasku kanałów
     to ok. 240 px szerokości, nie 960),
   - narysuj to, co platforma nałoży (gradient + nazwa serwera w dolnym lewym rogu),
   - narysuj prostokąt w miejscu elementu, który zasłania kadr (okno zaproszenia zajmuje
     środek, ok. 26% szerokości i 46% wysokości).
3. Werdykt wydawaj z tego arkusza. Duży kadr służy tylko do oglądania detalu.

## Why this works
Kompozycja, która w pełnym kadrze wygląda na wyważoną, po nałożeniu ramki bywa martwa albo
zasłonięta. W tej sesji podgląd wyłapał to od razu: logotyp postawiony na 26% wysokości tła
zaproszenia wyglądał świetnie w kadrze, a okno zaproszenia wchodziło mu na napis STUDIO.
Poprawka to jedna liczba (0,26 → 0,16), ale bez podglądu wyszłaby dopiero po wgraniu na
żywy serwer — czyli po rundzie „nie podoba mi się, ale nie wiem czemu".

## Najdroższa pułapka: ten sam plik, dwa proporcje ekranu

Grafika 16:9 pokazana na telefonie w pionie jest **przycinana do środka**, a nie skalowana.
Ile zostaje, liczy się jednym działaniem: `proporcja_ekranu / proporcja_pliku`. Dla telefonu
390×844 i pliku 16:9 wychodzi **26% szerokości** — pas x od 0,37 do 0,63. Wszystko poza nim
na telefonie nie istnieje.

W tej sesji to przewróciło ranking: dwa z trzech wariantów tła zaproszenia miały znak
w rogu i na telefonie pokazywały **puste ciemne pole**, a trzeci gubił ostatnią literę
logotypu. Dopiero czwarty wariant, z logotypem zmieszczonym w tym pasie, działał na obu
urządzeniach. Na pełnym kadrze wszystkie trzy wyglądały dobrze.

Wniosek do przeniesienia: **licz pas przetrwania, zanim ustawisz kompozycję**, i rysuj go
na podglądzie. Dotyczy też okładek Steam, miniatur i grafik OG w mediach społecznościowych.

## Trade-offs
Arkusz to dodatkowy skrypt i kilkanaście minut roboty. Zwraca się przy drugim wariancie
i przy każdej kolejnej rundzie poprawek, bo rośnie razem z zestawem — ale przy pojedynczej
grafice bez alternatyw bywa przerostem formy.

Ramka jest odtworzona z pamięci i obserwacji, nie z dokumentacji platformy — to
**przybliżenie**, dobre do oceny kompozycji, złe do liczenia pikseli. Jeśli coś stoi
dokładnie na granicy, sprawdź na żywym koncie.

## Variants
- **Zestaw rozmiarów zamiast jednego**: przy ikonach rządek 160/128/64/48/32 px pokazuje,
  w którym momencie znak przestaje być czytelny (użyte wcześniej przy awatarze MGDB).
- **Podgląd w obu motywach**, gdy platforma ma jasny i ciemny interfejs.
- **Bezpieczne pole zamiast makiety**, gdy elementów nakładanych jest dużo: wystarczy
  zaznaczyć obszar, w którym nic ważnego nie może leżeć.

## Related
- [[20260813-0730-wycinanie-litery-z-rastra-po-barwie-nie-po-przerwie]]
