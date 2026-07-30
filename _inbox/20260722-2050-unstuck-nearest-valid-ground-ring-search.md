---
title: 'Unstuck / reset: szukaj najbliższego POPRAWNEGO gruntu zamiast teleportu do bazy'
type: pattern
status: draft
confidence: low
verified: ''
date: '2026-07-22'
project: Kerf - Sawmill Tycoon
tags:
- unity
- physics
- vehicle
- respawn
- unstuck
- raycast
- level-design
applies_to: []
source: ''
suggested-category: engine/patterns
---

# Unstuck / reset: szukaj najbliższego POPRAWNEGO gruntu zamiast teleportu do bazy

## When to use
Gracz może wjechać pojazdem (albo wejść postacią) w miejsce, z którego nie ma wyjścia:
woda, rów, rozpadlina, szczelina między skałami. Potrzebny przycisk "utknąłem".

Trzy typowe podejścia i kiedy który:
- **Reset w miejscu** (prostuje, zeruje prędkość) - ratuje TYLKO z dachowania. W wodzie
  i w rowie nie zmienia nic, bo "grunt pode mną" to dokładnie to dno, na którym siedzę.
- **Teleport do stałego punktu** (baza/spawn) - zawsze działa, ale w grach z transportem
  ładunku jest darmowym skrótem powrotnym i psuje ekonomię.
- **Najbliższy poprawny grunt** (ten wzorzec) - ratuje ze wszystkiego i nie skraca drogi.

## Steps
1. Pierścienie o rosnącym promieniu wokół obiektu; **pierwszy pierścień ma promień 0**,
   czyli własną pozycję. Dzięki temu na poprawnym gruncie funkcja nie rusza obiektem
   ani o metr - zero regresji dla starego zastosowania (dachowanie).
2. Liczba próbek na pierścień proporcjonalna do obwodu, obcięta do widełek (np. 8..24),
   plus przesunięcie kąta per pierścień, żeby próbki się nie pokrywały.
3. Kandydat przechodzi dopiero, gdy spełnia WSZYSTKIE warunki obrysu pojazdu:
   - twardy grunt pod środkiem i czterema rogami obrysu (maska "po czym da się jeździć"),
   - nachylenie każdego trafienia w limicie (inaczej obiekt zjedzie z powrotem),
   - rozrzut wysokości rogów w limicie (odsiewa krawędź klifu i stopnie),
   - brak wody nad gruntem (patrz gotcha),
   - wolne miejsce na nadwozie.
4. Pierwszy kandydat wygrywa - to automatycznie najbliższy.
5. Fallback do starego zachowania, gdy w zasięgu nic nie ma. Nigdy nie rzucaj obiektem
   w losowe miejsce.
6. Obrys pojazdu bierz z DANYCH RIGU (rozstaw kół), nie ze stałych - inny pojazd, inny rozmiar.

## Why this works
Warunek "najbliższy" jest zaszyty w kolejności przeszukiwania, więc nie trzeba liczyć
odległości ani sortować. Warunek "poprawny" jest sprawdzany na obrysie, a nie w punkcie -
punkt zawsze przejdzie na krawędzi klifu, obrys nie.

Sprawdzanie przeszkód **kulami uniesionymi nad grunt**, nie pudłem obrysu: pudło na stoku
zawsze zahacza o teren i odsiewałoby wszystkie poprawne brzegi. Dwie kule (przód i tył)
o promieniu połowy szerokości pokrywają pojazd i nie łapią stoku do ~30 stopni.

## Trade-offs
- Kilkaset raycastów w najgorszym razie (środek dużego jeziora). Nieistotne, bo leci
  raz na wciśnięcie klawisza, a typowo kończy się na pierwszym pierścieniu.
- Wynik zależy od jakości kolizji terenu. Cienka siatka bez collidera = brak kandydatów.
- Nie zastępuje projektowania mapy: jeśli brzeg jest wszędzie pionowy, ratunek wyrzuci
  gracza dalej, niż się spodziewa.

## Variants
- Dla postaci pieszej wystarczy jeden promień i mniejszy obrys.
- Zamiast pierścieni można iść wzdłuż wektora "na zewnątrz wody", ale wymaga to znajomości
  geometrii akwenu i nie ratuje z rowu ani z rozpadliny.

## Gotcha: fizyka nie widzi teleportu
Przy `m_AutoSyncTransforms: 0` (domyślne w nowszych projektach Unity) po przestawieniu
transformu **trzeba zawołać `Physics.SyncTransforms()`**, zanim cokolwiek się o to miejsce
zapyta promieniem. Inaczej test sprawdza STARĄ pozycję i świeci na zielono bez związku
z rzeczywistością.
