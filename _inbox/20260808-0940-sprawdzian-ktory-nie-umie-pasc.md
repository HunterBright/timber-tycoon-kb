---
title: Sprawdzian, ktory nie pada pod dzwignia porazki, niczego nie sprawdza
type: lesson
status: draft
confidence: high
verified: 2026-08-08
tags: [testy, bramki-jakosci, narzedzia, metoda]
date: 2026-08-08
project: GameDevOS
source: wlasny pomiar, tools/rozjazd_faktow.py, przebieg 2026-08-08
applies_to: [bramki-jakosci, skrypty-narzedziowe, testy-jednostkowe]
severity: high
time_lost: ok. 20 minut
---

# Sprawdzian, ktory nie pada pod dzwignia porazki, niczego nie sprawdza

## Objaw

Dopisalem do detektora nowa funkcje (waskie wyciszanie pojedynczej wartosci
zamiast calej kategorii) i trzy sprawdziany do niej. Wszystkie trzy przeszly.
Potem uruchomilem **dzwignie trybu porazki**, czyli przelacznik, ktory celowo
przywraca stare, zle zachowanie. Sprawdziany **dalej przechodzily, wszystkie
szesnascie**. Narzedzie zepsute i narzedzie naprawione dawaly identyczny wynik.

## Przyczyna

Sprawdzalem skutek, ktory wychodzi **tak samo przy obu zachowaniach**.

Konkretnie: sprawdzalem, czy po wyciszeniu plik przestaje byc zglaszany.
Przy wyciszeniu waskim przestaje, bo znika jedna wartosc. Przy wyciszeniu
szerokim tez przestaje, bo znika caly plik. **Ten sam wynik z dwoch roznych
powodow**, wiec test nie odrozniał jednego od drugiego.

To jest dokladnie ta sama rodzina bledu co test binarny bez kontroli: mierzy
sie wlasna slepote zamiast swiata.

## Rozwiazanie

Przebudowalem przypadek testowy tak, zeby **oba zachowania dawaly rozne wyniki**.
Zamiast pliku z jedna wartoscia podlozylem plik z **dwiema**: jedna nalezaca do
cudzego narzedzia (wyciszana) i jedna wlasna, przestarzala (ma zostac zgloszona).

- waskie wyciszenie: plik **nadal jest zglaszany**, za swoja wlasna wartosc
- szerokie wyciszenie: plik **znika calkowicie**, czyli sprawdzian pada

Po tej zmianie dzwignia dziala: bez niej 16 na 16, z nia 15 na 16, i pada
dokladnie ta jedna proba, ktora ma paść.

## Co NIE zadzialalo

- **Dopisanie trzeciego sprawdzianu na to samo.** Trzy testy sprawdzajace ten
  sam nieodrozniajacy skutek to nadal zero informacji, tylko trzy razy dluzej.
- **Uznanie, ze skoro dzwignia nic nie lamie, to funkcja jest odporna.** Kusilo,
  ale to jest wnioskowanie odwrotne: dzwignia nie lamala testu, bo test byl slepy,
  a nie dlatego, ze kod byl dobry.

## Dowod

`python tools\rozjazd_faktow.py --zepsuj --sprawdz-sie-samemu` daje
`PORAZKA (1 z 16 prob nie przeszlo)`, a bez `--zepsuj` daje `OK (16 prob)`.
Przed poprawka oba warianty dawaly `OK (16 prob)`.

## Regula do zapamietania

> **Zanim uwierzysz nowemu sprawdzianowi, zepsuj kod, ktory on sprawdza,
> i zobacz, czy pada.** Sprawdzian, ktory przechodzi na kodzie zepsutym,
> nie jest slabym sprawdzianem - nie jest sprawdzianem wcale.

Prosty sposob na zaprojektowanie takiego testu: **wypisz, co dostaniesz przy
zachowaniu prawidlowym i co przy blednym. Jesli to ta sama wartosc, testujesz
nie to co trzeba.**

## Czy to przeniesie sie na inny projekt

Tak, i to bez zmian. Dotyczy kazdej bramki jakosci: testu jednostkowego,
walidatora assetow, sprawdzianu importu FBX, bramki budowania. Jest szczegolnie
grozne przy bramkach pisanych przez agenta, bo agent chetnie dopisuje testy,
ktore przechodza, i rzadko sprawdza, czy potrafia nie przejsc.

## Powiazane

- [[20260807-0830-zwiad-bez-wyszukiwarki-api-i-kanaly-atom]]
