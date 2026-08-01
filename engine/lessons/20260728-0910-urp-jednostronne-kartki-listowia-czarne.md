---
title: Jednostronne kartki listowia na URP/Lit wychodza CZARNE
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-28'
project: Kerf - Sawmill Tycoon
tags:
- unity
- urp
- shader
- foliage
- alpha-cutout
- normals
- backface
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Jednostronne kartki listowia na URP/Lit wychodza CZARNE

## Objaw

Drzewa z kartkami lisci (alpha cutout) renderuja sie prawie czarne, mimo ze
tekstura lisci jest jasna i zdrowa. Sylwetka poprawna, kolor martwy. Wyglada
jak zly material albo za ciemna tekstura - i wlasnie na to sie zwala.

## Przyczyna

Kartki lisci z generatora sa JEDNOSTRONNE: jeden trojkat na kartke, zero par
o przeciwnym obiegu. Zeby byly widoczne z obu stron, material dostaje `_Cull = 0`.

I tu jest pulapka: **URP/Lit rysuje tylna strone, ale NIE odwraca normalnej.**
Piksel tylnej strony liczy swiatlo z normalnej wskazujacej w przeciwna strone,
czyli zawsze "od slonca" - wychodzi sama skladowa otoczenia, praktycznie czern.
Polowa kartek w koronie jest odwrocona tylem do kamery, wiec czernieje polowa
drzewa, a druga polowa zaslania reszte.

## Jak sprawdzic w 30 sekund

Policz w siatce pary trojkatow o tych samych wierzcholkach i przeciwnym obiegu.
Zero par = kartki sa jednostronne = `_Cull = 0` na URP/Lit da czern.

Drugi test: obejrzyj sama teksture (nie model). Jesli tekstura jest jasna,
a model czarny - to jest wlasnie ten przypadek, a nie zla tekstura.

## Rozwiazanie

Shader, ktory odwraca normalna dla tylnej strony. W HLSL to jedna linijka
z semantyka VFACE:

```hlsl
half4 frag(Varyings input, half facing : VFACE) : SV_Target
{
    float3 normal = normalize(input.normalWS) * (facing > 0 ? 1 : -1);
    ...
}
```

Do tego osobny przebieg ShadowCaster z tym samym przycinaniem alfa i `Cull Off`,
inaczej cienie kartek beda pelnymi prostokatami.

## Koszt uboczny, o ktorym trzeba wiedziec

Wlasny shader zwykle nie ma `#pragma multi_compile_instancing`, wiec listowie
traci instancjonowanie GPU. Kora moze zostac na URP/Lit z instancjonowaniem -
podzial na dwa materialy (kora osobno, listowie osobno) oplaca sie takze z tego
powodu, nie tylko wizualnie.

## Czego NIE robic

- Nie podkrecac jasnosci materialu, zeby "rozjasnic liscie". To maskuje objaw:
  strona przednia robi sie przepalona, tylna nadal ciemna.
- Nie wlaczac trybu przezroczystego (alpha blend) zamiast przycinania. Problem
  jest w normalnej, nie w mieszaniu, a dochodzi sortowanie.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260531-1612-quaternius-lowpoly-nature-urp-import|Importing Quaternius "Stylized Nature MegaKit" (and similar low-poly packs) into URP]] - wspolne: foliage, shader, urp
- [[20260605-1250-urp-flow-shader-scroll-sign|Scrolling/flow shaders: visual motion runs OPPOSITE to the flow vector]] - wspolne: shader, urp
- [[20260702-2140-shader-property-stale-serialized-material-values|Dodanie property do shadera może aktywować STARE, ukryte wartości w materiałach]] - wspolne: shader, urp
- [[20260710-1300-vertex-colors-vs-basecolor-linear|Vertex colors vs _BaseColor w Linear color space - ten sam hex renderuje się INACZEJ]] - wspolne: shader, urp
- [[20260623-1508-instanced-grass-cards|Performant stylized grass: textured cards + GPU instancing (no GameObjects)]] - wspolne: foliage, urp
- [[20260710-2010-material-props-wrong-shader-inert|Ustawianie właściwości materiału bez sprawdzenia SHADERA = ciche nic + ryzyko nadpisania]] - wspolne: shader, urp
<!-- /POWIAZANE:auto -->
