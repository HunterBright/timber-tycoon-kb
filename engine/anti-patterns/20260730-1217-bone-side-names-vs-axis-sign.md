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

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260531-0934-humanoid-orientation-from-armature-not-bbox|Determine a humanoid's up/forward axis from ARMATURE bone landmarks, not from bounding-box max-spread - a T-pose arm span can beat true height]] - wspolne: armature, blender
- [[20260731-1050-rowne-krawedzie-ubran-bisect-plane|20260731-1050-rowne-krawedzie-ubran-bisect-plane]] - wspolne: bmesh, blender
- [[20260531-1530-unity-humanoid-autorig-mirrored-foot|Crooked foot under Unity Humanoid = auto-rig copied the foot bind pose instead of mirroring it]] - wspolne: rig, blender
- [[20260725-1015-ai-autorig-proportions-crush-humanoid|Szkielet z auto-rigu AI ma inne proporcje niz siatka: postac w grze skladasie w harmonijke]] - wspolne: rig, blender
- [[20260612-1200-eevee-shadow-acne-wavy-lines|Wavy dark lines in EEVEE preview renders = shadow acne, not geometry]] - wspolne: debugging, blender
- [[20260610-1820-blender-mcp-failure-headless-fallback|blender-mcp bridge failure modes + headless CLI fallback]] - wspolne: debugging, blender
<!-- /POWIAZANE:auto -->
