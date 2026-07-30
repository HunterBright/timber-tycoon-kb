---
title: 'Wysokosc stop NPC: szacunek z plikow i obrys siatki KLAMIA - prawde mowia kosci stop mierzone w buildzie'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-16'
project: Timber_Tycoon
tags:
- unity
- animation
- generic-rig
- measurement
- mixamo
- skinnedmeshrenderer
- bounds
applies_to: []
source: ''
severity: medium
time_lost: 3 iteracje pomiarowe w jednej sesji (szacunek FK -> obrys siatki -> kosci)
promoted: '2026-07-30'
---

# Wysokosc stop NPC: szacunek z plikow i obrys siatki KLAMIA - prawde mowia kosci stop mierzone w buildzie

## Problem
Czesc NPC z puli kilkunastu modeli (wspolny kontroler Generic, klipy mixamo) wygladala na
chodzaca za nisko. Trzy kolejne metody diagnozy daly SPRZECZNE wyniki:
1. Szacunek FK z YAML prefabow (dlugosc lancucha nog, bind-pose): delty -11..+8 cm.
2. Pomiar w buildzie z minimum SkinnedMeshRenderer.bounds: delty +7..-3 cm, INNE modele.
3. Pomiar w buildzie z kosci palcow stop (transformy *Toe*): CALA pula w ~2 cm od wzorca.

## Root cause
- Szacunek bind-pose FK nie opisuje runtime: Unity przy imporcie Generic z ustawionym root
  node NORMALIZUJE krzywe bioder per awatar - rozne dlugosci nog NIE przekladaja sie wprost
  na zaglebienie stop. Hipoteza "Generic bez retargetu = wspolna wysokosc bioder" byla
  falszywa dla tej konfiguracji importu.
- Minimum obrysu siatki (bounds.min.y, nawet z updateWhenOffscreen=true) potrafi siedziec
  9-18 cm PONIZEJ kosci stop i to trwale (nie tylko w klatkach przejsciowych crossfade'u) -
  obrys laczy w sobie geometrie, marginesy skinningu i pozy posrednie; jako sygnal
  wyrownania stop jest bezwartosciowy.
- Kosci palcow (min world-Y transformow zawierajacych "Toe" w trakcie pelnego cyklu chodu,
  PO rozgrzewce ~0.4 s na crossfade) daja stabilny, porownywalny miedzy modelami sygnal.

## Solution
Sonda w BUILDZIE: spawn kazdego prefabu, wymus stan chodu, odczekaj rozgrzewke, przez >1
cykl klipu zbieraj min Y kosci palcow wzgledem roota; werdykt = delta vs model wzorcowy
(tolerancja 2 cm); obrys siatki loguj tylko diagnostycznie z ostrzezeniem przy rozjezdzie
obrys-vs-kosci > 6 cm. Tabela korekt per model w SO + dodawanie do agent.baseOffset zostaje
jako straznik regresji dla przyszlych modeli - wypelniana WYLACZNIE liczbami z sondy.

## What didn't work
- Bind-pose FK z YAML (przekonujace liczby, zly mechanizm).
- Werdykt z bounds.min.y (artefakty rzedu kilkunastu cm u KAZDEGO modelu).
- Globalny baseOffset (poprzednie sesje) - nie adresuje wariancji per model, o ile ona
  w ogole istnieje (u nas po pomiarze: nie istniala).

## Transferability
Kazdy pipeline z pula postaci: (1) nie wnioskuj o runtime z bind-pose; (2) nie mierz stop
obrysem siatki; (3) mierz kosci w buildzie po rozgrzewce animatora; (4) dwa niezalezne
sygnaly pomiarowe od razu - rozjazd miedzy nimi wykryl artefakt, ktory pojedynczy sygnal
sprzedalby jako fakt.

## Related
- [[20260716-0843-dual-owner-persistence-duplication]] (ta sama sesja)
