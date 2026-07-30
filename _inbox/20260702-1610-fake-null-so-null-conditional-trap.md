---
type: anti-pattern
project: Timber_Tycoon
suggested-category: engine/lessons
tags: [unity, scriptableobject, fake-null, null-conditional, events, nre, missing-asset]
date: 2026-07-02
status: draft
---

# Operator `?.` NIE chroni referencji Unity — brakujący asset SO wybucha NRE w środku kanału eventowego

## Objaw
`NullReferenceException` w `GameEventSO<T>.Raise` przy każdym `EconomyManager.AddMoney`. Stos pokazywał TYLKO ramkę `Raise` (bez ramki listenera), a wywołanie było zabezpieczone `onMoneyChanged?.Raise(Money)`. Efekt kaskadowy: wyjątek przerywał `CustomerOrder.Complete()` PO dodaniu pieniędzy, ale PRZED `isCompleted = true` — sprzedaż "działała wizualnie" (kasa rosła, klient odchodził przez inną ścieżkę), a stan logiczny zostawał rozjechany. Diagnoza zmyliła najpierw w stronę martwych listenerów.

## Przyczyna
Slot Inspectora (`public IntGameEventSO onMoneyChanged`) wskazywał **BRAKUJĄCY asset** (skasowany/zgubiony .asset). Unity tworzy wtedy „fake-null" — zarządzana skorupa istnieje, więc **operatory `?.` i `??` (czysty C#, porównanie referencji) przepuszczają wywołanie**. Metoda wykonuje się na martwej skorupie, której pola (np. `readonly List listeners`) nie zostały zainicjalizowane → NRE na pierwszym dostępie do pola. Tylko przeciążone operatory Unity (`==`, `!=`) wykrywają fake-null.

## Fix (dwupoziomowy)
1. **Centralny strażnik w samej metodzie SO** (chroni WSZYSTKIE call-site'y naraz — u nas 21 miejsc `?.Raise(` w 12 plikach):
   ```csharp
   public void Raise(T value)
   {
       if (this == null || listeners == null) return; // uniowe ==, łapie fake-null/missing
       for (int i = listeners.Count - 1; i >= 0; i--) { ... }
   }
   ```
   Przy okazji: pomijaj i usuwaj martwe wpisy listenerów (`listener == null || (listener is Object uo && uo == null)`).
2. Docelowo napraw dane: podpiąć właściwy asset / wyczyścić slot w Inspectorze.

## Jak diagnozować szybko następnym razem
- Stos z JEDNĄ ramką w metodzie SO + `?.` u wywołującego = pierwszy podejrzany: **fake-null asset**, nie logika metody.
- Rozstrzyga sonda: `Debug.Log(soField == null)` (uniowe ==) vs `soField is null` (C#) — rozjazd tych dwóch = fake-null.
- try/catch z `ex.StackTrace` daje pełny stos tam, gdzie konsola ucina.

## Reguła
W polach Unity (MonoBehaviour/ScriptableObject/Component) **nigdy `?.` ani `??`** — zawsze jawne `if (x != null)` (uniowe). `?.` jest bezpieczne tylko dla czystych typów C#.
