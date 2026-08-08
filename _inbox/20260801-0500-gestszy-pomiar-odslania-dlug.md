---
type: pattern
project: Kerf - Sawmill Tycoon
suggested-category: engine/patterns
tags: [pomiar, solver, proceduralne, ubrania, bramki, blender, iteracja]
date: 2026-08-01
status: draft
---

# Zagęszczenie pomiaru w zbieżnym układzie odsłania dług i destabilizuje - kolejność napraw

## Kontekst
Proceduralny układ warstw ubrań (pchnięcia do marginesów + wygładzanie +
mechanizmy krawędzi, pętla do zbieżności). Widoczne wady (warstwa spodnia
przebijająca PRZEZ ścianki między wierzchołkami) wymusiły dodanie pomiaru
środków ścianek do istniejącego pomiaru wierzchołkowego. Skutek: 10
czerwonych budów zanim układ znów zbiegł - ale każda czerwień to była
realna wiedza.

## Wzorzec: co się dzieje po zagęszczeniu pomiaru
1. **Odsłania stary dług**: rzeczy, które zawsze były zepsute, ale
   niemierzalne (kamizelka miała te same dziury co ogrodniczki).
2. **Destabilizuje sprzężone mechanizmy**: nowe pchnięcia sumują się na
   wspólnych wierzchołkach (wierzchołek należy do ~4 ścianek =
   poczwórstrzelenie), szarpią krawędzie, budzą kaskadę warstw
   (każda podnosi następną z opóźnieniem przejścia).
3. **Odsłania fantomy ramy pomiaru**: promień "wysokości nad tułowiem"
   trafia w RĘKAW źródła przy pasze; nowe punkty pomiaru trafiają tam,
   gdzie wierzchołki nigdy nie trafiały.

## Kolejność napraw (sprawdzona)
1. MAKSIMUM zamiast sumy pchnięć per wierzchołek w obrębie stanu.
2. Wierzchołki linii krawędzi PRZYPIĘTE wobec pchnięć powierzchni -
   krawędzie mają własne mechanizmy i osobną miarę.
3. Nowy pomiar dostaje WŁASNY, mały margines (gasi widoczną wadę);
   pełne odstępy trzyma stary pomiar - inaczej lawina.
4. Fantomy ramy pomiaru gasić U ŹRÓDŁA (wyciąć rękawy z BVH źródła),
   NIE pudełkami stref: fantom o ciągłym rozkładzie zawsze wyjdzie na
   krawędzi pudełka (4 budowy gonienia po y/z bez skutku).
5. Strefy fizycznie niespełnialne (4 warstwy w szczelinie biodro-ręka)
   wyłączać JEDNĄ wspólną funkcją dla naprawiacza i bramki.
6. Wszystkie mechanizmy i bramki na TYCH SAMYCH, ŚWIEŻYCH mapach
   klasyfikacji per przejście - stara mapa = mechanizm nie widzi tego,
   co mierzy bramka (trzy niezależne wystąpienia w jednej sesji).

## Anty-lekcja nośnika
Stan przenoszony między budowami przez plik (JSON malowania zbierany
z POPRZEDNIEGO artefaktu) nadpisuje ręczne poprawki nośnika - redukcja
"8 ścianek -> 2" utrwaliła się w nośniku i poprawka reguły nie miała
już na czym działać. Naprawa: przywrócić nośnik z kopii i zbudować BEZ
poprzedniego artefaktu. Zawsze rób kopię nośnika przed zmianą reguły,
która go redukuje.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260728-1500-bramka-ponad-sufitem|Prog bramki ponad zmierzonym sufitem zamienia kazda runde w porazke]] - wspolne: iteracja, bramki, pomiar
- [[20260731-1055-post-krok-poza-petla-solvera|20260731-1055-post-krok-poza-petla-solvera]] - wspolne: iteracja, solver
- [[20260801-0700-adr-npc-od-zera-zamiast-solvera-warstw|20260801-0700-adr-npc-od-zera-zamiast-solvera-warstw]] - wspolne: solver, ubrania
- [[20260731-2200-slepa-dzwignia-debugger-bramek|20260731-2200-slepa-dzwignia-debugger-bramek]] - wspolne: proceduralne, bramki, blender
- [[20260731-1050-rowne-krawedzie-ubran-bisect-plane|20260731-1050-rowne-krawedzie-ubran-bisect-plane]] - wspolne: ubrania, blender
- [[20260728-1140-miernik-ktory-klamie-inaczej|Zanim zaufasz bramce, sprawdz, czy mierzy to, co widac]] - wspolne: bramki, pomiar
<!-- /POWIAZANE:auto -->
