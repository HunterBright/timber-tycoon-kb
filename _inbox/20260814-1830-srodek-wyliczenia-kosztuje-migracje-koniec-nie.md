---
title: Dokładanie wartości w ŚRODKU wyliczenia kosztuje migrację zapisów, na KOŃCU nie
type: lesson
status: draft
confidence: high
verified: ''
date: 2026-08-14
project: Another Quest
tags:
- save-schema
- design-decisions
- migration
- gate
- progression
applies_to: []
source: 'GATE B8, pytanie 2 (drabina rang F-D / F-A / S-SS-SSS)'
severity: high
suggested-category: design/lessons
time_lost: ''
---

# Dokładanie wartości w ŚRODKU wyliczenia kosztuje migrację zapisów, na KOŃCU nie

## Problem

Sesja decyzyjna miała ustalić drabinę rang. Źródła projektu podawały wyłącznie **zakresy**:
demo F–D, wczesny dostęp F–A, pełna gra S–SS–SSS. Litery E, C i B nie występowały w żadnym dokumencie —
nikt nigdy nie zdecydował, czy istnieją.

Pytanie wyglądało na jedno („ile stopni?"), a naprawdę było dwoma o **radykalnie różnym koszcie**:

- czy istnieje **E**, czyli litera leżąca **w środku zakresu dema**;
- czy istnieją **SS/SSS**, czyli stopnie na **końcu** drabiny.

Bez rozdzielenia tych dwóch pytań łatwo odłożyć oba („zdecydujemy przed wczesnym dostępem"),
bo brzmią tak samo.

## Root cause

Wartość wyliczeniowa zapisana u gracza to **liczba porządkowa, nie napis**. Zapis gry przechowuje pozycję
w wyliczeniu, a nie jej znaczenie.

- **Dopisanie na końcu** (`F E D C B A S` → `+ SS SSS`) nie rusza żadnej istniejącej pozycji.
  Zapis gracza, który miał `A`, dalej znaczy `A`. Koszt: zero.
- **Wstawienie w środku** (`F D` → `F E D`) przesuwa wszystko powyżej punktu wstawienia.
  Zapis gracza, który miał `D`, po zmianie znaczy `E`. **Wszystkie istniejące zapisy trzeba przeliczyć** —
  to migracja plików ludzi, którzy już grają, a nie zmiana danych w projekcie.

Ta sama asymetria dotyczy każdego wyliczenia idącego do zapisu: powody zakończenia sesji, typy zdarzeń,
poziomy trudności, kategorie przedmiotów.

## Solution

W sesji decyzyjnej **rozbij pytanie o wyliczenie na dwa i wyceń je osobno**:

1. **„Czy między istniejącymi wartościami czegoś brakuje?"** — musi paść **przed** pierwszym zapisem gracza.
   To jest pytanie drogie i nieodwracalne.
2. **„Czy na końcu coś dojdzie?"** — można odłożyć bez kosztu. Warto powiedzieć wprost, że odłożenie
   jest bezpieczne **wyłącznie na końcu**, bo inaczej za pół roku ktoś dołoży w środku „bo przecież
   odłożyliśmy tę decyzję".

Praktyczny skrót do postawienia decydentowi: *„górę można dopisać kiedykolwiek; ta jedna litera
w środku kosztuje przepisanie zapisów wszystkich graczy"*. W Another Quest to zdanie zamieniło
odłożenie decyzji na rozstrzygnięcie całej drabiny w jednej turze.

## What didn't work

**Pytanie „ile stopni ma mieć drabina?"** — dostaje odpowiedź o odczuciu z gry („żeby awans był
odczuwalny"), a nie o koszcie technicznym, bo w tej formie koszt jest niewidoczny.

**Odłożenie całości do czasu, aż będzie wiadomo więcej** — brzmi ostrożnie, a jest najdroższą opcją,
jeśli „więcej" oznacza dane z playtestu **po** pierwszych zapisach.

## Transferability

Dotyczy każdego projektu z trwałym stanem gracza, niezależnie od gatunku i silnika: rangi, poziomy
trudności, typy zdarzeń w telemetrii, kategorie przedmiotów, powody zakończenia sesji, statusy
w bazie danych. Wszędzie tam, gdzie zapisujesz **pozycję w wyliczeniu**, a nie pełny napis, obowiązuje
ta sama asymetria: koniec jest darmowy, środek kosztuje migrację.

Poza grami: dokładnie ten sam mechanizm w migracjach schematów baz danych i w wersjonowaniu protokołów.

## Related

- [[pokaz-material-i-zasade-nie-sam-wynik]] — bez warstwy „zasada" decydent nie widzi kosztu opcji
  i może tylko przyjąć albo odrzucić całość.
