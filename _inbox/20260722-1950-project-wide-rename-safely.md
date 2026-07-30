---
title: Zmiana nazwy w calym projekcie bez psucia zapisow i kodowania plikow
type: pattern
status: draft
confidence: low
verified: ''
date: '2026-07-22'
project: Kerf - Sawmill Tycoon
tags:
- rebranding
- rename
- refactor
- encoding
- unity
- editor-menus
- save-data
applies_to:
- any-project
- unity
source: ''
suggested-category: workflow/patterns
---

# Zmiana nazwy w calym projekcie bez psucia zapisow i kodowania plikow

## Kiedy stosowac
Gdy pada polecenie "zmien nazwe X na Y wszedzie". Brzmi mechanicznie, a ma trzy pulapki:
zbedny zakres, dane gracza i kodowanie plikow.

## Krok 1: policz, KTO co widzi (zanim cokolwiek zmienisz)
Nie "ile jest wystapien", tylko **w czym one siedza**. Jedno polecenie klasyfikujace linie
(menu edytora / atrybut / komentarz / napis w interfejsie) potrafi zmienic decyzje o zakresie.

W tym projekcie z 338 wystapien w kodzie **jedno** widzial gracz (stopka w oknie tworcow);
297 to byly sciezki menu edytora, widoczne wylacznie dla autora gry. "Rebranding calej gry"
okazal sie w praktyce rebrandingiem warsztatu. Bez tego pomiaru cala robota wygladalaby na
ryzykowna zmiane produktu, a byla kosmetyka narzedziowa.

## Krok 2: oddziel nazwy WIDOCZNE od nazw NOSNYCH
Nazwa nosna to taka, od ktorej cos zalezy poza estetyka:
- **nazwa produktu** - w Unity wyznacza folder zapisow (`Application.persistentDataPath`).
  Jej zmiana **odcina graczy od istniejacych zapisow**. Prawie nigdy nie warto.
- **nazwa pliku wykonywalnego, biblioteki kodu, klasy budujacej, folderu projektu** - siedza
  w poleceniach bramki CI/buildu, sciezkach raportow i dokumentacji. Zysk dla gracza: zerowy
  albo nazwa pliku exe. Domyslnie NIE ruszac.
- **sciezki menu edytora, komentarze, napisy w interfejsie** - czysta kosmetyka, zmieniac swobodnie.

Osobno sprawdz **znaki niedozwolone w sciezkach**: dwukropek w "MARKA: Podtytul" jest w Windows
nielegalny w nazwie folderu. Jesli nazwa trafia do sciezki, wersja z dwukropkiem moze byc tylko
stylizacja logo (a logo zwykle i tak jest obrazkiem).

## Krok 3: podmieniaj na BAJTACH, nie na tekscie
Podejscie tekstowe (`open(p).read()`) wywraca sie na pierwszym pliku w innym kodowaniu - i to
**po przepisaniu czesci plikow**, wiec zostawia projekt w polowie zmieniony. Dodatkowo grozi
zdjeciem znacznika BOM i zamiana koncow linii, co robi z jednej podmiany diff na caly plik.

Jesli szukany napis jest czysto ASCII (a nazwy produktow zwykle sa), podmiana bajtowa jest
identyczna w kazdym kodowaniu i nie dotyka niczego innego:

    d = open(p, 'rb').read()
    open(p, 'wb').write(d.replace(b'Stara Nazwa', b'Nowa Nazwa'))

Wbuduj **straznik**: liczba wystapien nazw wewnetrznych (`Stara_Nazwa`, `StaraNazwa`) przed
i po musi byc identyczna. Wariant z podkreslnikiem i bez spacji to zwykle wlasnie te nosne.

## Krok 4: weryfikacja po zmianie (cztery pytania)
1. Czy ktorys plik **stracil BOM** wzgledem stanu z repozytorium?
2. Czy roznice sa **drobne i symetryczne**? Plik z diffem "cala tresc" = zepsute konce linii.
3. Czy podmiana **nie dotknela kluczy trwalych** (klucze zapisu, PlayerPrefs, sciezki plikow)?
   Przefiltruj dodane linie z pominieciem menu i komentarzy - powinno zostac kilkanascie
   swiadomych trafien, ktore da sie przeczytac okiem.
4. Czy **build i testy** przechodza? Zmiana atrybutow kompilowanych to zmiana kodu.

## Pulapki specyficzne dla Unity
- `[CreateAssetMenu(menuName = ...)]` mozna przemianowac bez ryzyka: istniejace pliki `.asset`
  trzymaja identyfikator skryptu, a nie sciezke menu. Warto to potwierdzic wyszukiwaniem.
- **Komunikaty odsylajace do menu** ("uruchom Narzedzia/Setup/...") musza isc razem z
  przemianowaniem - inaczej instrukcja w logu wskazuje pozycje, ktorej juz nie ma.
- Korzen menu edytora z dluga nazwa marketingowa zabiera duzo miejsca w pasku. Warto rozwazyc
  krotki alias marki jako korzen.

## Czego nie zmieniac
**Archiwow i raportow historycznych.** Dokument opisujacy stan sprzed zmiany nazwy jest
swiadectwem, a nie tekstem marketingowym - przepisany klamie o tym, co wtedy istnialo.

## Related
- [[build-is-the-only-truth-editor-lies]] - zmiana atrybutow to zmiana kodu, wiec konczy sie buildem
