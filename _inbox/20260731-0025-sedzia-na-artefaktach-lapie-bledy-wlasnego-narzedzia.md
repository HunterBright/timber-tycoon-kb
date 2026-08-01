---
title: Sedzia oceniajacy artefakty lapie bledy w narzedziu, ktore te artefakty produkuje
type: pattern
status: draft
confidence: high
verified: '2026-07-31'
date: '2026-07-31'
project: Kerf - Sawmill Tycoon
tags:
- qa
- sedzia
- walidacja
- artefakty
- metodologia
source: budowa sedziego jakosci 2026-07-31
---

# Sedzia oceniajacy artefakty lapie bledy w narzedziu, ktore te artefakty produkuje

## Kiedy stosowac

Gdy chcesz miec niezalezna ocene wlasnej pracy, a nie masz drugiego czlowieka.

## Wzorzec

Rozdziel na dwie role, ktore **nie moga byc ta sama osoba ani tym samym przebiegiem**:

1. **Kolektor** - skrypt, ktory zbiera artefakty w jedno miejsce: co sie zmienilo,
   co powiedzial build, co powiedziala sonda, co powiedzialy testy, i **ile te pliki maja lat**.
2. **Sedzia** - agent z uprawnieniami **tylko do odczytu**, ktory dostaje wylacznie
   ten katalog i opis zmiany. Nie moze niczego poprawic, wiec nie ma pokusy, zeby
   "szybko naprawic" zamiast ocenic.

Werdykt musi miec wiecej niz dwie wartosci. Minimum: przeszlo, popraw, cofnij,
**brak dowodu**, oraz **wymaga oceny czlowieka**. Bez dwoch ostatnich sedzia musi klamac
w jedna albo druga strone, gdy dowodow nie ma albo gdy sprawa jest estetyczna.

## Dlaczego to dziala lepiej, niz sie spodziewasz

Sedzia czyta artefakty **i sposob, w jaki powstaly**. Przy pierwszym uruchomieniu naszego
sedziego wyszlo, ze:

- plik `dowody.json` produkowany przez kolektor ma **zero bajtow** (blad w skrypcie:
  `switch -Regex` bez `break` dopasowywal kilka wzorcow naraz, klucz przestawal byc tekstem
  i serializacja sie wywracala),
- kolektor pokazuje **440 zmienionych plikow**, wiec nie odroznia ocenianej pracy od tla
  sprzed zadania (brakowalo snapshotu poczatkowego),
- w zmienionym pliku konfiguracji **to samo slowo znaczy dwie rozne rzeczy**
  (dwie niezalezne numeracje "bramek").

Zaden z tych bledow nie zostalby znaleziony przez autora, bo autor wiedzial, co mial na mysli.

## Warunek konieczny

**Kolektor musi liczyc wiek artefaktow.** Przerwana sonda zostawia poprzedni plik wyniku,
wiec raport bez daty potrafi udawac swiezy. Jesli artefakt jest starszy niz oceniana zmiana,
jedyny uczciwy werdykt to "brak dowodu".

Przy pierwszym uruchomieniu nasz sedzia wlasnie tak zrobil: odmowil uznania dwunastogodzinnego
raportu za dowod zmiany sprzed dziesieciu minut.

## Koszty

Jeden przebieg sedziego to kilkanascie wywolan narzedzi. Przy drobnych zmianach to za duzo -
dlatego warto miec macierz "typ zmiany, wymagane dowody", zeby zmiana w dokumentacji
nie uruchamiala pelnej procedury.

## Powiazane

- [[gate-must-have-provable-failure-mode]]
- [[build-is-the-only-truth-editor-lies]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[SEDZIA|Sędzia jakości - na czym go zbudowaliśmy]] - wspolne: artefakty, sedzia, walidacja
- [[gate-must-have-provable-failure-mode|Bramka bez udowodnionego trybu porazki niczego nie pilnuje]] - wspolne: metodologia, walidacja, qa
- [[build-is-the-only-truth-editor-lies|Edytora nie da sie oszukac, zeby udawal build]] - wspolne: metodologia, qa
<!-- /POWIAZANE:auto -->
