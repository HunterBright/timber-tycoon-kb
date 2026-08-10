---
type: decision
project: Kerf - Sawmill Tycoon
suggested-category: engine/decisions
tags: [npc, postacie, low-poly, solver, ubrania, mirror, optymalizacja, adr]
date: 2026-08-01
status: draft
---

# ADR: NPC od zera (symetryczny low-poly, stroj w siatce) zamiast solvera warstw ubran

## WERDYKT KONCOWY (Hunter, 2026-08-01, po obejrzeniu prototypu):
## "Porzucamy ten pomysl. Pracujemy nad starym modelem." - DECYZJA COFNIETA.
Prototyp (924 tri, 4 bramki z dowodami, ~1 h pracy, budowy w sekundy)
zostal zbudowany i pokazany - i NIE przeszedl oka rezysera. Wartosc
lekcji: TANI PROTOTYP JAKO BRAMKA DECYZYJNA zadzialal wzorowo - godzina
pracy zweryfikowala kierunek PRZED tygodniami przesiadki (suwaki,
warianty, eksport). Skrypt budowa_npc_v2.py zostaje uspiony w repo
(commity bb15698, c90a49c). Reszta wpisu = stan wiedzy sprzed werdyktu.

## Decyzja pierwotna (Hunter, 2026-08-01 rano: "Zapiszmy ten model co mamy i lecimy z C")
Porzucamy podejscie "ubrania jako fizyczne warstwy kopiowane ze scianek
gestego, recznie rzezbionego (asymetrycznego) ciala + solver dopasowania".
Nowy NPC: budowany od zera skryptem, SYMETRYCZNY (mirror od pierwszego
wierzcholka), ~2-3k tri, stroj WBUDOWANY w siatke (wytloczenia + regiony
materialow), prosta twarz w stylu gry (wzorzec Schedule 1), skinning do
ISTNIEJACEGO szkieletu (kosci nieruszane). Stary generator zarchiwizowany
(tag lph-generator-v16d-final + _archiwum_generator_v16d_2026-08-01).

## Kontekst
Solver warstw pochlonal wiele sesji (prasowanie, prostowanie, sondy
przebic/brzegu/wysp/odstepu, pomiar srodkow, fantomy ramy pomiaru).
Kazdy mechanizm byl uzasadniony i dzialal, ale klasa problemow byla
niewyczerpywalna: pchniecia dopasowujace warstwy MARSZCZA gesta siatke,
a asymetria bazy zamienia kazda krawedz w zygzak do prostowania.
Werdykt oka (Hunter): pas/krocze "strasznie poszarpane" mimo 18 budow
w jedna noc i zielonych bramek.

## Przeslanki
1. Gesta siatka nie kupuje jakosci w stylu low-poly - kupuje faldy.
   Styl gry (Schedule 1) to duze plaskie scianki i ostre sylwetki.
2. Fizyczne warstwy ubran maja sens przy symulacji; w stylizowanym
   low-poly stroj wbudowany w siatke daje ten sam odczyt bez calej
   klasy przebic/marginesow/przenikania w animacji.
3. Symetria musi byc od PIERWSZEGO wierzcholka - symetryzacja recznej
   rzezby po fakcie niszczy prace autora i nie usuwa przyczyny.
4. Umowa z wczesniejszego planu brzmiala: "jesli po wygladzeniu nadal
   zlepek - rozmowa o glebszej przebudowie". Warunek sie spelnil.

## Konsekwencje
- Przepada: reczna twarz i chirurgia nog z bazy (zaakceptowane przez
  autora), solver warstw jako kod aktywny.
- Zostaje: know-how bramek/dzwigni/pomiarow (przenosi sie), szkielet
  i animacje, skala i proporcje (mierzone ze starej bazy jako referencja).
- Prototyp jako bramka decyzyjna: baza + stroj robotnika + twarz ->
  sedzia + werdykt oka PRZED pelnym pakietem (suwaki, warianty).

## Lekcja przenosna
Gdy utrzymanie zbieznosci ukladu wymaga coraz wiekszej liczby
wyjatkow/stref/progow, a werdykt oka dalej mowi "poszarpane" - to sygnal,
ze walczysz z ARCHITEKTURA, nie z bugami. Zmien reprezentacje problemu
(tu: warstwy fizyczne -> stroj w siatce), zamiast dokladac mechanizmy.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260809-2320-adr-siatki-postaci-z-generatora-nie-proceduralnie|ADR - siatki postaci bierzemy z generatora, proceduralnie robimy wszystko PO siatce]] - wspolne: adr, postacie, npc
- [[20260801-0500-gestszy-pomiar-odslania-dlug|20260801-0500-gestszy-pomiar-odslania-dlug]] - wspolne: solver, ubrania
- [[20260731-1050-rowne-krawedzie-ubran-bisect-plane|20260731-1050-rowne-krawedzie-ubran-bisect-plane]] - wspolne: ubrania, low-poly
- [[20260809-2140-metakule-i-remesh-to-technika-bazowa-nie-wykonczeniowa|Metakule plus remesh wokselowy to technika BRYLY BAZOWEJ, nie wykonczeniowa - do postaci uzywaj loftu z funkcja ksztaltujaca]] - wspolne: postacie, low-poly
<!-- /POWIAZANE:auto -->
