---
title: Licencja marki nie jest licencja produktu
type: anti-pattern
status: draft
confidence: high
verified: 2026-08-01
tags: [licencje, ai, generatory-3d, ue, due-diligence]
date: 2026-08-01
project: GameDevOS
source: https://raw.githubusercontent.com/Tencent-Hunyuan/Hunyuan3D-2.1/main/LICENSE
applies_to: [wybor-narzedzi, generatory-3d, modele-otwarte]
---

# Licencja marki nie jest licencja produktu

## Pułapka

Duzy dostawca AI ogłasza zmianę licencji na permisywną i zniesienie ograniczeń
terytorialnych. Prasa branżowa powtarza to nazwą marki: "Tencent Hunyuan
przechodzi na Apache 2.0, koniec blokady UE". Skoro raz odrzuciłeś narzędzie
tej marki przez licencję, odruch jest naturalny: przeszkoda zniknęła, wracamy
do tematu.

Kusi, bo:
- nazwa się zgadza,
- kierunek zmiany jest prawdziwy (licencja faktycznie się zmieniła),
- źródła są wiarygodne (VentureBeat, serwisy branżowe),
- jest ich wiele i wszystkie mówią to samo.

## Dlaczego zawodzi

Duzi dostawcy wypuszczają pod jedną marką **wiele niezależnych produktów
z osobnymi plikami licencji**: model językowy, generator obrazu, generator 3D,
model świata. Zmiana licencji dotyczy jednego repozytorium, nie marki.

Konkretnie: 6 lipca 2026 Tencent wydał **Hy3** - model **językowy** na 295 mld
parametrów - na Apache 2.0, znosząc wykluczenie UE, Wielkiej Brytanii i Korei
Południowej. **Hunyuan3D**, generator siatek 3D, to inny produkt, inne
repozytorium, inny plik licencji. Ten plik ani drgnął.

Dziennikarz opisujący premierę modelu językowego nie ma powodu wspominać
o generatorze 3D. To czytelnik skleja dwie rzeczy w jedną, bo pamięta markę,
a nie numer produktu.

## Objawy

- Cytujesz **artykuł prasowy** albo podsumowanie wyszukiwarki zamiast pliku
  `LICENSE` z repozytorium.
- W jednym zdaniu pojawia się nazwa marki bez numeru wersji produktu
  ("Hunyuan już wolno w UE").
- Uzasadnienie brzmi "wszędzie piszą, że zmienili licencję".
- Nie potrafisz wskazać commita ani daty zmiany w tym konkretnym repozytorium,
  z którego chcesz korzystać.

## Co robić zamiast

1. **Otwórz surowy plik licencji z gałęzi głównej tego repozytorium, z którego
   pobierasz wagi.** Nie stronę projektu, nie README, nie artykuł.
   Wzór: `raw.githubusercontent.com/<org>/<repo>/main/LICENSE`.
2. **Szukaj w nim słowa "Territory".** Klauzula wykluczająca ma postać:
   *"Territory shall mean the worldwide territory, excluding the territory of
   the European Union, United Kingdom and South Korea"*.
3. **Sprawdź datę ostatniej zmiany pliku.** Jeśli licencja miała się zmienić
   w lipcu, a plik nie był ruszany od czerwca zeszłego roku - nie zmieniła się.
4. **Zajrzyj do zgłoszeń (issues) o licencję.** Otwarte zgłoszenie bez odpowiedzi
   producenta to mocniejszy dowód stanu faktycznego niż dziesięć artykułów.
5. Dopiero potem oceniaj jakość narzędzia. Licencja jest bramką, jakość jest
   drugim krokiem.

## Dowód

Stan na 2026-08-01, sprawdzony bezpośrednio w plikach źródłowych:

- `Hunyuan3D-2.1/LICENSE` zawiera klauzulę **wykluczającą UE, Wielką Brytanię
  i Koreę Południową**. Dodatkowo otwiera się zdaniem pisanym wersalikami:
  *"THIS LICENSE AGREEMENT DOES NOT APPLY IN THE EUROPEAN UNION, UNITED KINGDOM
  AND SOUTH KOREA"*.
- Zgłoszenie [#94](https://github.com/Tencent-Hunyuan/Hunyuan3D-2.1/issues/94)
  z prośbą o dopuszczenie UE jest **otwarte od 7 lipca 2025 i bez odpowiedzi
  Tencenta** - ponad rok.
- Doniesienie o Apache 2.0 dotyczy modelu językowego Hy3 z 6 lipca 2026
  ([VentureBeat](https://venturebeat.com/technology/tencents-apache-licensed-hy3-takes-on-glm-5-2-at-half-the-size-and-wins-everywhere-except-coding)).

Koszt pomyłki, gdyby jej nie wychwycić: zbudowanie potoku produkcyjnego
na generatorze, którego licencja nie obejmuje kraju, w którym wydajesz grę.
Wykrycie tego po premierze oznacza wymianę wszystkich zasobów.

## Czy to przeniesie się na inny projekt

Tak, i to poza AI. Reguła "sprawdź plik licencji tego konkretnego artefaktu,
którego używasz, a nie komunikat o marce" działa tak samo przy silnikach,
bibliotekach, czcionkach, paczkach dźwięków i zasobach z targowisk.
Im większy dostawca, tym więcej produktów pod jedną nazwą i tym łatwiej
o tę pomyłkę.

## Powiązane
- [[MAPA-LOW-POLY]]
- [[20260801-0826-trellis-2-generator-3d-bez-blokady-ue]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260801-0826-trellis-2-generator-3d-bez-blokady-ue|TRELLIS.2 jako generator 3D bez blokady licencyjnej w UE]] - wspolne: ue, generatory-3d, licencje
<!-- /POWIAZANE:auto -->
