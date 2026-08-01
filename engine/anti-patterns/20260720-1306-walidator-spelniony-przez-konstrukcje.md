---
title: Walidator, ktory jest spelniony automatycznie przez konstrukcje
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-07-20'
project: Kerf - Sawmill Tycoon
tags:
- validation
- testing
- geometry
- procedural-generation
- false-confidence
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Walidator, ktory jest spelniony automatycznie przez konstrukcje

## The trap

Piszesz proceduralny generator geometrii i dokladasz walidator, ktory ma pilnowac kluczowej
wlasnosci wyniku. Walidator swieci na zielono, liczba wyglada imponujaco (u nas: "odchylenie
od plaszczyzny = 0.000e+00"), wiec traktujesz ja jako dowod. W dokumentacji zapisujesz:
"wymog jest teraz LICZBA, nie opinia".

Pulapka polega na tym, ze mierzona wielkosc wynika WPROST ze sposobu, w jaki generujesz
geometrie - i nie da sie jej zlamac zadna zmiana danych wejsciowych.

## Why it fails

Konkret. Pierscienie kabiny auta budowane byly wzorem:

    x = x_base - (z - z_ref) * s

Kazdy punkt takiego pierscienia spelnia `x + s*z = const`, czyli **lezy w plaszczyznie z
definicji** - dla dowolnego profilu i dowolnego pochylenia. Check "czy pierscien jest plaski"
nie mogl wiec zawiesc nigdy, poza jednym egzotycznym przypadkiem (czlon x zalezny od punktu),
ktorego nikt nie zamierzal wprowadzic.

Tymczasem wlasnosc, na ktorej NAPRAWDE zalezalo (plaskie szyby), zalezy od czegos innego:
czy OBA pierscienie danego pasma maja ten sam profil (y, z). Bok miedzy pierscieniami o
roznych profilach jest wykrzywiony - i tego nie pilnowal nikt.

Efekt: system mial strażnika w miejscu, gdzie nic nie moglo pojsc nie tak, i ani jednego
tam, gdzie mogla wejsc cicha regresja psujaca kluczowa ceche modelu.

## Symptoms

- Walidator NIGDY nie byl czerwony, odkad powstal, i nikt nie potrafi opisac zmiany, ktora
  by go zapalila.
- Zwracana wartosc to dokladne zero albo dokladna stala, a nie liczba z szumem numerycznym.
  **Dokladne zero to sygnal ostrzegawczy, nie powod do dumy** - oznacza zwykle tozsamosc
  algebraiczna, nie zmierzona wlasnosc.
- Check mierzy pojedynczy element (pierscien, wierzcholek, obiekt), podczas gdy wlasnosc,
  o ktora chodzi, dotyczy RELACJI miedzy elementami (pasmo, krawedz, para).
- Opis checku w komentarzu mowi o skutku ("zeby szyby byly plaskie"), a kod mierzy przyczyne
  posrednia, ktora z tym skutkiem nie jest rownowazna.

## Correct approach

1. **Kazdy walidator musi miec udowodniona dzwignie, ktora go lamie.** Nie "wyobrazam sobie,
   ze moglby zawiesc" - realny, uruchamialny samotest, ktory psuje wejscie i sprawdza, ze
   check faktycznie krzyczy. U nas: `selftest_validators.py`, 11 dzwigni + kontrola
   kalibracji (nietkniety model MUSI przejsc, inaczej test moglby byc czerwony zawsze).
2. **Pytaj, na jakim POZIOMIE lezy wlasnosc.** Jesli zalezy Ci na plaskosci powierzchni
   miedzy elementami, mierz powierzchnie miedzy elementami, a nie same elementy.
3. **Traktuj dokladne zero jako podejrzane.** Sprawdz algebraicznie, czy wynik nie jest
   tozsamoscia. Jesli jest - check jest dekoracja.
4. Przy okazji sprawdz POKRYCIE checku, nie tylko jego wynik. U nas druga wersja tego samego
   walidatora probkowala cztery wysokosci wpisane na sztywno, przez co jeden pierscien nie
   byl sprawdzany ANI RAZU - i mogl przebijac sasiada bez sygnalu.

## Related

- [[20260720-1308-pula-jednoelementowa-udaje-pelne-pokrycie]] - ten sam gatunek falszywej pewnosci, tylko
  po stronie doboru probki zamiast po stronie mierzonej wielkosci.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260727-1535-gates-must-not-identify-parts-by-world-coordinate|A geometry gate that identifies body parts by raw world coordinate is a gate on credit]] - wspolne: geometry, testing
- [[20260728-1105-samotest-sprawdzajacy-wlasne-normalne-jest-slepy|Samotest sprawdzajacy WLASNE normalne jest slepy na odwrocona scianke]] - wspolne: geometry, testing
<!-- /POWIAZANE:auto -->
