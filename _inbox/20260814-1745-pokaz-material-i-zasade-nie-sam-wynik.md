---
title: Przy decyzji projektowej pokaż materiał i zasadę, nie sam wynik
type: lesson
status: draft
confidence: high
verified: '2026-08-14'
date: 2026-08-14
project: Another Quest
tags:
- komunikacja
- projektowanie
- prezentacja-decyzji
- non-programmer
applies_to: []
source: 'sesja GATE 1, Another Quest — dwie odrzucone prezentacje tej samej tabeli'
severity: medium
suggested-category: workflow/lessons
time_lost: '~20 min'
---

# Przy decyzji projektowej pokaż materiał i zasadę, nie sam wynik

## Problem

Przygotowałem tabelę przypisania 12 czarów do 6 broni i poprosiłem o zatwierdzenie.
Odpowiedź: **„Nie rozumiem, jakimi czarami?"**

Rozpisałem więc, co robi każdy z 12 czarów, i pokazałem tabelę ponownie.
Odpowiedź: **„To nie jest to, co chcę"** — i zmiana kierunku całego systemu.

Trzecia próba, po zmianie modelu: zestaw startowy czterech czarów. Odpowiedź:
**„Szarża dla wojownika, Fireball dla maga, uzdrowienie dla wszystkich. Co jest w takim razie
dla łucznika?"**

Zestaw był poprawny — czwarty czar **był** czarem łucznika (jako jedyny w zestawie skalował się
zręcznością, i istniał w tej postaci wyłącznie dlatego, że wcześniejsza errata naprawiła brak
czarów łucznika w kanonie). Ale w mojej prezentacji tego nie było widać, więc wyglądał na
przypadkowy dobór.

Trzy odrzucenia. Za każdym razem inny brak, ale ten sam kształt błędu.

## Root cause

Prezentowałem **wynik pracy analitycznej**, pomijając dwie warstwy pod nim:

1. **Materiał** — z czego wynik powstał (czym są te 12 czarów).
2. **Zasada porządkująca** — reguła, według której wynik jest taki, a nie inny
   (jedna odpowiedź na problem każdej klasy).

Dla mnie obie warstwy były oczywiste, bo dopiero co je przeczytałem. Dla odbiorcy tabela była
listą nazw bez kontekstu — czyli czymś, co można tylko przyjąć na wiarę albo odrzucić.

**Sedno: bez zasady nie da się ocenić wyniku, można go tylko zaakceptować albo odrzucić w całości.**
Odbiorca nie mógł powiedzieć „zamień pozycję 7", bo nie widział, według czego pozycja 7 tam trafiła.
Odrzucenie w całości było jedyną dostępną mu odpowiedzią.

## Solution

Prezentując decyzję projektową do zatwierdzenia, podaj trzy warstwy w tej kolejności:

1. **Materiał** — czym są elementy, o których mowa. Krótko, po ludzku, jedno zdanie na element.
   Jeśli elementów jest więcej niż kilka, tabela z kolumną „co to robi".
2. **Zasada** — reguła, według której zostały ułożone, i **skąd ta reguła się wzięła**
   (z dokumentu, z pomiaru, z ograniczenia technicznego).
3. **Wynik** — dopiero teraz tabela.

Plus dwie rzeczy, które okazały się skuteczne:

- **Oddziel wymuszone od uznaniowego.** „Te trzy pozycje wynikają z kanonu, swobody jest realnie
  na trzy" pozwala odbiorcy skupić uwagę tam, gdzie jego zdanie faktycznie coś zmienia.
- **Sam wskaż najsłabsze ogniwo.** Zdanie „gdyby coś w tej tabeli miało być zamienione, to właśnie
  to" daje konkretny uchwyt zamiast wyboru „całość albo nic".

## What didn't work

- **Dosypanie szczegółów bez zasady.** Druga próba dodała opisy wszystkich 12 czarów — więcej
  materiału, dalej brak reguły. Odrzucona tak samo.
- **Założenie, że nazwy własne niosą znaczenie.** „Oślepiający Błysk" mówi coś tylko temu, kto
  przed chwilą czytał tabelę z jego parametrami.
- **Uznanie odrzucenia za sygnał, że wynik jest zły.** Wynik był dobry przy drugim i trzecim
  podejściu. Zła była prezentacja — a to dwa różne defekty i mają dwie różne naprawy.

## Transferability

Dotyczy każdej sytuacji, w której ktoś przygotowuje analizę, a ktoś inny ma ją zatwierdzić:
przegląd architektury, wybór biblioteki, plan migracji, propozycja cennika, projekt poziomu.
Nie jest specyficzne dla gier ani dla pracy z osobą nietechniczną — z osobą techniczną kończy
się tym samym, tylko szybciej dochodzi do „a dlaczego akurat tak?".

Wniosek ogólny: **odbiorca, który nie widzi zasady, nie może współpracować przy wyniku — może go
tylko przyjąć lub odrzucić.** Jeśli dostajesz odrzucenia całościowe zamiast poprawek punktowych,
podejrzewaj brak zasady w prezentacji, zanim zaczniesz poprawiać treść.

## Related

- [[wyluskaj-niezmiennik-zanim-obalisz-decyzje]] — ta sama sesja; tam chodzi o czytanie cudzych
  decyzji, tu o przedstawianie własnych
