---
title: Domyslne wypychanie z przenikania wystrzeliwuje pojazdy w niebo
type: lesson
status: draft
confidence: high
verified: 2026-07-31
tags: [unity, physx, rigidbody, pojazdy, fizyka, sonda]
date: 2026-07-31
project: Kerf - Sawmill Tycoon
source: zgloszenie testera 2026-07-27 ("zjebalem samohut"), 4 filmy z rozgrywki
applies_to: [unity, dowolny projekt z pojazdem na Rigidbody]
severity: high
time_lost: ok. 2 h
---

# Domyslne wypychanie z przenikania moze wystrzelic pojazd w gore

> **UWAGA - korekta z tego samego dnia.** Ten wpis powstal, gdy podejrzewalem wypychanie
> z przenikania o zgloszony blad "auto jedzie do nieba". **To NIE byla przyczyna tamtego bledu** -
> prawdziwa opisuje [[20260731-1700-trzymanie-boczne-liczone-z-osi-nadwozia-zjada-grawitacje]]. Zgloszonego
> przypadku NIE udalo sie odtworzyc wypychaniem: ani 20 rozbiegow prawdziwym autem na przeszkody,
> ani auto zatopione w bryle nie dalo wystrzalu (z limitem i bez - ten sam wynik).
> Wpis zostaje, bo sam mechanizm i liczba sa zmierzone i realne, ale to zabezpieczenie
> na zapas, a nie diagnoza. Nie cytowac go jako przyczyny latajacych aut.

## Objaw
Cialo dynamiczne wcisniete w statyczna bryle potrafi zostac z niej wystrzelone w gore.
Im ciezsze i im glebiej wcisniete, tym mocniej to widac.

## Mechanizm
`Physics.defaultMaxDepenetrationVelocity` (ProjectSettings > Physics) ma domyslnie **10 m/s**.
To limit predkosci, z jaka silnik fizyki rozpycha dwa ciala, ktore sie w siebie wcisnely.
Dla lekkiej skrzynki 10 m/s jest niewidoczne. Dla nadwozia to kop **36 km/h w gore**.

Ciezar nie chroni: przy wypychaniu ze statycznego collidera cala poprawka idzie w cialo dynamiczne.

## Rozwiazanie
Dwie warstwy, obie tanie:

1. Na Rigidbody pojazdu: `rb.maxDepenetrationVelocity = 2f` (zamiast globalnych 10).
   Wypycha w tym samym kierunku, tylko spokojnie.
2. Bezpiecznik na predkosc: jesli pojazd stal kolami na gruncie w ciagu ostatnich ~0,25 s,
   a nagle ma predkosc w gore ponad prog (u nas 6 m/s), scinamy skladowa pionowa. Takiej
   predkosci nie da sie zebrac na zadnej gorce w mapie, wiec prog nie tyka legalnych najazdow.

Warstwa 2 jest potrzebna, bo warstwa 1 nie lapie kopow z innych zrodel.

## Co NIE zadzialalo
- Szukanie winnego wsrod ladunku (klody na pace) - wizualizacja paki to wypieczone siatki
  bez colliderow, wiec nie ona.
- Istniejaca blokada wspinaczki po przeszkodach (filtr normalnej + bramka gazu, naprawa
  z 24.07). Ona pilnuje NAPEDU, a wystrzal przychodzi od solvera kolizji - inna droga.

## Gotcha przy mierzeniu (wazniejsza niz sama naprawa)
Wypchniecie z przenikania **nie zapisuje sie w `rb.linearVelocity`**. Pierwsza wersja bramki
mierzyla predkosc i pokazywala 0,0 m/s przy bryle, ktora leciala w gore - bramka byla slepa.
Mierz PRZEMIESZCZENIE na krok fizyki (`(y - prevY) / Time.fixedDeltaTime`), bo to widzi gracz.

Po zmianie pomiaru bramka pokazala: bez limitu **10,0 m/s** w gore, z limitem pojazdu **2,0 m/s**.

## Dowod
- Sonda buildowa Kerf, sekcja `Wystrzal/limit` + `Wystrzal/pomiar`.
- Zielony przebieg: `passed 243/243`, exit 0.
- Czerwona probka `-carlaunchredproof` (limit nie jest nakladany): exit 1, `Wystrzal/limit` FAIL.
- Pomiar w obie strony w jednym przebiegu: bryla bez limitu MUSI wystrzelic, inaczej check
  sam zglasza "scenariusz nie powstal".

## Czy to przeniesie sie na inny projekt
Tak, na kazdy projekt Unity z pojazdem albo duzym cialem dynamicznym. Wartosc domyslna 10 m/s
jest globalna i nikt jej nie rusza, dopoki cos nie odleci.

## Lekcja metodyczna (wazniejsza niz sama liczba)
Znalazlem prawdopodobny mechanizm, zmierzylem go i uznalem sprawe za zamknieta - a zgloszony
blad mial INNA przyczyne. Dopiero zdjecia od zglaszajacego (auto lezace NA BOKU) pokazaly wlasciwy
trop. **Zmierzony mechanizm nie jest dowodem, ze to TEN mechanizm.** Dopoki nie odtworzysz
zgloszonego przypadku, mow "zabezpieczylem droge X", a nie "naprawilem blad".

## Powiazane
- [[20260731-1700-trzymanie-boczne-liczone-z-osi-nadwozia-zjada-grawitacje]] - prawdziwa przyczyna tamtego zgloszenia
- [[gate-must-have-provable-failure-mode]]
- [[build-is-the-only-truth-editor-lies]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260731-1700-trzymanie-boczne-liczone-z-osi-nadwozia-zjada-grawitacje|Trzymanie boczne liczone z osi nadwozia zjada grawitacje przewroconemu autu]] - wspolne: fizyka, pojazdy, sonda
<!-- /POWIAZANE:auto -->
