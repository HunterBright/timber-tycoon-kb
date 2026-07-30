---
title: 'Anty-pattern: cienkie płaskie „soczewki" jako liście low-poly'
type: anti-pattern
status: draft
confidence: low
verified: ''
date: '2026-07-02'
project: Kerf - Sawmill Tycoon
tags:
- blender
- low-poly
- foliage
- trees
- modeling
applies_to: []
source: ''
suggested-category: engine/anti-patterns
---

# Anty-pattern: cienkie płaskie „soczewki" jako liście low-poly

## Co próbowaliśmy
Drzewo z wielkimi pojedynczymi liśćmi (Teak): każdy liść jako cienka dwustronna soczewka
(płaski obrys + 2 ściany). Licznik wierzchołków świetny, render z jednego kąta wyglądał OK.

## Dlaczego nie działa
Płaskie soczewki ZNIKAJĄ oglądane z boku (edge-on) — drzewo z 15 liśćmi wyglądało jak 5-6.
Dodatkowo automatyczne rozmieszczenie skupiło liście po jednej stronie, więc z drugiej strony
drzewo było gołe. Raport liczbowy (ilość liści, szerokość korony w bbox) PRZECHODZIŁ —
bounding box nie mierzy wizualnej masy.

## Poprawne podejście
1. Liść = niskopoligonowa „poduszka" z realną grubością ~0.10–0.15 m w środku (zbiega do krawędzi)
   — czyta się z każdego kąta, koszt ~18 wierzchołków.
2. Rozmieszczenie równomiernie dookoła pnia (azymut równy + jitter), nie losowo (losowe klastruje).
3. Weryfikacja renderem z DWÓCH kątów (przód/tył), nie jednym — i licz wizualną masę, nie bbox.

## Reguła
W low-poly każda bryła listowia musi mieć objętość 3D. Metryki (verts, bbox) nie zastępują
oglądu z co najmniej dwóch kątów.
