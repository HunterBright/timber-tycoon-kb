---
title: Adwersaryjna weryfikacja znalezisk z domyślnym obaleniem
type: pattern
status: draft
confidence: high
verified: '2026-08-14'
date: 2026-08-14
project: Another Quest
tags: [agenci, weryfikacja, przeglad-dokumentacji, code-review, orkiestracja]
applies_to: [praca-z-llm, przeglad-kodu, audyt-dokumentacji]
source: 'Batch B8 — 172 znaleziska, 84 obalone (49%), 52 potwierdzone'
suggested-category: workflow/patterns
---

# Adwersaryjna weryfikacja znalezisk z domyślnym obaleniem

## Kiedy stosować

Gdy zlecasz agentom **szukanie problemów** — w kodzie, w dokumentacji, w projekcie — i wynik ma trafić
do człowieka jako lista rzeczy do zrobienia. Czyli zawsze, gdy fałszywy alarm kosztuje czyjś czas.

Model proszony o znalezienie problemów **znajdzie je zawsze**, także tam, gdzie ich nie ma. Surowa lista
z przeglądu ma zwykle 40–60% szumu i nie da się jej odróżnić od sygnału bez otwarcia każdego pliku —
czyli bez wykonania całej pracy raz jeszcze.

## Kroki

1. **Faza szukania: wiele wąskich obiektywów, nie jeden szeroki.** Zamiast „przejrzyj wszystko" — kilkanaście
   agentów, każdy z jednym zakresem (para dokumentów · konkretny blok modułów · erraty · liczby · kompletność).
   Każdy zwraca znaleziska w **ustalonej strukturze**, z obowiązkowym dosłownym cytatem z obu stron.
2. **Faza obalania: 2–3 sceptyków na znalezisko, każdy z innym kątem ataku.** Kąty, które się sprawdziły:
   - **wierność cytatu** — otwórz plik, sprawdź, czy cytat tam stoi, dosłownie i w kontekście;
   - **klasyfikacja** — czy to naprawdę sprzeczność, czy tylko luka; czy hierarchia źródeł już tego nie rozstrzyga;
   - **realność skutku** — załóż, że to prawda, i sprawdź, czy cokolwiek złego się stanie.
3. **Instrukcja sceptyka musi zawierać domyślne rozstrzygnięcie:** *„twoim domyślnym założeniem jest, że
   znalezisko jest błędne; w razie wątpliwości orzekaj OBALONE"*. Bez tego zdania sceptyk potwierdza wszystko,
   bo model domyślnie współpracuje z tezą, którą dostał.
4. **Znalezisko przeżywa większością głosów.** Zapisz liczbę głosów przy każdym — czytelnik od razu widzi,
   co jest pewne, a co przeszło 2:1.
5. **Raportuj też obalone.** Krótka lista „co zgłoszono i dlaczego odrzucono" chroni następną sesję przed
   odkryciem tego samego i naprawianiem rzeczy, która jest w porządku.

## Dlaczego to działa

Znalezienie problemu i obalenie problemu to **dwa różne zadania poznawcze**. Ten sam model, poproszony
o jedno i drugie w tej samej odpowiedzi, wykona pierwsze i udawał będzie drugie — bo nie ma powodu
kwestionować własnej tezy sprzed dwóch akapitów. Rozdzielenie ról na osobne konteksty usuwa tę zachętę.

Domyślne obalenie przesuwa **koszt błędu** na właściwą stronę: fałszywy alarm kosztuje człowieka rozmowę
o problemie, którego nie ma, a przeoczenie i tak wyjdzie przy następnej bramce.

## Kompromisy

- **Kosztuje.** W AQ: 14 obiektywów → 172 znaleziska → 322 agentów łącznie. Nie stosuj do zadań,
  gdzie lista i tak trafi z powrotem do agenta, a nie do człowieka.
- **Obala też prawdziwe znaleziska.** Przy 49% odrzuceń część odrzuconych była pewnie słuszna.
  Dlatego lista obalonych ma zostać w raporcie, a nie zniknąć.
- **Sceptycy pisują długie uzasadnienia.** Warto je zachować przy znaleziskach o głosach 2:1 i przyciąć
  przy 3:0 — inaczej raport puchnie ponad użyteczność.

## Warianty

- **Tanio:** jeden sceptyk zamiast trzech, ale z tą samą instrukcją domyślnego obalenia. Łapie większość szumu.
- **Świadomy limit:** znaleziska najniższej wagi puszczaj bez weryfikacji, ale **oznacz je w raporcie
  jako niezweryfikowane i policz** — cichy limit czyta się potem jako „sprawdzono wszystko".
- **Zamiast głosowania większością:** przy decyzjach nieodwracalnych podnieś próg do jednomyślności.
