---
title: Nie zgaduj strony ciała ze znaku osi ani z nazwy kości (.L/.R)
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-07-30'
project: Kerf - Sawmill Tycoon
tags:
- blender
- rig
- bmesh
- armature
- symmetry
- weights
- debugging
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Nie zgaduj strony ciała ze znaku osi ani z nazwy kości (.L/.R)

## Anty-wzorzec

W skrypcie operującym na siatce per strona ciała (chirurgia nóg w bmesh)
przyjęto `x_str = +1 dla "L", -1 dla "R"` - założenie, że kości `.L` leżą
na +X. W tym rigu było ODWROTNIE (`.L` = -X, zmierzone: średnie x wierzchołków
ważonych na kości .L wynosiło -0.127).

## Skutek (kaskada, ~2 h debugowania)

- przebieg "L" ciął i mostkował nogę +X, przypisując nowym pierścieniom
  wagi LEWYCH kości (skórowanie na krzyż),
- pierścienie artykulacyjne lądowały na jeszcze niesprzątanej drugiej nodze,
- przebieg "R" kasował te świeże pierścienie razem ze swoim pasem usuwania,
- log twierdził "utworzono 14 wierzchołków", a w ZAPISANEJ siatce ich nie było
  (utworzone naprawdę, skasowane później) - log operacji to nie stan pliku.

## Reguła

1. Stronę X mierz z geometrii: `sign(średnie x wierzchołków o wagach kości
   tej strony)` - nigdy z nazwy kości ani konwencji.
2. Bezpiecznik na KOŃCU potoku sprawdza stan zapisywanej siatki (np. "pierścień
   istnieje po OBU stronach"), nie tylko wynik pojedynczej operacji.
   Log "utworzono" nie dowodzi, że przetrwało do zapisu.

## Bonus z tej samej sesji

Dwie powłoki (ciało + ubranie-kopia) tnące nieplanarne czworokąty po RÓŻNYCH
przekątnych przy renderze/deformacji = pozorne "dziury w ubraniu". Leczenie:
jawna triangulacja regionu w źródle - kopia dziedziczy te same trójkąty.
