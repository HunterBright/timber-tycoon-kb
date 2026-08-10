---
title: Malowanie szyby kolorem na powierzchni z loftu nigdy nie da płaskiej tafli
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-07-20'
project: Kerf - Sawmill Tycoon
tags:
- blender
- proceduralne-modelowanie
- low-poly
- loft
- pojazdy
- vertex-color
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Malowanie szyby kolorem na powierzchni z loftu nigdy nie da płaskiej tafli

## Co próbowaliśmy zrobić

Proceduralny model auta low-poly w Blenderze: przekrój poprzeczny jako **jeden zamknięty
pierścień** od podłogi po dach, mostkowany wzdłuż auta (loft o stałej topologii). Szyby,
słupki i lakier nadawane **kolorem per ścianka**, na podstawie numeru paska pierścienia
i położenia środka ścianki.

Zaleta była realna: bryła jest szczelna z definicji (zero krawędzi brzegowych), więc
znika cała klasa prześwitów.

## Dlaczego to nie zadziałało

Przekrój auta jest gładką krzywą. Szyba namalowana na takiej powierzchni **jest wygięta,
bo powierzchnia jest wygięta**. Żadne przesuwanie granic koloru tego nie naprawi.

Objawy, które gonimy w kółko i które są tylko skutkami, nie przyczyną:
- okna "opływają" narożnik dachu i wyglądają jak rozlane
- granice słupków mają schodki (klasyfikacja per ścianka na ukośnej linii)
- słupki wychodzą grube, bo są fasetami zakrzywionej powierzchni
- przednia szyba wychodzi w kolorze lakieru, bo w przekroju jest **górną
  powierzchnią**, a nie bokiem

Zrobiliśmy **trzy rundy poprawek kolorowania** (progi wysokości, granice po stacjach,
odcinki opisujące słupki) zanim stało się jasne, że to wada konstrukcyjna, a nie
kosmetyczna. To jest koszt tej pomyłki.

## Co robić zamiast tego

Rozbić bryłę na **dwie przenikające się bryły zamknięte**:

1. **Dolna** - loft jak dotąd, ale zamknięty płaskim pokładem na wysokości linii pasa.
2. **Kabina** - osobna bryła o przekroju prawie prostokątnym: płaskie pionowe boki,
   płaski dach, mała faza na krawędzi.

Zasada "przenikające się bryły zamknięte nigdy nie dają prześwitu" zostaje zachowana,
więc nie tracimy nic z pierwotnej zalety.

**Kluczowa własność, którą warto zapamiętać osobno:**

> Jeśli wszystkie pierścienie kabiny mają ten sam profil (y, z) i różnią się WYŁĄCZNIE
> przesunięciem w X zależnym od z, to każdy czworokąt boku jest płaski niezależnie od
> pochylenia pierścienia.

Dowód: dwa narożniki czworokąta mają identyczne (y, z) jak dwa pozostałe, więc czworokąt
leży w płaszczyźnie rozpiętej przez oś X i kierunek (0, dy, dz).

Skutek praktyczny: **każdy pierścień może mieć własny kąt pochylenia**, a słupki A, B i C
powstają jako prawdziwe wąskie kliny z geometrii. Kolor wynika wtedy z pary
(numer pasma, numer paska), czyli z prawdziwych krawędzi siatki - **schodki stają się
niemożliwe z definicji**, zamiast być zwalczane progami.

Skrajny pierścień pochylony pod kątem szyby, zamknięty wachlarzem, daje **płaską
pochyloną taflę**. Wpust 1.2 cm w X tworzy ramkę i szyba czyta się jako osadzona
w gumie, nie namalowana.

## Sygnał ostrzegawczy

Jeśli po raz **drugi** przesuwasz granice kolorów, żeby naprawić kształt czegoś, co ma być
płaskie - przestań i sprawdź, czy powierzchnia pod spodem w ogóle jest płaska. Kolor nie
zmienia geometrii.

## Jak to udowodnić maszynowo

Nie na oko. Wyznacz płaszczyznę pierścienia metodą Newella i zmierz maksymalne odchylenie
punktów od niej. Przy poprawnej konstrukcji wychodzi **dokładnie 0.0**, nie "prawie zero".
To zamienia "szyby wyglądają krzywo" w liczbę, o którą można się spierać.

## Efekt uboczny, którego nie zakładaliśmy

Nowa konstrukcja wyszła **o 19 procent lżejsza** (6216 zamiast 7704 trójkątów) i pozwoliła
usunąć około 60 linii najbardziej kruchego kodu (klasyfikacja słupków przez odległość od
odcinków). Prostsza konstrukcja okazała się i tańsza, i ładniejsza.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260809-2140-metakule-i-remesh-to-technika-bazowa-nie-wykonczeniowa|Metakule plus remesh wokselowy to technika BRYLY BAZOWEJ, nie wykonczeniowa - do postaci uzywaj loftu z funkcja ksztaltujaca]] - wspolne: loft, proceduralne-modelowanie, low-poly
- [[20260809-2320-adr-siatki-postaci-z-generatora-nie-proceduralnie|ADR - siatki postaci bierzemy z generatora, proceduralnie robimy wszystko PO siatce]] - wspolne: loft, proceduralne-modelowanie
- [[20260628-1105-lowpoly-lake-shore-jagged-fix|Low-poly lake shore looks jagged (serrated) - submerge the rim + widen the water, don't densify]] - wspolne: vertex-color, low-poly
- [[vertex-color-gamma-correction-blender-to-unity|Vertex Color Gamma Correction Blender → Unity]] - wspolne: vertex-color, blender
- [[20260702-1135-lowpoly-thin-planar-leaves-antipattern|Anty-pattern: cienkie płaskie „soczewki" jako liście low-poly]] - wspolne: low-poly, blender
- [[20260711-1647-blender-prop-contact-interpenetrate-not-gap|Anty-wzorzec: szczeliny powietrza jako ochrona przed z-fightingiem (lewitujące propy)]] - wspolne: low-poly, blender
<!-- /POWIAZANE:auto -->
