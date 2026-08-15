---
title: "Nota zmian mówi, że przepisano — nie, że pasuje. Kontrakt zamrażaj po maszynowym porównaniu z konsumentem"
type: anti-pattern
status: draft
confidence: high
verified: '2026-08-14'
date: 2026-08-14
project: Another Quest
tags: [dokumentacja, kontrakt-danych, lock, weryfikacja, schemat, save-migration]
applies_to: [projekty-doc-first, schematy-danych, definicje-save, api-kontrakty]
source: 'AQ, wejście do LOCK CUR; raport _Handoff/RAPORT_WEJSCIOWY_LOCK_CUR_2026-08-14.md'
suggested-category: workflow/anti-patterns
---

# Nota zmian mówi, że przepisano — nie, że pasuje

## The trap

Dokument-kontrakt (schemat danych, definicja zapisu gry, kształt rekordu) ma zostać zamrożony.
Warunkiem wejścia jest „konsumenci przepisani wg werdyktów". Konsument zostaje przepisany, dostaje
na końcu **notę zmian** — tabelę „było → jest → którym werdyktem" — i nota wygląda na dowód.

Pokusa: sprawdzić **pokrycie werdyktów** (czy każda decyzja ma ślad) i uznać warunek za spełniony.
Pokrycie jest łatwe do policzenia, wygląda rygorystycznie i daje zielone światło.

## Why it fails

**Pokrycie werdyktów i zgodność kształtu to dwa różne pytania.** Nota zmian odpowiada wyłącznie
na pierwsze. Konsument może nanieść wszystkie decyzje co do jednej, a mimo to renderować kontrakt
inaczej — bo przy przepisywaniu dokłada pola, których potrzebuje do swojej roboty, i robi to w dobrej
wierze, często pisząc wprost „przepisane pole po polu ze źródła".

W AQ pokrycie wyszło idealne: 30 z 30 werdyktów miało ślad, lista „bez śladu" była pusta.
Maszynowe zestawienie **listy pól** obu dokumentów dało **osiem różnic** — w tym dwa pola, których
w zamrażanym kontrakcie w ogóle nie było, dwie różne nazwy tego samego pola, dwa różne typy jednego
pola i brakującą wartość w wyliczeniu, przez którą główne źródło waluty wpadało do worka „inne".

Koszt pomyłki jest asymetryczny: **przed zamrożeniem** to edycja linijki, **po zamrożeniu** — formalna
poprawka kontraktu, a jeśli kontrakt trafia do zapisu gry, także migracja zapisów graczy.

## Symptoms

- Warunek wejścia brzmi „konsument przepisany", a nie „kształt zgodny";
- konsument pisze o sobie „przepisane pole po polu ze źródła" (to deklaracja intencji, nie wynik testu);
- kontrakt i konsument mają **wspólną nazwę struktury**, ale jeden jest wejściem, a drugi wynikiem;
- w kontrakcie jest wyliczenie („źródła", „kanały", „typy") ogłoszone jako kompletne, a konsument
  dokłada nowy przypadek, który do żadnej wartości nie pasuje i ląduje w `Other`;
- **pytanie „czy to w ogóle trafia do zapisu?" nie ma odpowiedzi** — czyli nikt nie zna ceny pomyłki.

## Correct approach

Przed zamrożeniem kontraktu zrób **dwa osobne przebiegi**, nie jeden:

1. **Pokrycie decyzji** — czy każda decyzja ma ślad. Tanie, mechaniczne, `grep`em, nie modelem.
2. **Różnica kształtu** — wypisz **listę pól z obu dokumentów obok siebie** i porównaj pole po polu:
   nazwa, typ, sekcja, kto wypełnia. Każda różnica to pytanie do właściciela decyzji, nie do redakcji.
   Zwróć uwagę na trzy rzeczy, które łatwo przeoczyć: pola dołożone przez konsumenta,
   **wyliczenia bez wartości na nowy przypadek**, i pola nazwane inaczej po obu stronach.
3. **Ustal cenę pomyłki, zanim ją popełnisz** — czy kontrakt trafia do trwałego zapisu. Odpowiedź
   „nie" zmienia decyzję z nieodwracalnej w odwracalną i wolno wtedy zamrażać szybciej.

Reguła w jednym zdaniu: **nota zmian jest dowodem pracy, nie dowodem zgodności.**
Dowodem zgodności jest różnica dwóch list pól.

## Dopisek z sesji decyzyjnej tego samego dnia — różnicę też trzeba zrobić w OBIE strony

Przebieg nr 2 został wykonany i dał osiem różnic. **Dziewiątą wykryła dopiero bramka decyzyjna**,
przy okazji werdyktu o wspólnej nazwie struktury.

Pominięte pole nie wyglądało na różnicę kontraktu, tylko na **wewnętrzną hydraulikę konsumenta**:
było wejściem dla trzeciego modułu (obietnica nagrody, którą konsument przewozi dalej, bo tylko on
umie ją odczytać). Porównanie szło od strony zamrażanego dokumentu i pola „nie moje" cicho odpadło.

**Zaostrzenie punktu 2:** wypisz najpierw **pełną listę pól konsumenta** i przy każdym zaznacz,
czy stoi w zamrażanym kontrakcie. Pola bez pary rozstrzygnij jawnie w trzech kubełkach — *dochodzi
do kontraktu* / *zostaje po stronie konsumenta* / *idzie osobnym argumentem wywołania*.
Kubełek trzeci jest tym, o którym się zapomina, i właśnie dlatego pole wygląda na niczyje.

Praktyczny sygnał: **werdykt „jedna struktura, nie dwie" natychmiast zmienia status pól niczyich**
z „szczegół konsumenta" na „rubryka zamrażanego kontraktu". Jeśli taki werdykt pada, przejrzyj
listę pól **jeszcze raz, po nim** — poprzednia różnica była liczona przy innym założeniu.
