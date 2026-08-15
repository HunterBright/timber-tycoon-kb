---
title: Werdykt zapisany w osobnym pliku decyzji to nie werdykt naniesiony na dokument
type: anti-pattern
status: draft
confidence: high
verified: '2026-08-14'
date: 2026-08-14
project: Another Quest
tags: [proces, dokumentacja, decyzje, sign-off, dryf-dokumentow]
applies_to: [dokumentacja-projektowa, workflow-decyzyjny]
source: 'sesja korekty SZKIELET_DANYCH_CUR.md przed LOCK-iem CUR'
suggested-category: process/anti-patterns
---

# Werdykt zapisany w osobnym pliku decyzji to nie werdykt naniesiony na dokument

## The trap

Sesja decyzyjna kończy się zapisem werdyktów do własnego pliku (`SIGNOFF_*.md`, `GATE_*.md`,
`*_WERDYKTY.md`). Plik jest dokładny, ma numery, uzasadnienia i tabelę „co to zmienia w polach".
Wygląda na zamknięcie sprawy — i sesja odchodzi z poczuciem, że decyzje „są w repo".

**Nie są.** Są w rejestrze decyzji. Dokument, który te decyzje zmieniają, dalej mówi to, co mówił przed sesją.

## Why it fails

Rejestr decyzji i dokument opisujący stan to **dwa różne artefakty o dwóch różnych czytelnikach**.
Rejestr czyta ten, kto pyta „dlaczego tak zdecydowano". Dokument czyta ten, kto pyta „co obowiązuje".
Drugi czytelnik jest liczniejszy i to on implementuje.

Mechanizm porażki jest cichy, bo **żadna bramka tego nie łapie**. Kontrola spójności plików stanu
przechodzi na zielono — nic nie jest zepsute, po prostu jedna prawda jest nowsza od drugiej.
Rozjazd rośnie z każdą kolejną sesją decyzyjną, aż dokument idzie do zamrożenia (LOCK, podpis,
przekazanie) niosąc pytania opisane jako **otwarte**, na które padły odpowiedzi tygodnie wcześniej.

Wzmacniacz: **karta zadania dla następnej sesji jest pisana przez sesję decyzyjną**, więc dziedziczy
jej pole widzenia. Karta wymienia werdykty tej jednej sesji i milczy o zaległościach poprzedniej.
Sesja wykonawcza, która robi dokładnie to, co w karcie, **utrwala rozjazd zamiast go zamknąć**.

## Symptoms

- Plik werdyktów zawiera zdania w rodzaju „**do poprawienia** w §X przed LOCK-iem" — adresowane
  do nikogo konkretnego, bez własnej pozycji na liście zadań.
- Sekcja „pytania otwarte" w dokumencie roboczym i tabela werdyktów w rejestrze **mówią co innego
  o tym samym pytaniu**.
- Dokument cytuje wartości enuma albo nazwy pól, które inny werdykt już skasował
  (u nas: warunek wypłaty odsyłał do dwóch wartości `ExitReason`, z których jedna została usunięta,
  a druga przemianowana).
- Nowa sesja robi rekonesans i odkrywa, że „to już było rozstrzygnięte" — po raz drugi.

## Correct approach

**Werdykt ma dwa adresy i oba są obowiązkowe:** rejestr decyzji (proweniencja: kto, kiedy, dlaczego)
oraz dokument stanu (co obowiązuje). Sesja decyzyjna nie jest skończona, dopóki nie zapisze
**drugiego adresu jako zadania z terminem**, a nie jako zdania w środku uzasadnienia.

Trzy rzeczy, które to domykają:

1. **Rejestr werdyktów kończy się listą adresatów z plikami i sekcjami** — jedna pozycja na dokument,
   nie na werdykt. Zdanie „do poprawienia w §3.3" wewnątrz akapitu jest nośnikiem, którego nikt nie czyta.
2. **Sesja wykonawcza zaczyna od pytania „czy przede mną było coś, czego nie naniesiono?"** — czyli
   `grep`a po nazwie edytowanego pliku w całym repo, zanim otworzy kartę zadania. To jest kilkanaście
   sekund i to właśnie tak wyszła zaległość z poprzedniej sesji.
3. **Dokument niesie własny dziennik zmian z numerami werdyktów.** Wtedy „czego jeszcze nie naniesiono"
   jest różnicą dwóch list, a nie ćwiczeniem z pamięci.

**Czego to NIE znaczy:** że sesja wykonawcza ma dobierać sobie zakres. Naniesienie zaległego werdytu
jest bezpieczne, bo decyzja już padła i ma podpis. **Rozstrzygnięcie czegoś, co nie padło, nie jest** —
i te dwie rzeczy trzeba w raporcie rozdzielić wprost, bo z zewnątrz wyglądają identycznie:
w obu przypadkach dokument urósł o rzeczy spoza karty.
