---
title: Mixamo nie ma animacji „obsługi maszyny / pracy fizycznej" — użyj busy-idle + animacji maszyny; dla noszenia jest „Carrying"
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-04'
project: Kerf - Sawmill Tycoon
tags:
- mixamo
- animation
- npc
- unity
- generic-rig
- animator-override
applies_to:
- unity
- mixamo
source: ''
severity: medium
time_lost: ~2h (kilka rund pobierania złych klipów + tuning)
suggested-category: engine/lessons
---

# Mixamo nie ma animacji „obsługi maszyny / pracy fizycznej" — użyj busy-idle + animacji maszyny; dla noszenia jest „Carrying"

## Problem
Potrzebna była animacja „robotnik obsługuje maszynę" (stojąc, ruch rąk) z Mixamo. Szukanie po
`Cranking` / `Lever Pull` / `Using` / `Pulling` zwracało **nic sensownego**. Podobnie brak: sawing,
hammering, chopping, planing, sanding, sharpening, shoveling, stirring, pumping.

## Root cause
Biblioteka Mixamo (~2400 klipów) to **lokomocja / walka / taniec / gesty / idle społeczne** — NIE ma
dedykowanych animacji rzemiosła / obsługi maszyn / obróbki drewna. To nie kwestia złych słów kluczowych
— tych klipów tam po prostu nie ma (potwierdzone researchem 3 agentów po realnej bibliotece + wątki
dev-forum „does Mixamo even have chopping/mining?").

## Solution
1. **Nie szukaj dosłownego czasownika „operuje maszyną" — nie istnieje.** Zamiast tego:
   - Najtaniej: zostaw robotnika w **zwykłym idle**, a „pracę" niech opowiada **własna animacja maszyny**
     (kręcące się części) + pasek/kółko postępu. Człowiek stoi, maszyna żyje — czyta się jako praca.
   - Jeśli musi być ruch rąk: **busy-hands idle/gesture** (`Typing`, `Texting` [VERIFIED], `Talking`,
     `Button Pushing`) zwolniony ~20% — czyta się jako „zajęty przy pulpicie".
2. **Noszenie DZIAŁA:** `Carrying` (cradle na stojąco) + carry-walk („Walking … carrying") istnieją i są
   dobre. Skrzynię/rekwizyt doczep do kości dłoni (`mixamorig:RightHand`), offset dostrój na oko.
3. **Per-rola bez ruszania współdzielonego kontrolera:** `AnimatorOverrideController` na bazowym
   kontrolerze podmienia klipy tylko dla jednej roli (reszta NPC nietknięta). Po podmianie
   `animator.runtimeAnimatorController` w runtime — **re-apply parametry** (rebind resetuje bool-e).

## Gotchas techniczne (Mixamo → Unity Generic rig)
- **Zaznacz „In Place"** przy pobieraniu klipów lokomocji. Bez tego jest root-motion; przy ruchu przez
  NavMeshAgent (`applyRootMotion=False`) pozycja NIE dryfuje, ale stopy **ślizgają się** (moonwalk).
- **Różna liczba kości gra OK:** klip wyeksportowany z palcami (~50 kości mixamorig) odtwarza się
  poprawnie na uproszczonym rigu bez palców (~35 kości) — rdzeń (Hips→Spine→Arm→Hand) pasuje po
  ścieżkach, krzywe palców po prostu nie mają celu (no-op). Nie psuje pozy.
- **Import jako Generic, NIGDY Humanoid** dla tego typu rigu (Humanoid auto-map gnie stopy — osobna
  lekcja). Klip trzyma się szkieletu po ścieżkach kości.

## What didn't work
- Szukanie literalnych czasowników maszynowych na Mixamo (Cranking/Lever/Using/Pulling) — pustka.
- Pierwsze pobrania „Idle"/„Walking" jako carry — okazały się ZWYKŁYM idle/walk (ręce w dół), nie pozą
  trzymania pudła. Weryfikuj pozę zrzutem ekranu, nie nazwą pliku.

## Transferability
Każdy projekt Unity z NPC-ami z Mixamo, który potrzebuje animacji „pracy/rzemiosła" (tartak, fabryka,
kuchnia, sklep). Wniosek uniwersalny: dla „NPC zajęty przy stanowisku" bierz busy-idle/gesture +
animację obiektu, a nie szukaj dosłownej mocap-pracy — jej w Mixamo nie ma.

## Related
- (project) foot-rig: Humanoid auto-map gnie stopy → Generic + mixamorig
- (pattern) AnimatorOverrideController do podmiany klipów per-rola na wspólnym kontrolerze
