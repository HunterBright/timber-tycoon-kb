---
title: Cztery pytania, ktore trzeba zadac regulaminowi kazdej uslugi generatywnej, zanim jej wynik trafi do gry
type: pattern
status: draft
confidence: high
verified: '2026-08-09'
suggested-category: workflow/patterns
tags:
- licencje
- ai
- generatory-3d
- asset-pipeline
- prawo
- uslugi-chmurowe
date: '2026-08-09'
project: GameDevOS
source: 'Odczytane przeze mnie u zrodla 2026-08-09: https://www.tencentcloud.com/document/product/301/74049 (Tencent Cloud AI Service Terms, Last updated 2026-06-17), https://www.tencentcloud.com/document/product/301/9248 (Terms of Service, tabela podmiotu kontraktujacego), https://www.tripo3d.ai/terms sekcja 5.2.1, https://www.meshy.ai/terms-of-use sekcja 3.2 i 2.9'
applies_to: []
---

# Cztery pytania, ktore trzeba zadac regulaminowi kazdej uslugi generatywnej

## Kiedy stosowac

Zawsze, gdy model, tekstura, animacja albo dzwiek wygenerowany u kogos w chmurze
ma trafic do gry, ktora sprzedajemy. Cena za model jest najmniej wazna liczba
w tej rozmowie i zwykle jedyna, ktora widac na stronie glownej.

## Kroki

Cztery pytania. Odpowiedz na kazde szuka sie w PLIKU regulaminu, nie w opisie
marketingowym, i cytuje sie doslownie.

### 1. Czyj jest wynik

Najgrozniejsze pytanie, bo najczesciej ma **rozne odpowiedzi dla planu darmowego
i platnego**. Zmierzone 09.08.2026 na trzech uslugach:

| Usluga | Plan darmowy | Plan platny |
|---|---|---|
| Tripo | *„Tripo retains all rights (...) as well as all Intellectual Property rights arising therefrom"* | wynik i prawa naleza do klienta |
| Meshy | *„Provider owns all right, title, and interest (...) in and to the AI Customer Output"*, klient dostaje CC BY 4.0, czyli **obowiazek podania zrodla w napisach koncowych** | licencja dla Meshy opisana, wlasnosc klienta **nigdzie nie stwierdzona wprost** |
| Tencent HY 3D | *„you own all rights, title, and interest, including all Intellectual Property Rights (...) in and to (...) the AI Output"* | to samo |

**Wniosek operacyjny: darmowy poziom uslugi generatywnej to nie jest tansza
wersja platnego, tylko inna umowa.** Model wygenerowany za darmo potrafi nalezec
do dostawcy razem z prawami autorskimi.

### 2. Czy trenuja na naszych danych

Szukaj slowa „train". Trzy spotykane odpowiedzi, w kolejnosci od najlepszej:
- **nie, chyba ze wlaczysz zgode** (Tencent, sekcja 3.3: *„Tencent will not use
  User Input to train Tencent's AI models and systems, unless you grant your
  explicit opt-in consent"*),
- **nie dla platnych** (Tripo, sekcja 5.2.2),
- **tak, dla wszystkich poza planem dla firm** (Meshy, sekcja 2.9: *„may use
  Customer Inputs and Customer Outputs from non Enterprise Customers"*).

Ostatni przypadek jest wazny, bo wyglada jak pierwszy: klient placi, wiec
zaklada, ze kupil prywatnosc. Nie kupil.

### 3. Ktory podmiot z nami kontraktuje i jakie prawo obowiazuje

To jest pytanie, ktore rozstrzyga o terytorium, i **odpowiedz bywa odwrotna niz
przy wagach tej samej firmy**. Wagi Hunyuan3D na Hugging Face maja w naglowku
karty modelu `extra_gated_eu_disallowed: true` i wykluczaja Unie Europejska,
a usluga chmurowa tej samej marki ma w tabeli podmiotow:

> Twoja lokalizacja: **Europejski Obszar Gospodarczy, Wielka Brytania, Szwajcaria**
> Podmiot: **Tencent Cloud Europe B.V.**, Buitenveldertselaan 1-5, 1082 VA Amsterdam
> Prawo wlasciwe: Anglia i Walia

Szukaj tabeli o naglowku „Contracting Entity" albo rozdzialu o EOG. Jesli jej nie
ma, poszukaj, czy w ogole istnieje wersja regulaminu dla naszego regionu.

### 4. Co jestesmy winni PO wygenerowaniu

Najczesciej pomijane, bo nie dotyczy pobrania pliku, tylko wydania gry. Dwa
zapisy znalezione 09.08.2026 u jednego dostawcy:
- **obowiazek oznaczenia**: *„If you share or publish AI Output publicly (...)
  you must clearly disclose that the content is AI-generated in a manner that is
  easily visible and understandable to recipients"*,
- **zakaz publikowania porownan**: *„You shall not publish or disclose to any
  third party any performance or benchmark tests of the AI Services or underlying
  models without (...) prior written consent"*.

Pierwszy zwykle nic nie kosztuje, bo sklepy z grami i tak wymagaja oznaczenia AI.
Drugi kosztuje wtedy, gdy planujemy napisac publicznie, ktore narzedzie jest
lepsze. Do audytu wewnetrznego wolno, na blog studia nie.

## Dlaczego to dziala

Bo rozdziela cztery rzeczy, ktore w glowie zlewaja sie w jedno pytanie „czy wolno
tego uzyc komercyjnie". Odpowiedz „tak" na to pytanie jest zgodna z sytuacja,
w ktorej wynik nalezy do dostawcy, trenuja na nim i musimy podac ich nazwe
w napisach koncowych.

## Koszty i kompromisy

Czytanie czterech dokumentow zajmuje kilkadziesiat minut na dostawce. Skrot,
ktory dziala: pobrac surowy tekst regulaminu i przeszukac po slowach `own`,
`train`, `Contracting Entity`, `disclose`, `benchmark`. Zawsze z kontrola na
zdaniu, ktorego tam byc nie moze, bo strona narysowana JavaScriptem potrafi
oddac te sama tresc dla kazdego adresu.

**Zastrzezenie, ktore trzeba postawic uczciwie:** to jest lista pytan do zadania,
a nie porada prawna. Zaden z tych zapisow nie daje gwarancji, ze wynik nie okaze
sie podobny do cudzego chronionego modelu. Tencent mowi to wprost i wprost
odmawia zabezpieczenia (*„indemnity"*) na taki wypadek.

## Warianty

Przy **wagach modelu do pobrania** te cztery pytania sie nie stosuja - tam
odpowiedzi daje plik licencji i pole `extra_gated_eu_disallowed` w karcie modelu.
To sa dwa rozne produkty pod ta sama nazwa i **czesto maja rozne warunki**.
Trzecim produktem bywa kod na GitHubie, tez na wlasnej licencji.

## Dowod, ze zadzialalo u nas

Zastosowane 09.08.2026 do Tencent HY 3D Global. Cztery odpowiedzi zamknely watek
otwarty od czterech dni i zamienily „nie wiemy, czy wolno" na „wolno, przy dwoch
zobowiazaniach". Wszystkie cytaty odczytane u zrodla z kontrola na zdaniu
zmyslonym, ktore dalo zero trafien.

## Powiazane
- [[20260808-1120-zasoby-w-dodatkach-blendera-maja-byc-cc0]]
- [[20260808-0940-sprawdzian-ktory-nie-umie-pasc]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260801-0825-licencja-marki-nie-jest-licencja-produktu|Licencja marki nie jest licencja produktu]] - wspolne: ai, generatory-3d, licencje
- [[20260801-1245-regulamin-uslugi-a-licencja-wag-to-dwa-rozne-swiaty|Regulamin uslugi w chmurze i licencja pobieranych wag to dwa rozne dokumenty - sprawdzaj oba]] - wspolne: prawo, generatory-3d, licencje
- [[20260801-0826-trellis-2-generator-3d-bez-blokady-ue|TRELLIS.2 jako generator 3D bez blokady licencyjnej w UE]] - wspolne: generatory-3d, licencje
- [[20260801-1140-licencja-modelu-ai-to-trzy-osobne-dokumenty|Licencja modelu AI to trzy osobne dokumenty i wystarczy, ze jeden zabroni]] - wspolne: prawo, licencje
- [[20260808-1120-zasoby-w-dodatkach-blendera-maja-byc-cc0|Zasoby w dodatkach Blendera maja byc na CC0, a nie na GPL dodatku]] - wspolne: prawo, licencje
- [[20260725-0625-ai-model-community-license-excludes-eu|Nie stawiaj pipeline'u assetow na modelu AI z licencja "Community", nie czytajac pierwszej linii LICENSE]] - wspolne: prawo, licencje
<!-- /POWIAZANE:auto -->
