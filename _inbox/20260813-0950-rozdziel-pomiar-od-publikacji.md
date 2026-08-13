---
title: Rozdziel pomiar od publikacji, zeby raport przestal sam siebie poprawiac
type: pattern
status: draft
confidence: high
verified: '2026-08-13'
date: 2026-08-13
project: GameDevOS
tags:
- agenci
- automatyzacja
- wiarygodnosc
- raportowanie
applies_to: []
source: 'D:\GameDevOS\tools\kwarantanna.py, D:\GameDevOS\tools\radar_prompt.md'
suggested-category: workflow/patterns
---

# Rozdziel pomiar od publikacji, zeby raport przestal sam siebie poprawiac

## When to use

Gdy automat regularnie oglasza cos, co nastepnego dnia odwoluje albo prostuje.
Objaw jest charakterystyczny: **korekty nie ubywaja mimo dopisywania kolejnych
regul**. Kazda regula opisuje jeden kanal zrodlowy i powstaje PO szkodzie,
a nowy kanal przynosi swoj wlasny pierwszy blad.

Typowe zrodla takich twierdzen: roznica miedzy dwiema migawkami strony, relacja
podagenta, jednorazowy odczyt ceny albo licznika.

## Steps

1. **Nazwij chwile, w ktorej automat podejmuje decyzje za wczesnie.** Zwykle
   jest to moment, w ktorym publikuje twierdzenie w tym samym przebiegu,
   w ktorym je pierwszy raz zobaczyl.
2. **Rozdziel pomiar od publikacji na dwa osobne procesy.** Pomiar zostaje
   czesty (u nas codzienny, sam Python, bez modelu jezykowego). Publikacja
   staje sie rzadsza (u nas tygodniowa).
3. **Postaw miedzy nimi rejestr kwarantanny** - plik ze stanem, nie regule
   w instrukcji. Kazde twierdzenie ma tam odcisk, liste dni, w ktorych je
   widziano, i liste niezaleznych potwierdzen.
4. **Odcisk licz z trescia zbita do wzorca**: zamien liczby na znak zastepczy,
   inaczej „219 pobran" i „231 pobran" beda dwoma osobnymi watkami i nic
   nigdy nie potwierdzi sie drugi raz.
5. **Wpisz zasade w kod, nie w prompt**: narzedzie ODMAWIA oznaczenia jako
   opublikowane czegos, co ma jedna probke. Regula, ktora chodzi sama, dziala
   takze wtedy, gdy agent o niej zapomni.
6. **Pokaz w raporcie oba konce**: osobna sekcje „sygnaly do potwierdzenia"
   i osobna „falszywe alarmy, ktore nie weszly do raportu". Ta druga jest
   dowodem, ze mechanizm dziala.

Wyjatek, ktory trzeba zapisac wprost: **wlasny pomiar jest faktem od razu.**
Kwarantanna dotyczy tego, czego agent nie widzial na wlasne oczy.

## Why this works

Jedna probka nie odroznia zmiany w swiecie od potkniecia zrodla. Zadna regula
jezykowa tego nie naprawi, bo agent w chwili publikacji nie ma czym rozstrzygnac.
Drugi pomiar rozstrzyga to bez zgadywania.

Kluczowe jest to, ze **czestotliwosc pomiaru zostaje bez zmian**. Rzadszy pomiar
nie jest pewniejszy, jest tylko rzadszy. Zmienia sie wylacznie moment publikacji,
czyli chwila, w ktorej niepewnosc zamienia sie w zdanie oznajmujace.

## Trade-offs

- **Wiadomosc dociera pozniej.** Przy tygodniowej publikacji rzecz z wtorku
  czlowiek przeczyta w poniedzialek. To jest realny koszt i trzeba go przyjac
  swiadomie: kupujemy nim to, ze przeczytane raz nie bedzie odwolane.
- **Dwa procesy zamiast jednego.** Wiecej ruchomych czesci, wiecej miejsc,
  w ktorych cos moze przestac chodzic. Osobne zadanie pomiarowe warto zrobic
  tanim i bez modelu jezykowego, zeby awaria drogiej czesci nie zabierala danych.
- **Rejestr trzeba sprzatac.** Bez zamykania porzuconych sygnalow lista rosnie
  w nieskonczonosc i przestaje cokolwiek znaczyc.

## Variants

- **Potwierdzenie kanalem zamiast czasem.** Gdy czekanie jest zbyt drogie,
  drugim pomiarem moze byc inne zrodlo tego samego dnia. U nas obie drogi sa
  rownowazne: dwa rozne dni ALBO jedno niezalezne potwierdzenie.
- **Prog wiekszy niz dwa.** Przy kanalach znanych z halasu warto wymagac trzech
  probek. Wtedy prog nalezy do kanalu, a nie do calego systemu.
- **Zastosowanie poza raportowaniem rynku.** Ten sam uklad pasuje do alarmow
  z monitoringu (jeden odczyt to nie awaria), do wykrywania regresji wydajnosci
  i do wszystkiego, gdzie pomiar jest zaszumiony, a decyzja kosztowna.
