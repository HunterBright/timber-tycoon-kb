---
type: pattern
project: Timber Tycoon
suggested-category: engine/patterns
tags: [unity, choreography, pour, prop-animation, coroutine, low-poly, machine]
date: 2026-07-02
status: draft
---

# Wzorzec: worek wsypuje kawałki do maszyny (bag-pour choreography)

## Problem
Maszyna przyjmuje surowiec "z worka" — potrzebna czytelna, tania animacja wsypywania
bez systemu cząsteczek i bez pełnej fizyki kontenera.

## Rozwiązanie (zwalidowane 2x: Pelletizer chips, FertilizerMaker bark)
1. **Worek = prefab instancjonowany na czas wsypu** przy kotwicy `PourPoint` (dziecko prefabu
   maszyny — designer przeciąga w Scene View). Zdejmij mu fizykę (Destroy collidery+RB),
   warstwa IgnoreRaycast (nie psuje raycastu guzików minigry).
2. **Kotwica Z BOKU otworu, nie nad nim.** Worek bezpośrednio nad otworem wygląda jak
   "dziurawy" (kawałki lecą spod dna). Przechył (Slerp o ~120° wokół lokalnej osi, smoothstep,
   ~1.1 s) ma NIEŚĆ wylot nad otwór.
3. **Emisja kawałków Z WYLOTU worka, śledzona na żywo**: punkt emisji =
   `bag.transform.TransformPoint(0, wysokośćWorka, 0)` co spawn — nie stały punkt.
4. **Emisja startuje po ~1/3 przechyłu** (`InverseLerp(0.35, 1, k)` na progres emisji) —
   nic nie wypada, póki otwór nie celuje w dół. Kawałki porcjami: `target = floor(k*total)`.
5. **Kawałki balistycznie**: RB z grawitacją + losowy angularVelocity (koziołkowanie),
   collider WYŁĄCZONY, kierunek = (środek maszyny − wylot) + stożek ~5°. Znikają poniżej
   linii Y wewnątrz maszyny (jeśli wnętrze zamknięte — "znika za krawędzią" czyta się dobrze
   i jest darmowe). Lokalny recykling instancji (Queue) zamiast globalnej puli.
6. **Cleanup odporny na abort**: worek + kawałki trzymane w polach, niszczone w
   EndSessionCleanup ORAZ w abort PRZED StopAllCoroutines (korutyna wsypu ginie w pół drogi
   — sprzątanie nie może być w niej).

## Kiedy stosować
Każda maszyna/kontener z widocznym załadunkiem sypkiego surowca w grze low-poly.

## Anti-gotcha
Nie podpinaj się pod cudzą współdzieloną pulę obiektów (u nas WoodChunkPool wspólny
Rębak+Peleciarka, rejestrowany w Service Locatorze PO TYPIE — druga instancja koliduje).
Mały lokalny recykling per sesja wystarcza przy ~10 obiektach.
