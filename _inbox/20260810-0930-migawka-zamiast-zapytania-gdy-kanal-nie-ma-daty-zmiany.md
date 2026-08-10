---
title: Migawka zamiast zapytania, gdy kanal nie ma daty zmiany
type: pattern
status: draft
confidence: high
verified: '2026-08-10'
tags: [monitoring, zwiad, licencje, ceny, narzedzia, radar]
date: '2026-08-10'
project: GameDevOS
suggested-category: workflow/patterns
source: https://extensions.blender.org/api/v1/extensions/ oraz https://openrouter.ai/api/v1/models
applies_to: [radar, sledzenie-zaleznosci, sledzenie-licencji, sledzenie-cen]
---

# Migawka zamiast zapytania, gdy kanal nie ma daty zmiany

## Kiedy stosowac

Gdy chcesz wiedziec, **co sie zmienilo** w cudzym katalogu, rejestrze albo
sklepie, a ten katalog **nie ma widoku „ostatnio zmienione"**.

Rozpoznanie jest proste i warto je zrobic zanim zaczniesz szukac lepszego
zapytania: **jesli kazde pole z data opisuje moment DODANIA pozycji, a nie
moment jej ostatniej zmiany, to zadne zapytanie nie odpowie na Twoje pytanie.**
Szukanie sprytniejszego zapytania jest wtedy strata czasu.

Trzy zmierzone przyklady tego samego ksztaltu:
- **sklep z rozszerzeniami Blendera**: sortowanie idzie po dacie pierwszego
  zatwierdzenia, wiec aktualizacja istniejacego dodatku jest niewidoczna;
- **rejestr modeli u posrednika**: jest data pierwszego wystawienia modelu,
  nie ma zadnej daty zmiany, wiec **podwyzka ceny jest niewidoczna**;
- **rejestr wtyczek Obsidiana**: plik z lista, bez wersji i bez dat.

## Kroki

1. **Sprawdz, czy sortowanie po dacie w ogole dziala**, i zrob to na kluczu
   celowo bezsensownym. Jesli `?sort_by=-cokolwiek` daje te sama liste co
   `?sort_by=-date_updated`, to sortowanie po cichu wraca do domyslnego.
2. **Znajdz interfejs oddajacy komplet jednym strzalem.** Zwykle istnieje obok
   strony i jest szybszy niz przechodzenie po stronach listy.
3. **Policz wiersze i policz rozne klucze.** Jesli sie roznia, masz pozycje
   wystepujace kilka razy (rozne systemy, rozne warianty) i klucz po samym
   identyfikatorze bralby z nich losowa.
4. **Zapisz tylko te pola, ktorych zmiana cos znaczy.** Wiecej pol to wiecej
   falszywych alarmow.
5. **Znormalizuj wartosci przed porownaniem.** Ta sama liczba potrafi przyjsc
   w kilku zapisach.
6. **Dodaj kontrole pobrania**: minimalna sensowna liczba pozycji plus obecnosc
   jednej, o ktorej wiesz, ze tam jest.
7. **Pierwszy przebieg musi wygladac inaczej niz spokojny.** Brak poprzedniej
   migawki to nie jest „zero zmian".
8. **Sprawdz `git check-ignore` na pliku migawki**, zanim uznasz robote za
   skonczona.

## Dlaczego to dziala

Roznica dwoch stanow odpowiada na pytanie, na ktore **zadne pojedyncze
zapytanie odpowiedziec nie umie**, bo informacja o zmianie po prostu nie
istnieje po stronie serwera. Nie wyciagasz jej z kanalu, tylko **wytwarzasz ja
u siebie**, przechowujac wczorajszy stan.

## Koszty i kompromisy

- Jeden plik dziennie i jedno pobranie. Przy katalogu rzedu tysiaca pozycji
  to setki kilobajtow.
- **Zaczynasz widziec dopiero od drugiego dnia.** Migawki nie da sie zrobic
  wstecz, wiec im pozniej zaczniesz, tym pozniej zaczniesz widziec.
- Wykrywasz **fakt** zmiany, nie jej powod. Powod trzeba doczytac.

## Trzy pulapki, kazda zlapana na zywych danych

1. **Klucz sortowania cicho wracajacy do domyslnego.** Bez kontroli na kluczu
   bezsensownym uwierzysz, ze masz liste posortowana po dacie zmiany.
2. **Ten sam identyfikator w kilku wierszach.** 1345 wierszy okazalo sie 1221
   pozycjami, bo czesc ma po kilka paczek na rozne systemy. **Sama zmiana
   kolejnosci po stronie serwera dalaby wtedy kilkadziesiat falszywych
   alarmow.** Lekarstwo: trzymaj zbior posortowany, nie pojedyncza wartosc.
3. **Ta sama wartosc w kilku zapisach.** Cena `0.000005`, `5e-06`
   i `0.0000050` to jedna liczba. Porownanie tekstow zglaszaloby zmiane
   codziennie, a **falszywy alarm kosztuje tyle samo zaufania co przeoczenie**:
   bramka, ktora krzyczy bez powodu, zostaje wylaczona po tygodniu i wtedy nie
   zlapie tej jednej zmiany, ktora naprawde uwiera.

## Warianty

- **Migawka pol wrazliwych prawnie** (licencja, blokada terytorialna). Tu
  wartosc jest najwyzsza, bo **licencja jest jedyna rzecza, ktora potrafi
  uniewaznic cala prace wstecz**, a zmienia sie po cichu.
- **Migawka liczb, ktore raz wpisane do zestawienia zyja tam miesiacami**
  (cena, limit, wersja). Nikt ich nie mierzy ponownie, bo zwiad patrzy na to,
  co nowe.
- **Migawka obecnosci** (czy pozycja nadal istnieje). Znikniecie z katalogu
  bywa wazniejsze niz pojawienie sie.

## Dowod, ze zadzialalo

Migawka sklepu Blendera powstala 09.08.2026 i **zadzialala drugiego dnia**:
10.08 zglosila **16 nowych pozycji naraz** po dziesieciu dniach ciszy. Strony
tych dodatkow podawaly daty publikacji sprzed tygodnia, a lista sortowana po
dacie zatwierdzenia potwierdzila, ze katalog **zatwierdzil zalegla kolejke
jednym ruchem**. Bez migawki przeczytalibysmy to jako „stare rzeczy, ktore
przeoczylismy", i **przegapili narzedzie, na ktore czekalismy**.

Migawka rejestru modeli powstala 10.08.2026. Kontrola na celowo zepsutej
migawce wykryla wszystkie cztery podlozone zmiany, w tym **przemianowanie
dostawcy przy niezmienionym identyfikatorze**, ktorego nie planowalem jako
kategorii i ktore dopisalem dopiero po zobaczeniu go w zywych danych.

## Czy to przeniesie sie na inny projekt

Tak, i to bez zmian. Kazdy projekt zalezy od cudzych katalogow: rejestrow
pakietow, sklepow z dodatkami, list modeli, cennikow. **Wszedzie tam pytanie
„co sie zmienilo od wczoraj" jest wazniejsze niz „co tam jest", a prawie
zaden z tych katalogow na nie nie odpowiada.**

## Powiazane
- [[20260810-1015-nieobecnosc-w-jednym-kanale-nie-jest-nieobecnoscia-w-swiecie]]
- [[20260809-1140-cztery-pytania-do-regulaminu-uslugi-generatywnej]]
