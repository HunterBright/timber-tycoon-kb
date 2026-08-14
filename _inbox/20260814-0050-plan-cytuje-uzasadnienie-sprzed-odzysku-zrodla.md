---
title: Job nocny powtarza rekomendację planu, choć źródło odzyskane tej samej nocy ją unieważniło
type: anti-pattern
status: draft
confidence: medium
verified: ''
date: 2026-08-14
project: Another Quest
tags: [proces, plan, delegacja, nocne-joby, kanon, weryfikacja-u-zrodla]
applies_to: [praca-z-planem, joby-nocne, dokumentacja-decyzji]
source: 'AQ sprint D1 — [S3-doc] memo GATE 2; erraty p/q/r skanonizowane przez [S1] tej samej nocy'
suggested-category: proces/anti-patterns
---

# Job nocny powtarza rekomendację planu, choć źródło odzyskane tej samej nocy ją unieważniło

## The trap

Plan sprintu zawiera rekomendację razem z gotowym uzasadnieniem („opcja A, bo nie ma żadnego X").
Job, który ma tę rekomendację rozpisać, traktuje uzasadnienie jako dane wejściowe i przepisuje je
w ładniejszej formie. Wygląda to na wierność planowi — a jest przepisywaniem nieaktualnego stanu.

## Why it fails

Plan jest **fotografią stanu wiedzy z chwili pisania**. W sprintach, gdzie ta sama noc odzyskuje
brakujące źródło (errata, oryginał dokumentu, eksport rozmowy), plan i kanon rozjeżdżają się
**w ciągu godzin**. Rekomendacja przeżywa swoje uzasadnienie: argument, na którym stała, przestaje
obowiązywać, ale nikt tego nie zauważa, bo kolejny job cytuje plan, a nie źródło.
Efekt: decydent dostaje materiał, który wygląda na przemyślany, a zawiera martwy argument —
i podejmuje decyzję za drogo (kupuje zmiany, które rozwiązują problem już rozwiązany).

## Symptoms

- Plan i świeżo skanonizowane źródło opisują ten sam koszt **innymi słowami** (np. plan: „opcja B
  konkuruje o slot"; errata: „obie rzeczy już siedzą w tym slocie").
- Rejestr stanu maszynowego (`findings.json` i podobne) niesie sformułowanie przepisane z planu,
  a nie wyprowadzone z kanonu.
- Job cytuje plan jako uzasadnienie, choć plan sam odsyła do dokumentu, którego wtedy nie było w repo.

## Correct approach

1. **Karta jobu powinna jawnie mówić: „nie papuguj planu — sprawdź tezy u źródła".** Jedno zdanie
   w zleceniu wystarcza, żeby job czytał erratę zamiast streszczenia.
2. Job zaczyna od **tabeli weryfikacji tez wejściowych** (teza → werdykt → cytat z adresem).
   Tabela zostaje w dokumencie — jest tania, a czyni rozjazd widocznym dla decydenta.
3. Rekomendacja **może zostać ta sama**, ale musi dostać **nowe uzasadnienie z dzisiejszego stanu**.
   Zmiana statusu argumentu („dlatego to konieczne" → „dlatego to opłacalne") jest wynikiem, nie porażką.
4. Rozjazd zapisany w planie/rejestrach zgłasza się jako **osobną pozycję do poprawienia**, zamiast
   poprawiać go po cichu w cudzym pliku — inaczej stara formuła wróci przy następnym cytowaniu.
