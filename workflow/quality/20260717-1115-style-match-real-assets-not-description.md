---
title: 'Anti-pattern: dopasowywanie stylu do OPISU stylu zamiast do prawdziwych assetów z gry'
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-07-17'
project: Timber_Tycoon
tags:
- art-style
- character
- blender
- procedural
- hunyuan
- asset-pipeline
- qa
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Anti-pattern: dopasowywanie stylu do OPISU stylu zamiast do prawdziwych assetów z gry

## Co się stało
Zadanie: nowa postać NPC (Legendarny Drwal) "w stylu pozostałych postaci w grze
(zaokrąglone, flat - styl Schedule 1)". Model zbudowano proceduralnie w Blenderze
(bmesh, faceted low-poly, flat shading) i QA robiono WZGLĘDEM SPECU TEKSTOWEGO
(poza, kolory, proporcje, tri-budget). Wszystkie sondy przeszły. Hunter odrzucił
model natychmiast: "wygląda tragicznie, w żaden sposób nie przypomina tego co mamy".

## Korzeń
Istniejące postacie w grze pochodziły z pipeline'u generatywnego (Hunyuan3D) +
szkielety Mixamo - są GŁADKIE i organiczne. Opis "low-poly, zaokrąglone, flat"
pasuje słownie do obu światów, ale wizualnie to dwa różne języki plastyczne.
Tekstowy opis stylu jest STRATNY - nie niesie informacji o gęstości siatki,
gładkości i "DNA" pipeline'u, z którego powstały istniejące assety.

## Reguła
Przed budową assetu, który ma "pasować do reszty":
1. OTWORZYĆ prawdziwe pliki istniejących assetów tej samej klasy (albo ich rendery)
   i porównać wizualnie - to jest wzorzec, nie zdanie w briefie.
2. Ustalić, JAKIM pipeline'em powstały (procedural? genAI? sklep? scan?) i użyć
   TEGO SAMEGO pipeline'u, chyba że świadomie zmieniamy kierunek artystyczny.
3. QA nowego assetu = render nowego OBOK renderu istniejącego (diff wizualny),
   nie tylko checklist specu.

To samo zjawisko co "kalibruj metryki na znanym-dobrym wzorcu" z diagnostyki
animacji - tylko w warstwie art directionu.

## Sygnatura
Asset przechodzi wszystkie techniczne bramki (poly budget, materiały, sondy),
a użytkownik odrzuca go w 5 sekund słowami "to nie wygląda jak nasza gra".

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260717-0010-generated-rig-bone-axis-defect-skeleton-transplant|Rigi z generatorów AI (Hunyuan): osie kości rozjechane z frontem modelu = wykrzywiona stopa w retargecie; lek = przeszczep szkieletu w Blenderze]] - wspolne: hunyuan, blender
- [[20260725-2320-fartuch-skinning-srednia-dwoch-ud-daje-zero|Fartuch ważony po połowie na oba uda NIE RUSZA SIĘ przy chodzie]] - wspolne: character, blender
- [[character-pipeline-tripo-mixamo-unity|Character pipeline: Tripo mesh → Mixamo rig → Unity (clean, working recipe)]] - wspolne: character, blender
- [[20260531-1500-mixamo-clean-mesh-extraction|Extract a clean mesh-only FBX from a rigged source for Mixamo re-rig]] - wspolne: character, blender
- [[20260711-1647-blender-prop-contact-interpenetrate-not-gap|Anty-wzorzec: szczeliny powietrza jako ochrona przed z-fightingiem (lewitujące propy)]] - wspolne: qa, blender
- [[20260725-2050-kontrakt-liczbowy-bez-nazw-osi|Wspolny kontrakt liczbowy dla kilku agentow, ktory nie nazywa osi]] - wspolne: asset-pipeline, blender
<!-- /POWIAZANE:auto -->
