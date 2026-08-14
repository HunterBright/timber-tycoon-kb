---
title: Nie cytuj usuwanego tekstu w notce patcha, gdy odbiór jest grepem
type: anti-pattern
status: draft
confidence: high
verified: '2026-08-13'
date: 2026-08-13
project: Another Quest
tags: [dokumentacja, grep, testy-odbioru, kanon, patch]
applies_to: [dokumentacja-kanonu, migracje-tekstowe, refaktory-nazw]
source: 'noc D1 AQ, job [S1] — patch K-01 (sufit enchantu) w M01/M17'
suggested-category: process/anti-patterns
---

# Nie cytuj usuwanego tekstu w notce patcha, gdy odbiór jest grepem

## The trap

Patchujesz dokument (albo zmieniasz nazwę symbolu w kodzie) i — słusznie — dopisujesz notkę „dawny zapis `X` wykreślony, obowiązuje Y". Odbiór zadania to `grep -E "<wzorzec defektu>"` = pusty. Notka jest dobrym zwyczajem archiwizacyjnym i dokładnie ona wywala test.

## Why it fails

Wzorzec odbioru nie odróżnia wystąpienia-defektu od wystąpienia-cytatu. Dla grepa „usunięte" i „opisane jako usunięte" to ten sam ciąg znaków. Im lepiej udokumentujesz zmianę cytatem verbatim, tym pewniej złamiesz bramkę.

## Symptoms

- Grep odbioru zwraca dokładnie tyle trafień, ile dopisałeś notek — i wszystkie wskazują na linie, które sam napisałeś tej sesji.
- Odruch „to fałszywy alarm, wzorzec jest za szeroki" → pokusa rozluźnienia wzorca. To pogarsza sprawę: wzorzec był zweryfikowany na źródle i trafiał dokładnie linie defektu.

## Correct approach

1. W notce patcha **opisz** stary zapis, nie cytuj go: „dawna stała-mnożnik sufitu (czerwcowy zapis, pięciokrotność bazy)" zamiast wklejenia `5× (base + punkty)`.
2. Verbatim starej treści trzymaj tam, gdzie grep odbioru nie sięga: wpis w rejestrze rozstrzygnięć / plik konfliktów / message commita. Historia i tak trzyma oryginał.
3. Uruchom grep odbioru **po** dopisaniu notek, nie po samej edycji defektu — inaczej wykryjesz to dopiero przy weryfikacji.
4. Nie modyfikuj wzorca odbioru, żeby przepuścił własną notkę. Wzorzec jest kontraktem; to notka ma się dopasować.
