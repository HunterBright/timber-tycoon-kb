---
type: anti-pattern
project: Timber_Tycoon
suggested-category: workflow/anti-patterns
tags: [art-style, character, blender, procedural, hunyuan, asset-pipeline, qa]
date: 2026-07-17
status: draft
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
