---
title: 'Rodzina assetów współdzieląca teksturę: wybierz strategię (atlas vs kafel) PRZED budową'
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-02'
project: Kerf - Sawmill Tycoon
tags:
- blender
- textures
- uv
- atlas
- tileable
- asset-family
- low-poly
applies_to: []
source: ''
suggested-category: engine/lessons
---

# Rodzina assetów współdzieląca teksturę: wybierz strategię (atlas vs kafel) PRZED budową

## Problem
6 gatunków drzew dostało teksturę kory wypieczoną jako ATLAS (wyspy UV na czarnym tle, Smart UV
Project). Gdy potem powstały kłody/pnie/pniaki tych samych gatunków, atlas nie nadawał się do
owinięcia walca — próba samplowania fragmentu atlasu dała „złote rury" i rozmazy. Skończyło się
podwójną robotą: wygenerowanie dedykowanych kafli + re-eksport WSZYSTKICH modeli drzew z nowym
UV walcowym + sprzątanie 28 osieroconych plików.

## Zasada
Jeśli materiał (kora, metal, tkanina) będzie współdzielony przez RODZINĘ assetów (drzewo + kłoda +
pniak; płot + brama; skrzynia + wieko), od razu twórz go jako **kafel bezszwowy (tileable)** +
proste projekcje UV (walcowa/box), a nie per-model atlas bake. Atlas ma sens tylko dla
pojedynczego hero-assetu, który nigdy nie odda tekstury nikomu innemu.

## Bonus z tej samej lekcji
1. Kafel bezszwowy „tylko poziomo" pokazuje poziomą granicę powtórzenia na wysokich obiektach
   (pień 6.5 m przy 1.1 m/repeat) — od razu rób bezszwowość w OBU osiach.
2. Rodzina wariantów (6 gatunków) generowana jednym parametrycznym generatorem = jeden pattern
   w różnych kolorach; użytkownik to widzi natychmiast. Wyróżnik gatunku musi być STRUKTURALNY
   (inne kształty wzoru), nie kolorystyczny.
