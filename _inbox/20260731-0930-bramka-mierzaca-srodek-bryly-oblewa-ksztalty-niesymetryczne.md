---
title: Sprawdzian celujący w środek bryły oblewa kształty, których masa jest przesunięta
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: engine/lessons
tags:
- testy
- bramki
- fizyka
- collider
- falszywy-alarm
- unity
date: '2026-07-31'
status: draft
confidence: high
verified: '2026-07-31'
source: naprawa sondy MeshReadabilityProbe 2026-07-31
severity: srednia
---

# Sprawdzian celujący w środek bryły oblewa kształty, których masa jest przesunięta

## Objaw

Bramka jakości w buildzie zgłosiła, że dziewięć zderzaków (collider) drzew nie ma kształtu:
„collider PUSTY". Ale bryła otaczająca (bounds) była poprawna i niezerowa, a silnik nie
zgłosił ani jednego błędu o niedostępnych danych siatki.

## Przyczyna

Detektor sprawdzał żywotność zderzaka, strzelając siedmioma promieniami **w środek bryły
otaczającej**: sześć wzdłuż osi i jeden po przekątnej. Dla kształtu zwartego (bryła ziemi,
kłoda, kamień) to wystarcza. Dla drzewa liściastego nie: goły pień jest cienki, korona
przesunięta w bok, więc geometryczny środek bryły wypada **w powietrzu między konarami**
i wszystkie promienie przelatują obok.

Zmierzone na tych samych modelach:

| Model | promienie w środek | promienie po całym przekroju |
|---|---|---|
| akacja, etap średni | 0 z 6 | 32 z 192 |
| heban, etap średni | 0 z 6 | 70 z 192 |
| świerk, etap średni (przechodził) | 4 z 6 | 88 z 192 |

Świerk przechodził wyłącznie dlatego, że jest stożkiem z pniem dokładnie pośrodku.

## Hipoteza, którą warto od razu odrzucić

Pierwsze podejrzenie padło na skalę: modele były autorowane w skali 0,03-0,04 jednostki
i powiększane sto razy na transformie, więc naturalne było przypuszczenie, że silnik fizyki
gubi zbyt drobne trójkąty przy gotowaniu zderzaka. **Sprawdzenie obaliło to w pięć minut:**
po wpieczeniu mnożnika sto w same wierzchołki wynik był identyczny, 0 z 6.

Warto ten eksperyment zrobić, zanim zacznie się przestawiać skalę assetów - zmiana skali
w projekcie z gotowymi prefabami jest droga i nieodwracalna po cichu.

## Poprawka

Dwie ścieżki w detektorze:

1. **Szybka** - dotychczasowe siedem promieni w środek. Kończy sprawę przy pierwszym trafieniu,
   więc dla kształtów zwartych nic nie kosztuje.
2. **Dokładna** - dopiero gdy szybka nic nie znalazła: siatka promieni po **całym przekroju**
   bryły, wzdłuż trzech osi, z obu stron każdej osi, w punkty wsunięte o dziesięć procent
   od krawędzi (żeby promień nie ślizgał się po ścianie bryły otaczającej).

## Warunek, bez którego poprawka jest szkodliwa

Rozluźnienie bramki, żeby przestała świecić na czerwono, jest z definicji podejrzane.
Dlatego poprawce **musi** towarzyszyć test, który:

- buduje kształt z geometrią **omijającą środek** bryły,
- sprawdza, że **wszystkie kierunki starego detektora chybiają** (inaczej test niczego nie dowodzi),
- sprawdza, że nowy detektor mimo to orzeka „żywy".

Pierwsza wersja tego testu była bezwartościowa: klocki leżały na przekątnej (1,1,1), czyli
dokładnie na jednym z siedmiu promieni starego detektora, więc test przechodziłby **także
przed poprawką**. Wykryte przez przestawienie znaku jednej współrzędnej.

Na koniec trzeba uruchomić test z **celowo wyłączoną poprawką** i zobaczyć czerwone światło.
Bez tego kroku nie wiadomo, czy test cokolwiek pilnuje.

## Wniosek przenośny

Każdy sprawdzian typu „czy ten kształt istnieje", który próbkuje **jeden punkt** bryły
otaczającej, ma wbudowany fałszywy alarm dla wszystkiego, co nie jest zwarte: drzew, drabin,
płotów, ram okiennych, obręczy, wsporników. Próbkuj przekrój, nie środek.

I szerzej: **czerwona bramka to hipoteza, nie werdykt.** Zanim ruszysz asset, sprawdź, czy
nie kłamie miernik - bo bramka, która oblewa poprawną pracę, w ciągu miesiąca uczy zespół
ignorować kolor czerwony, a to jest gorsze niż brak bramki.

## Powiązane

- [[gate-must-have-provable-failure-mode]]
- [[build-is-the-only-truth-editor-lies]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260809-1440-bramka-nie-moze-przyznawac-tego-co-sprawdza|20260809-1440-bramka-nie-moze-przyznawac-tego-co-sprawdza]] - wspolne: testy, bramki
- [[20260727-1422-bramka-musi-umiec-zaliczyc-nie-tylko-oblac|Bramka musi mieć udowodniony tryb ZALICZENIA, nie tylko PORAŻKI]] - wspolne: testy, bramki
- [[20260728-1500-bramka-ponad-sufitem|Prog bramki ponad zmierzonym sufitem zamienia kazda runde w porazke]] - wspolne: testy, bramki
- [[20260726-1930-zielone-bramki-nie-dowodza-ze-wyglada-dobrze|Zielona tablica bramek nie dowodzi, ze cos wyglada dobrze]] - wspolne: testy, bramki
<!-- /POWIAZANE:auto -->
