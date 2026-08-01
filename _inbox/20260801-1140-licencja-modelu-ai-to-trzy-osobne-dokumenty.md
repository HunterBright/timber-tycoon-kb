---
title: Licencja modelu AI to trzy osobne dokumenty i wystarczy, ze jeden zabroni
type: anti-pattern
status: draft
confidence: high
verified: '2026-08-01'
tags:
- licencje
- prawo
- modele-ai
- generowanie-3d
- ue
date: '2026-08-01'
project: GameDevOS
source: https://huggingface.co/TencentARC/Pixal3D
suggested-category: process/anti-patterns
applies_to:
- modele generatywne
- firma w Unii Europejskiej
---

# Licencja modelu AI to trzy osobne dokumenty i wystarczy, ze jeden zabroni

## Pulapka

Sprawdzasz, czy wolno uzyc modelu AI komercyjnie. Patrzysz na etykiete licencji
przy repozytorium na GitHubie. Widzisz „MIT". Uznajesz sprawe za zamknieta
i budujesz na tym potok.

Kusi, bo etykieta jest widoczna, jednoznaczna i stoi w miejscu, w ktorym przy
zwyklym oprogramowaniu naprawde konczy sie temat.

## Dlaczego zawodzi

Przy modelu AI **nie ma jednej licencji**. Sa co najmniej trzy dokumenty
i kazdy moze powiedziec cos innego:

1. **Licencja kodu** - plik `LICENSE` w repozytorium z kodem uruchamiajacym model.
2. **Licencja wag** - osobny dokument przy samym modelu, zwykle na Hugging Face.
   To wagi sa tym, czego naprawde potrzebujesz, i to one czesto maja wlasne warunki.
3. **Karta modelu i bramka pobierania** - naglowek pliku `README.md` przy wagach,
   ktory potrafi **technicznie zablokowac pobranie z calego terytorium**,
   niezaleznie od tego, co mowi licencja.

Do tego dochodzi czwarty, najlatwiejszy do przeoczenia: **licencje podmodulow
i zaleznosci**, ktore instalujesz razem z modelem.

Wystarczy, ze jeden z tych dokumentow zabroni, i calosc odpada.

## Objawy, po ktorych poznasz, ze w to wpadles

Konkretne, zebrane przypadki:

- **Etykieta mowi MIT, a pobranie jest zablokowane dla Unii Europejskiej.**
  Karta modelu ma w naglowku `extra_gated_eu_disallowed: true`. Prawnie wolno,
  technicznie nie dostaniesz wag. W pliku licencji na GitHubie nie ma o tym ani slowa.
- **Etykieta mowi Apache 2.0, a model jest pochodna czegos zakazanego.**
  Karta deklaruje Apache, a obok stoi pole `base_model` wskazujace na model,
  ktorego licencja rozciaga sie na wszystkie prace pochodne.
- **Etykieta na Hugging Face mowi „openrail", a plik licencji mowi
  „tylko do celow badawczych".** Etykieta to wybor z listy rozwijanej, wiec bywa
  przyblizeniem. Wiaze tekst, nie etykieta.
- **Kod jest na MIT, ale na sztywno pobiera model bazowy z licencja niekomercyjna.**
  Twoj wklad jest wolny, a bez tamtego modelu bezuzyteczny.
- **Opcjonalny podmodul do podgladu ma licencje badawcza.** Instaluje sie
  domyslnie w gotowych paczkach, choc do samego generowania nie jest potrzebny.
- **Repozytorium w ogole nie ma pliku licencji**, mimo ze README twierdzi „MIT".
  Brak licencji formalnie oznacza „wszelkie prawa zastrzezone".

## Co robic zamiast

**Czytaj pliki, nie etykiety, i czytaj wszystkie trzy miejsca.** Konkretnie:

1. Pobierz **tresc** pliku licencji z repozytorium kodu, nie jego nazwe.
2. Pobierz **karte modelu** przy wagach i przeczytaj jej naglowek. Szukaj bramek
   terytorialnych i pola wskazujacego model bazowy.
3. Sprawdz licencje podmodulow i zaleznosci, ktore faktycznie uruchamiasz.
4. Szukaj konkretnych zwrotow: wykluczenia terytorialnego, klauzuli
   niekomercyjnej, progu liczby uzytkownikow albo przychodu, zakazu trenowania
   innych modeli, wymogu pokazania cudzego znaku w twoim produkcie.
5. **Cytuj znalezione zdanie**, nie streszczaj go. Streszczenie gubi wlasnie
   te warunki, ktore przesadzaja.

Warto to zautomatyzowac, bo to jest praca mechaniczna. Ale przy pisaniu takiego
automatu pamietaj o dwoch pulapkach, ktore zamienia go w bezuzyteczny:

- **Licencje bywaja lamane na linie i wklejane jako komentarz.** Wzorzec szukajacy
  frazy w jednej linii przegapi zdanie przeciete koncem wiersza. Sklej tekst
  do jednej linii i usun znaki komentarza, zanim zaczniesz szukac.
- **Falszywy alarm kosztuje tyle samo co przeoczenie.** Slowo „non-commercial"
  wystepuje w standardowym tekscie licencji AGPL w zwrocie „occasionally
  and noncommercially", ktory dotyczy przekazywania kodu, a nie zakazu handlu.
  Wystepuje tez w zdaniu „applies to commercial and non-commercial products
  alike", czyli w znaczeniu dokladnie odwrotnym. Narzedzie, ktore krzyczy
  na czyste licencje, zostanie wylaczone po tygodniu i wtedy nie zlapie tej
  jednej, ktora naprawde wyklucza.

## Dowod

Wykluczenie terytorialne w rodzinie Hunyuan3D: *„Territory shall mean the
worldwide territory, excluding the territory of the European Union, United
Kingdom and South Korea"*, obecne w wydaniach 2.0, 2.1 oraz w najnowszym
Hunyuan3D-Part z grudnia 2025. Zgloszenie z prosba o dopuszczenie UE otwarte
od lipca 2025 bez odpowiedzi.

Rozjazd miedzy licencja a bramka pobierania: Pixal3D ma `license: mit`
i jednoczesnie `extra_gated_eu_disallowed: true` w karcie modelu.

Rozjazd miedzy etykieta a trescia: model Roblox Cube ma na Hugging Face etykiete
„openrail", a w pliku licencji zapis *„Permitted Purpose means for academic
or research purposes only"*.

## Rzecz, o ktorej nie pisze zadna licencja

Nawet gdy wszystkie trzy dokumenty pozwalaja sprzedac wynik, **czysty wytwor AI
moze w ogole nie podlegac ochronie prawnoautorskiej**. Wolno go uzyc, ale nie da
sie zablokowac konkurencji uzywajacej bardzo podobnego. Ochrone daje dopiero
wlasna praca nad wynikiem: retopologia, szkielet, animacje, wlasne tekstury.
Dlatego warto zapisywac historie edycji jako dowod wkladu tworczego.

## Powiazane

- [[MAPA-LOW-POLY]]
- [[STAN-RYNKU]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260801-0825-licencja-marki-nie-jest-licencja-produktu|Licencja marki nie jest licencja produktu]] - wspolne: ue, licencje
- [[20260801-0826-trellis-2-generator-3d-bez-blokady-ue|TRELLIS.2 jako generator 3D bez blokady licencyjnej w UE]] - wspolne: ue, licencje
- [[20260725-0625-ai-model-community-license-excludes-eu|Nie stawiaj pipeline'u assetow na modelu AI z licencja "Community", nie czytajac pierwszej linii LICENSE]] - wspolne: prawo, licencje
<!-- /POWIAZANE:auto -->
