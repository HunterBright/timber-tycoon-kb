---
type: anti-pattern
project: Kerf - Sawmill Tycoon
suggested-category: engine/anti-patterns
tags: [proceduralne-generowanie, api-narzedzi, suwaki, blender, ux-narzedziowy]
date: 2026-07-27
status: draft
---

# Jeden suwak sterujacy dwiema roznymi rzeczami

## Co sie stalo

W generatorze drzew suwak "splaszczenie" robil naraz dwie rzeczy: kladl kepki
listowia poziomo ORAZ zgniatal same kartki (czworokaty) wzdluz osi pionowej.

Gdy trzeba bylo zrobic drzewo o plaskim wierzchu (akacja parasolowata), jedynym
dostepnym narzedziem bylo podkrecenie tego suwaka do 0,85. Wynik: sylwetka faktycznie
sie splaszczyla, ale kartki zamienily sie w cieniutkie paski i cale drzewo czytalo
sie jak **zaluzje na maszcie**. Odwrotu nie bylo, bo obu efektow nie dalo sie
rozdzielic.

Gorzej: przez to, ze suwak "cos robil", przez kilka iteracji wygladalo na to, ze
mechanizm plaskiej korony istnieje. Naprawde nie istnial - brakowalo zupelnie innej
funkcji (wyrownania dlugosci galezi tak, zeby wszystkie konczyly sie na jednej
wysokosci).

## Dlaczego to jest pulapka, a nie tylko niewygoda

1. **Zaslania brak funkcji.** Suwak dajacy CZESC pozadanego efektu wyglada jak
   wlasciwe narzedzie i odsuwa moment, w ktorym zauwazysz, ze potrzebna funkcja
   w ogole nie zostala napisana.
2. **Nie da sie go rozdzielic pozniej bez ryzyka.** Kazdy gotowy zestaw, ktory uzywal
   posredniej wartosci, zalezy juz od OBU efektow naraz.
3. **Pomiary klamia.** Miernik "jak plaska jest korona" reagowal na suwak, wiec
   automatyczne strojenie chetnie go podkrecalo - i psulo kartki, ktorych zaden
   miernik nie pilnowal.

## Regula

Jeden suwak = jedna rzecz w swiecie. Jesli parametr steruje dwiema rzeczami, ktore
kiedykolwiek trzeba bedzie ustawic niezaleznie, rozdziel go **zanim** ktokolwiek
zapisze na nim zestaw.

Gdy rozdzielasz suwak juz uzywany: nowy parametr musi miec wartosc domyslna, przy
ktorej zachowanie jest **identyczne co do liczby** z poprzednim. Po zmianie przelicz
wszystkie zatwierdzone zestawy i porownaj twarda metryke (u nas: liczba trojkatow
i wysokosc co do drugiego miejsca po przecinku). Bez tego "kompatybilnosc wsteczna"
jest tylko deklaracja.

## Sygnal ostrzegawczy do wylapania wczesniej

Kiedy tlumaczysz komus dzialanie suwaka i w opisie pojawia sie spojnik "oraz"
odnoszacy sie do dwoch roznych obiektow ("klada kepki poziomo ORAZ zgniata kartki") -
to jest ten moment. Opis parametru, ktory potrzebuje dwoch zdan o dwoch roznych
rzeczach, opisuje dwa parametry.
