---
title: Autotest wołający funkcję zamiast CLI zostawia warstwę argumentów bez żadnego pokrycia
type: anti-pattern
status: draft
confidence: high
verified: '2026-08-15'
date: 2026-08-15
project: Another Quest
tags:
- testing
- cli
- gates
- false-green
applies_to: []
source: 'LOCK CUR 2026-08-15, tools/canon_manifest.py --sprawdz-sie-samemu'
severity: high
suggested-category: engine/anti-patterns
time_lost: 'wykryte przeglądem adwersaryjnym, nie na produkcji'
---

# Autotest wołający funkcję zamiast CLI zostawia warstwę argumentów bez żadnego pokrycia

## Co robiliśmy

Narzędzie bramkowe ma wbudowany autotest (`--sprawdz-sie-samemu`), który buduje sztuczne repo o znanym
z góry werdykcie i sprawdza, czy bramka łapie to, co ma łapać. Do tego dźwignię (`--zepsuj`), która psuje
mechanizm — autotest **musi** wtedy oblać, bo test dający ten sam wynik przy kodzie dobrym i zepsutym
nie mierzy niczego. Wzorzec dobry i sprawdzony.

Wszystkie próby wołały funkcję sprawdzającą **bezpośrednio** i przekazywały jej dźwignie jawnym argumentem.

## Dlaczego to nie działa

Cała warstwa `main()` — **co CLI parsuje, co przekazuje dalej, jaki zwraca kod wyjścia** — leżała poza
testem. Zmierzone konsekwencje, każda przy **zielonym autoteście**:

- dopisanie jednego argumentu w wywołaniu wewnątrz `main()` wyciszało ochronę całej klasy plików;
- flaga trybu porażki przechodziła do trybu produkcyjnego i bramka drukowała „ZIELONO" przy wyłączonym
  porównaniu — czyli **najgorszy możliwy wynik: fałszywe „sprawdzone"**;
- ta sama flaga była opisana w nagłówku narzędzia jako niedostępna w trybie produkcyjnym. Dokumentacja
  opisywała zamiar, nie kod, i nic tego nie łapało.

Sedno: **autotest sprawdzał silnik, a użytkownik dostaje samochód.** Kierownica nie była podłączona
do niczego i żaden test tego nie zauważał.

## Zamiast tego

1. **Przynajmniej jedna próba idzie przez prawdziwe `main()`** — z podmienionym `argv` i przechwyconym
   wyjściem — i sprawdza **kod wyjścia oraz treść komunikatu**, nie tylko wynik funkcji wewnętrznej.
2. **Każda dźwignia trybu porażki ma własną próbę na to, że jest odmawiana w trybie produkcyjnym.**
   Nie „nie używamy jej tam", tylko „narzędzie odmawia, kod 1".
3. **Dźwignia musi oblewać na INNEJ wartości niż kod poprawny** — i najlepiej z komunikatem nazywającym
   przyczynę, nie tautologicznym „coś jest nie tak". Inaczej martwa dźwignia i działająca dają ten sam
   wynik, a dokumentacja redproofu („dźwignia → kod 1") przechodzi tak samo dla obu.

## Jak to wyszło

Nie z testu — z **przeglądu adwersaryjnego**, w którym osobny agent dostał jedno zadanie: *„czy ten redproof
naprawdę coś dowodzi, czy jest samopotwierdzający?"*, z prawem do kopiowania narzędzia i wstrzykiwania
sabotaży. Wypisał listę sposobów wyciszenia ochrony i sprawdził **dla każdego**, czy autotest by go złapał.
Trzy przeszły niezauważone.

To jest tańsze niż się wydaje i działa tylko wtedy, gdy pytanie brzmi „obal ten dowód", a nie „sprawdź,
czy działa".

## Transferability

Dotyczy każdego narzędzia z wbudowanym autotestem i flagami: linterów, migratorów, skryptów CI, bramek
jakości, walidatorów danych. Im mocniejsza obietnica narzędzia („to jest sprawdzone"), tym gorszy skutek
dziury w warstwie, której autotest nie ogląda.

## Related

- [[20260815-2140-nowa-ochrona-wylacza-stara|Dodanie pliku do zbioru pilnowanego przez jedna bramke potrafi wylaczyc druga bramke]] - wspolne: bramki, regresja
- [[20260814-0930-dzwignia-zepsuj-musi-nazwac-winowajce|Dzwignia --zepsuj musi nazwac winowajce, nie tylko podniesc kod wyjscia]] - wspolne: bramki, redproof
