---
title: Brakujący klucz w assecie NIE oznacza zera - zmierz, zanim "naprawisz"
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-22'
project: Kerf - Sawmill Tycoon
tags:
- unity
- serialization
- scriptableobject
- debugging
- measurement
- build-probe
applies_to: []
source: ''
severity: medium
promoted: '2026-07-30'
---

# Brakujący klucz w assecie NIE oznacza zera - zmierz, zanim "naprawisz"

## Sytuacja

Zgłoszenie: kłody wpadają pod teren, "zwłaszcza klon". Przegląd plików `.asset` typów drzew
pokazał, że `TreeType_Maple` i `TreeType_Oak` **nie mają w ogóle klucza** `logSpawnYOffset`,
podczas gdy pozostałe osiem gatunków ma `logSpawnYOffset: 1`. Wniosek nasuwał się sam:
brakujący klucz = 0 = kłody rodzą się w gruncie. Gotowa diagnoza i gotowa łatka.

## Co pokazał pomiar

Sonda uruchomiona w PRAWDZIWYM buildzie odczytała wartość z żywych obiektów:
**1,00 m dla wszystkich dziesięciu gatunków.** Diagnoza była fałszywa.

Unity przy deserializacji ScriptableObjectu najpierw konstruuje obiekt (czyli wykonuje
inicjalizatory pól z kodu, tu `public float logSpawnYOffset = 1.0f;`), a dopiero potem
nakłada wartości z YAML-a. Klucz, którego w pliku nie ma, zostawia wartość z KODU.

Zerowanie dotyczy innego przypadku: pól typu tablica/lista dodanych do już istniejącego
komponentu w scenie, które potrafią wczytać się jako PUSTE. Stąd bierze się mylące
uogólnienie "nowe pole = wartość domyślna typu".

## Druga fałszywa hipoteza tego samego dnia

Drugim podejrzanym był losowy impuls przy spawnie kłód:
`rb.AddForce(Vector3.up * 2f + Random.insideUnitSphere * 0.5f, ForceMode.Impulse)`.
Wygląda groźnie, dopóki się nie policzy: `ForceMode.Impulse` dzieli przez masę, więc
2 Ns na ciele o masie 50 kg daje **0,04 m/s**. Nic nikogo nie wystrzeli.

## Reguła

Zanim naprawisz "oczywistą" przyczynę wynikającą z czytania plików albo kodu, **odczytaj
tę samą wartość w działającej grze**. Plik i kod opisują intencję; runtime opisuje stan.
Przy podejrzeniach o fizykę policz jednostki, zanim uznasz liczbę za dużą lub małą.

Praktyczny skutek dla projektu z sondą buildową: check, który wypisuje ZMIERZONE wartości
obok wyliczonego niezależnie oczekiwania, kosztuje kilkanaście linijek i od razu rozstrzyga
takie hipotezy - zamiast trafiać do commita jako "naprawa" czegoś, co działało.

## Rozwiązanie właściwego problemu

Przyczyna przeciskania się kłód przez teren pozostała nieznana, więc zamiast zgadywać
powstała siatka bezpieczeństwa: co sekundę zapamiętywane jest miejsce spoczynku każdego
ciała z fizyką i przy spadku poniżej twardej granicy obiekt wraca. Skutek przestał boleć
gracza, mimo że przyczyna nadal nie jest zdiagnozowana - to uczciwy stan, byle był nazwany.
