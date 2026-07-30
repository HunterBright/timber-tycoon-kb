---
title: 'Lekcja: licznik liczący PRZYROSTY z eventu magazynu fałszywie nalicza przy wczytaniu zapisu'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-06-22'
project: Kerf - Sawmill Tycoon
tags:
- save-system
- ISaveable
- load-order
- dontdestroyonload
- events
- race-condition
- unity
applies_to: []
source: ''
severity: high
promoted: '2026-07-30'
---

# Lekcja: licznik liczący PRZYROSTY z eventu magazynu fałszywie nalicza przy wczytaniu zapisu

## Kontekst
`ProductionTracker` zlicza dożywotnią produkcję, nasłuchując `StorageManager.OnStorageChanged`
i sumując DODATNIE przyrosty stanu magazynu. Problem: ten sam event odpala się też, gdy regały
ODTWARZAJĄ swoją zawartość podczas wczytywania zapisu - co wyglądało jak „świeża produkcja".

## Dlaczego to trudne (dwie pułapki naraz)
1. **DontDestroyOnLoad utrwala stan między wczytaniami.** Tracker przeżywał reload sceny z flagą
   `baselineReady = true` (ustawioną w poprzedniej sesji), więc gate „nie licz dopóki baza nie gotowa"
   NIE chronił przy drugim/kolejnym wczytaniu w tej samej sesji gry.
2. **Kolejność `ISaveable.LoadSaveData` nie jest gwarantowana.** SaveManager iteruje subskrybentów
   w kolejności rejestracji. Jeśli regały odtworzą zawartość (i odpalą event) ZANIM tracker wczyta
   swój stan - przyrosty zostaną policzone. A dla STARYCH zapisów (bez klucza trackera) jego
   `LoadSaveData` w ogóle nie jest wołane, więc nic tych fałszywych liczb nie wyczyści.

## Reguła (transferowalna)
**Jeśli system liczy przyrosty/zdarzenia, które odpalają się także podczas przywracania stanu z zapisu -
musisz wyciszyć liczenie na CAŁY czas wczytywania, niezależnie od kolejności ISaveable.**
Nie polegaj na fladze „gotowości" ustawianej per-instancja (przeżywa DontDestroyOnLoad) ani na tym,
że `LoadSaveData` twojego systemu wykona się przed innymi.

## Rozwiązanie zastosowane
- `SaveManager` ma synchroniczny `Load()` z prywatną flagą `isLoading` na czas całej operacji.
  Wystawiono read-only getter `public bool IsLoading => isLoading;` (zero zmian zachowania).
- W handlerze eventu: `if (loading) { zaktualizuj bazę; return; }` - podczas wczytywania baza
  podąża za przywracanym stanem, ale licznik nie rośnie. Po wczytaniu przyrosty liczą się od poprawnej bazy.
- Bonus: zależny system tworzy się przez `GetOrCreate()` (wzorzec samo-bootstrapu), żeby subskrypcja
  nigdy nie zginęła przez kolejność inicjalizacji.

## Jak wykryć w innym projekcie
Każdy „lifetime/cumulative counter" karmiony eventem stanu (magazyn, ekwipunek, waluta liczona z delty
salda) + system zapisu, który przywraca ten stan przez ten sam event → sprawdź zachowanie przy
WCZYTANIU PO ROZEGRANIU (nie tylko świeży start), zwłaszcza dla starych zapisów bez danych licznika.
