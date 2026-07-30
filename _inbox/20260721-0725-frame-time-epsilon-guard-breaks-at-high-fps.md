---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [unity, frame-rate, time-deltatime, diagnostics, build-verification, floating-point]
severity: high
time_lost: "~2 h (w tym 20 agentów analizy, ktora dala zla odpowiedz)"
date: 2026-07-21
status: draft
applies_to: [unity]
---

# Bezpiecznik `Time.deltaTime > 0.001f` cicho zeruje sygnał, gdy build chodzi szybciej niż 1000 FPS

## Problem

Auta NPC cofały na miejsce parkingowe, a ich przednie koła migotały między prawidłowym kątem
skrętu a pozycją na wprost. Objaw był widoczny **tylko w buildzie** i tylko w tym jednym manewrze.
W Edytorze i w symulacji przy 60 FPS nie odtwarzał się w ogóle.

## Root cause

Kod manewru liczył prędkość jako:

```csharp
float frameSpeed = Time.deltaTime > 0.001f ? Vector3.Distance(pos, prevPos) / Time.deltaTime : 0f;
```

Próg 0,001 s miał chronić przed dzieleniem przez zero. Ale **1 ms to 1000 FPS, a to nie jest
liczba niemożliwa** - prosta scena na mocnym GPU bez vsync ją przekracza. Pomiar w tym buildzie:
średnio **886 FPS, 92,9% klatek z dt <= 0,001 s**. Wszystkie te klatki dostawały twarde zero.

Zero samo w sobie nie bolało - bolało to, że dalej w łańcuchu **tryb rysowania kół był wnioskowany
z tej liczby** (`speedOverride != 0f`), więc przez 93% klatek system uznawał, że manewr nie trwa.
Patrz [[sentinel-value-as-mode-flag]].

## Solution

1. Próg obniżony do `1e-6f` - ma być bezpiecznikiem arytmetycznym, a nie progiem decyzyjnym.
   Bezpiecznik przed dzieleniem przez zero powinien stać przy epsilonie typu, nie przy
   "wygodnej okrągłej liczbie", bo ta okrągła liczba jest w rzeczywistości ukrytym założeniem
   o klatkażu.
2. Tryb wyprowadzony na jawną flagę, żeby nigdy więcej nie zależał od wartości pomiaru.
3. Ogranicznik prędkości zmiany w jedynym miejscu piszącym wyjście wizualne - klasa błędu
   zamknięta niezależnie od źródła.

## What didn't work

**Analiza kodu, i to bardzo dokładna.** Puściłem 5 niezależnych tropów, każdy z adwersarzem
mającym obalić wnioski. Adwersarze **obalili wszystkie hipotezy** - i mieli rację w każdym
pojedynczym argumencie:

- producent sygnału jest filtrowany dolnoprzepustowo (`Lerp` 8/s, tau = 0,125 s), więc nie może
  dać przeskoku w jednej klatce - PRAWDA;
- "dokładne zero z niedokładności float" wymagałoby ~1000 FPS, bo przy 60 FPS przesunięcie na
  klatkę to setki ULP-ów - PRAWDA;
- dane toru są gęstsze niż próg degeneracji, a krzywizna nie zmienia znaku - PRAWDA.

Wspólny błąd: **wszyscy symulowali 60 FPS**, bo taki jest cel wydajnościowy projektu. Nikt nie
zapytał, ile klatek build robi NAPRAWDĘ. I nikt nie przeczytał progu `0.001f` jako "ten kod
zakłada, że gra nie przekroczy 1000 FPS" - a dokładnie tym on jest.

Wniosek proceduralny: gdy N niezależnych analiz zgodnie orzeka "to niemożliwe", a użytkownik
widzi objaw na własne oczy - **to nie użytkownik się myli, tylko wszystkie analizy dzielą
nieuświadomione założenie**. Przestań analizować, zacznij mierzyć.

## How it was actually found

Rejestrator w sondzie buildowej: jedna linia na klatkę, zapis do CSV obok .exe - kąt odczytany
z NARYSOWANEGO transformu, czas klatki, stan maszyny stanów. Pierwsze spojrzenie na kolumnę
`dt` dało odpowiedź w kilkanaście sekund (`92,9% klatek <= 0,001 s`).

Dwie rzeczy, które ten pomiar uratowały:
- **Odczyt z narysowanego stanu, nie ze zmiennych sterownika.** Błędy tej rodziny powstają
  MIĘDZY obliczeniem a narysowaniem, więc pomiar na zmiennej pośredniej byłby ślepy dokładnie
  na to, czego szukamy.
- **Rozbicie wyniku na fazy** maszyny stanów. Bez tego nie widać, czy migocze faza zgłoszona
  przez gracza, czy inna. Tu wyszło, że migotały OBIE - druga nigdy nie została zgłoszona.

Pułapka po drodze: limit próbek (30 000) uciął pomiar **przed** interesującą fazą, a raport
i tak wyglądał sensownie. Limit bufora diagnostycznego musi być albo policzony na najdłuższy
możliwy przebieg, albo (lepiej) statystyki liczone bez limitu, a limitowany tylko zrzut do pliku.

## Transferability

- Każdy silnik z krokiem czasu zmiennym: **każdy próg na `deltaTime` jest ukrytym założeniem
  o klatkażu**. Wypisz je i sprawdź w docelowym buildzie, a nie w edytorze.
- Bezpiecznik przed dzieleniem przez zero należy stawiać przy epsilonie typu (`1e-6`), nigdy
  przy liczbie "wygodnej" - bo wygodna liczba prędzej czy później stanie się osiągalna.
- Sprzęt idzie do przodu, projekt stoi: kod napisany, gdy gra chodziła 60 FPS, zaczyna się
  psuć, gdy zacznie chodzić 900 FPS. To bomba z opóźnionym zapłonem, nie regresja - w git blame
  wygląda niewinnie i data commita nie ma nic wspólnego z datą objawu.
- Diagnostyka wizualnych błędów klatkowych: mierz różnicę między KOLEJNYMI klatkami, w jednostkach
  na sekundę, odczytaną z narysowanego stanu. Metryki typu "maksimum w przebiegu" są na migotanie
  ślepe - migoczący element ma takie samo maksimum jak zdrowy.

## Related

- [[sentinel-value-as-mode-flag]] - antywzorzec, który zamienił to zero w widoczny błąd
- [[measure-in-build-not-in-simulation]]
