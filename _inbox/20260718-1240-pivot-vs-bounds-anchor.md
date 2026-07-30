---
title: transform.position to PIVOT, nie geometria - kotwice wizualne licz z bounds
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-18'
project: Kerf - Sawmill Tycoon
tags:
- unity
- pivot
- bounds
- transform
- anchoring
- vfx
- diagnostics
applies_to: []
source: ''
suggested-category: engine/lessons
---

# transform.position to PIVOT, nie geometria - kotwice wizualne licz z bounds

## Kontekst
Efekt (trociny) mial byc emitowany w miejscu ostrza pily. Kotwiczenie po
`headTransform.position` dwa razy z rzedu dalo zle okno emisji (start za wczesnie,
koniec w polowie trasy) - pivot modelu karetki lezal na jej KONCU, +0.7 jednostki
od faktycznego ostrza. Objaw wygladal jak "bledna logika", a byl czystym offsetem pivota.

## Lekcja
1. Kotwica wizualna (punkt emisji, cel kamery, marker) = **srodek obrysu rendererow**
   (`Renderer.bounds` z Encapsulate, z pominieciem ParticleSystemRenderer), NIE
   `transform.position`. Pivot z FBX potrafi lezec na krawedzi/koncu modelu.
2. Staly offset kotwicy wzgledem prawdy PRZESUWA cale okno logiki zaleznej od pozycji -
   objaw "dziala, ale przesuniete o stala" to niemal zawsze pivot.
3. Po DWOCH nietrafionych poprawkach na slepo: STOP i diagnostyka z dowodem -
   metoda edytorowa w batchmode wypisala liczby (pivot vs srodek obrysu, zakresy na osi)
   i zrobila zrzuty z markerami-kulkami. Korzen znaleziony w jednym przebiegu,
   hipoteza POTWIERDZONA liczbami przed napisaniem poprawki.
