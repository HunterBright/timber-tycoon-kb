---
title: Zanim zaufasz bramce, sprawdz, czy mierzy to, co widac
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-28'
project: Kerf - Sawmill Tycoon
tags:
- pomiar
- kontrola-jakosci
- proceduralne-generowanie
- bramki
- debugowanie
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Zanim zaufasz bramce, sprawdz, czy mierzy to, co widac

## Sytuacja

Reżyser zglosil prosty defekt: "wystaje ten pien ponad korone i wyglada jak kikut".
Zeby naprawic go raz na zawsze, a nie na oko, potrzebna byla LICZBA, ktora da sie
wpisac w automatyczne strojenie. Napisanie tej liczby zajelo CZTERY podejscia.
Kazde z trzech pierwszych bylo zielone albo czerwone z niewlasciwego powodu.

## Cztery wersje tego samego miernika

**1. "Ile kepek listowia siega ponad czubek pnia."**
Bramka pokazywala 30, 60, 100 kepek - a pien na renderze nadal bylo widac. Kepki
liczone byly OBOK pnia, nie PRZED nim. Miernik mierzyl obecnosc listowia w okolicy,
a nie zaslanianie.

**2. "Ile golego pnia jest w gornych 45% drzewa."**
Ta wersja oblala drzewo, ktore bylo POPRAWNE: akacja parasolowata ma goly pien pod
korona z definicji, wiec dostala 2,24 m "bledu". Miernik nie odrozniał golizny
pozadanej od niepozadanej.

**3. "Najwyzsze drewno przy osi pnia minus gora korony."**
Najsprytniejsza i najgorsza. Naprawa polegala na zadzieraniu gornych galezi do gory -
ale stromy konar TEZ lezy przy osi, wiec kazda poprawka podnosila zmierzony "czubek"
razem z korona. Bramka nie mogla zazielenic sie NIGDY, niezaleznie od tego, jak dobrze
wygladalo drzewo. Automat kręcil sie w kolko przez kilkanascie iteracji.

**4. Dziala: wysokosc czubka PNIA (z jego wlasnej definicji, nie z geometrii)
minus najwyzszy rog kartki listowia.**
Zadnej dwuznacznosci: pien to pien, listowie to listowie.

## Regula

Kiedy miernik ma sterowac automatyczna naprawa, sprawdz go w trzech miejscach:

1. **Na przypadku ZLYM** - czy pokazuje defekt (to sprawdza kazdy).
2. **Na przypadku DOBRYM, ale nietypowym** - czy nie oblewa czegos, co jest
   poprawne z innego powodu (wersja 2 tu polegla).
3. **W SPRZEZENIU z naprawa** - czy poprawa sytuacji faktycznie obniza wynik.
   Miernik, ktory rosnie razem z lekarstwem, zamienia petle strojaca w blednik
   (wersja 3).

Punkt 3 jest tym, o ktorym sie zapomina, i jedynym, ktorego nie widac na pojedynczym
przypadku testowym - ujawnia sie dopiero, gdy miernik zamyka petle.

## Sygnal ostrzegawczy

Petla strojaca, ktora wykonuje wszystkie dozwolone kroki i konczy z tym samym wynikiem,
prawie nigdy nie znaczy "za slabe lekarstwo". Znaczy "miernik porusza sie razem
z lekarstwem". Zanim dolozysz kroki albo poszerzysz zakresy - policz recznie, co
miernik zwraca przed i po JEDNYM kroku naprawy.

## Dodatek: prawdziwa przyczyna okazala sie ogolna

Przy okazji wyszlo, ze defekt nie dotyczyl jednego gatunku. Zarys korony zaokraglonej
przycinal najwyzsza galaz do 12% dlugosci - kilkunastu centymetrow - wiec zaden
mechanizm nie mial czym zakryc konca pnia. Lekarstwem byla PODLOGA na ten zarys,
z wartoscia domyslna 0 (czyli bez zmian dla wszystkiego, co juz zatwierdzone).
Defekt zglaszany jako "problem z tym jednym modelem" byl wlasciwoscia calego
generatora.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260727-0510-rozproszenie-kartek-punkty-zaczepienia|Rozsypywanie kartek po szkielecie: licz NOSNIKI, nie kartki]] - wspolne: proceduralne-generowanie, pomiar
- [[20260728-1320-mniej-wiekszych-kartek|Polowa kartek o rozmiarze wiekszym o jedna trzecia wyglada tak samo]] - wspolne: proceduralne-generowanie, pomiar
- [[20260727-1422-bramka-musi-umiec-zaliczyc-nie-tylko-oblac|Bramka musi mieć udowodniony tryb ZALICZENIA, nie tylko PORAŻKI]] - wspolne: kontrola-jakosci, bramki
- [[20260728-1500-bramka-ponad-sufitem|Prog bramki ponad zmierzonym sufitem zamienia kazda runde w porazke]] - wspolne: bramki, pomiar
- [[20260801-0500-gestszy-pomiar-odslania-dlug|20260801-0500-gestszy-pomiar-odslania-dlug]] - wspolne: bramki, pomiar
- [[20260728-1900-miara-z-degeneracja|Miara optymalizowana samotnie znajduje rozwiazanie zdegenerowane]] - wspolne: bramki, pomiar
<!-- /POWIAZANE:auto -->
