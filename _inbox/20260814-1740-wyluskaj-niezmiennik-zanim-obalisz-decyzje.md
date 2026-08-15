---
title: Wyłuskaj niezmiennik, zanim obalisz starą decyzję
type: pattern
status: draft
confidence: high
verified: '2026-08-14'
date: 2026-08-14
project: Another Quest
tags:
- projektowanie
- kanon
- decyzje
- refaktor
- dlug-projektowy
applies_to: []
source: 'sesja GATE 1, Another Quest — odłączenie czarów od broni; docs/process/GATE1_WERDYKTY.md'
suggested-category: workflow/patterns
---

# Wyłuskaj niezmiennik, zanim obalisz starą decyzję

## When to use

Gdy ktoś — dyrektor kreatywny, klient, Ty sam za pół roku — chce zmienić decyzję zapisaną wcześniej
jako obowiązująca. Szczególnie gdy:

- stara decyzja wygląda na arbitralną albo niewygodną,
- nowa propozycja brzmi prościej i „naturalniej",
- nikt obecny nie pamięta, **dlaczego** stara decyzja została podjęta w tej formie.

Ostatni punkt jest najgroźniejszy: decyzja, której uzasadnienie wyparowało, wygląda dokładnie jak
decyzja bez uzasadnienia.

## Steps

1. **Znajdź oryginalny zapis** i przeczytaj jego uzasadnienie, nie samą treść. Jeśli uzasadnienia
   nie ma — to osobne znalezisko, zapisz je.
2. **Rozdziel dwie rzeczy, które w zapisie są sklejone:**
   - **diagnoza / niezmiennik** — jaki problem ta decyzja rozwiązywała,
   - **nośnik** — konkretny sposób, w jaki go rozwiązała.
3. **Sprawdź, czy nowa propozycja podważa diagnozę, czy tylko nośnik.** Prawie zawsze tylko nośnik.
4. **Przenieś niezmiennik do nowej decyzji jako jawny warunek konieczny.** Zapisz go w nowym dokumencie
   tak, żeby następna osoba nie musiała powtarzać tej archeologii.
5. **Powiedz wprost, że diagnoza nie została odrzucona.** To zdejmuje z decydenta poczucie, że
   „cofa własną decyzję", i jednocześnie chroni przed wróceniem starego błędu.

## Why this works

Stare decyzje w dojrzałym projekcie rzadko są kaprysem — zwykle są odpowiedzią na konkretny defekt.
Ale zapisuje się **odpowiedź**, nie **pytanie**. Po miesiącach zostaje sam nośnik, wyglądający na
dogmat, a problem, który on rozwiązywał, jest niewidzialny.

Obalenie nośnika bez wyłuskania niezmiennika **odtwarza pierwotny defekt**, tylko późno i drożej —
bo teraz stoi na nim więcej rzeczy.

Konkretny przebieg, z którego to wyszło: czary były przywiązane do broni. Wyglądało to na sztuczne
ograniczenie i takie było — **ale powodem nie była chęć ograniczania.** Powodem było to, że w całej
grze nie istniała ŻADNA droga zdobycia czaru: definicje czarów były danymi bez wydawcy, drzewko
talentów miało zakaz dawania mocy, skrzynia wydawała tylko sprzęt, tabele łupu nie wydawały czarów.
Broń była jedyną rzeczą, która i tak wypadała, więc na niej powieszono czary.

Niezmiennik brzmiał: **„musi istnieć konkretne źródło, z którego gracz dostaje czar"**.
Nowy model (osobne sloty + zwoje ze skrzyń) spełnia go inaczej. Gdyby ktoś odłączył czary od broni
i nie dopisał źródła, wróciłby dokładnie ten sam bloker — wykryty pierwotnie dopiero pełną symulacją
rozgrywki, czyli drogo.

## Trade-offs

- **Kosztuje jedno czytanie więcej** na starcie. Przy prostych zmianach to narzut bez zysku —
  stosuj przy decyzjach oznaczonych jako obowiązujące, nie przy każdej drobnicy.
- **Może opóźnić decyzję o kilka minut** w momencie, gdy decydent jest już zdecydowany. Warto to
  powiedzieć jako „sprawdzam, czego nie wolno nam zgubić", a nie jako podważanie.
- **Czasem diagnoza NAPRAWDĘ jest nieaktualna** — wtedy wzorzec kończy się wnioskiem „można obalić
  w całości", co też jest wartościowe, bo świadome.

## Variants

- **Gdy uzasadnienia nie ma w zapisie:** poszukaj w commit message i w dokumencie, który decyzję
  wprowadził. Jeśli dalej nic — zapisz brak uzasadnienia jako osobny dług, zanim zmienisz decyzję.
- **Gdy niezmienników jest kilka:** wypisz je listą i przy każdym zaznacz, czy nowy nośnik go spełnia.
  Ten, którego nie spełnia, jest pytaniem blokującym, nie „szczegółem do dopracowania".
- **Gdy zmiana jest duża i wieloplikowa:** niezmienniki idą do dokumentu werdyktów jako sekcja
  „warunek konieczny", nie do komentarza w kodzie — komentarz zginie przy pierwszym refaktorze.

## Related

- [[test-hooka-musi-isc-kanalem-poza-permissions]] — pokrewny kształt: rzecz, która wygląda na
  działającą, bo jej awarii nie da się odróżnić od poprawnego zachowania
