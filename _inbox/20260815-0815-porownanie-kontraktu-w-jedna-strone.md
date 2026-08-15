---
title: Porównanie dokumentu z jego konsumentem tylko w jedną stronę
type: anti-pattern
status: draft
confidence: high
verified: ''
date: 2026-08-15
project: Another Quest
tags: [dokumentacja, kontrakty, weryfikacja, lock, przeglad]
applies_to: [proces, dokumentacja, spec-review]
source: 'weryfikacja gotowosci SZKIELET_DANYCH_CUR.md do LOCK-a, 2026-08-15'
suggested-category: process/anti-patterns
---

# Porównanie dokumentu z jego konsumentem tylko w jedną stronę

## The trap

Struktura danych jest opisana w dokumencie A (właściciel kształtu), a dokument B (konsument)
deklaruje, że przepisał ją „pole po polu". Przed zamrożeniem A ktoś zestawia obie listy i wypisuje
różnice. Zestawienie idzie odruchowo w jedną stronę: **co ma konsument, czego nie ma właściciel** —
bo to są rzeczy, które trzeba dopisać, więc same się pchają na listę. Wynik: „osiem różnic,
wszystkie rozstrzygnięte, pozycji bez werdyktu zero". Brzmi jak komplet.

## Why it fails

Druga strona zestawienia — **co ma właściciel, czego nie ma konsument** — nie produkuje żądań
zmiany, więc nikt jej nie liczy. A to ona ujawnia pola-sieroty: takie, które nikt nie wypełnia.
Pole bez producenta wygląda w dokumencie identycznie jak pole żywe: ma nazwę, typ i komentarz.
Wychodzi dopiero przy implementacji, już po zamrożeniu — czyli wtedy, gdy usunięcie kosztuje
procedurę zmiany, a nie skreślenie linijki.

Efekt uboczny tej asymetrii: pola-sieroty są zwykle **najstarsze** w dokumencie. Powstały w pierwszej
wersji, przeszły przez wszystkie przeglądy i ani razu nie były przedmiotem decyzji — więc grep po
rejestrach werdyktów daje dla nich zero trafień. Brak śladu w rejestrze czyta się jako
„bezsporne", a znaczy „nikt nigdy na to nie spojrzał".

## Symptoms

- Raport z przeglądu podaje liczbę różnic i słowo „wszystkie" — bez podania, **w którą stronę** liczono.
- Konsument deklaruje „przepisane pole po polu", ale jego lista jest krótsza od listy właściciela.
- Pole ma komentarz odsyłający do modułu, który jeszcze nie powstał.
- Pole jest wyliczalne z dwóch innych pól obok (znacznik startu i końca → czas trwania), w dokumencie,
  który skądinąd konsekwentnie kasuje sumy jako „drugie źródło prawdy".

## Correct approach

Zestawienie kontraktu robi się **jako różnicę symetryczną dwóch zbiorów nazw** i raportuje obie
połowy osobno, nawet gdy druga jest pusta — pusta połowa też jest wynikiem. Mechanicznie: wypisz
nazwy pól z obu dokumentów, odejmij w obie strony, dopiero potem oceniaj.

Dla każdego pola, które zostaje po stronie właściciela bez odpowiednika u konsumenta, zadaj jedno
pytanie: **kto to wypełnia i w którym kroku**. Brak odpowiedzi = pozycja do werdyktu przed
zamrożeniem, nie po. Kryterium jest to samo, którym sprawdza się puste sloty na wartości
(„slot bez konsumenta = do wyrzucenia") — tylko obrócone: **pole bez producenta = do wyrzucenia
albo do przypisania właściciela**.
