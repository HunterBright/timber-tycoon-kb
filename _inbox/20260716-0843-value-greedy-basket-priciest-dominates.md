---
type: pattern
project: Timber_Tycoon
suggested-category: genre/tycoon
tags: [economy, order-generation, weighted-random, game-balance, tycoon]
date: 2026-07-16
status: draft
---

# Koszyk dobijany do kwoty "krokiem najblizej celu" = najdrozszy produkt dominuje kazde zamowienie

## Problem / kontekst
Generator zamowien NPC: wylosuj kwote docelowa, potem dosypuj po 1 sztuce produktu, ktory
"zbliza sume najblizej do celu". Brzmi neutralnie, ale PONIZEJ celu krok najblizej celu to
ZAWSZE najdrozsza sztuka. Efekt: najdrozszy produkt w puli (u nas worki zrebkow 20$ vs opal
10$) wypelnia kazde zamowienie do swojego limitu, tansze produkty zostaja przy 1 szt.
Gracz widzi "prawie kazde zamowienie to X" natychmiast po odblokowaniu drozszego produktu,
a zapasy tanszych rosna bez zbytu. Problem wraca z KAZDYM nowym najdrozszym produktem.

## Rozwiazanie (pattern)
Rozdziel trzy role, ktore latwo skleic w jedno:
1. **Waga wyboru** produktu - proporcjonalna do realnych zapasow gracza (to, co zebral /
   ma na regalach), NIE do teoretycznej podazy swiata (to dawalo drugiemu bledowi paliwo).
2. **Sklad koszyka** - dopoki zaden krok nie przebija kwoty docelowej, dosypuj sztuke
   LOSOWANA WAZONO tymi samymi wagami co wybor typow (koszyk odzwierciedla zapasy).
3. **Domkniecie koncowki** - dopiero gdy kazdy krok przebija cel (albo suma jest ponizej
   dolnej granicy widelki), pojedynczy krok zachlanny "najblizej celu".
Wyplata pozostaje dokladna suma sztuk x cena (przejrzystosc dla gracza).

## Kiedy stosowac
Kazdy tycoon/shop-sim z zamowieniami generowanymi do kwoty: sklad koszyka nigdy nie moze
byc pochodna wylacznie ceny jednostkowej, bo kazdy nowy najdrozszy produkt przejmuje rynek.

## Related
- Docelowa architektura (FAZA 1 reworku): koszyk liczony w SZTUKACH zamiast kwoty -
  usuwa cala klase problemu.
