---
title: Vertex colors vs _BaseColor w Linear color space - ten sam hex renderuje się INACZEJ
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-10'
project: Kerf - Sawmill Tycoon
tags:
- unity
- urp
- linear-color-space
- vertex-color
- materials
- shader
applies_to: []
source: ''
severity: high
promoted: '2026-07-30'
---

# Vertex colors vs _BaseColor w Linear color space - ten sam hex renderuje się INACZEJ

## Objaw
Dopasowywanie koloru materiału (flat `_BaseColor`) do wzorca malowanego vertex colors:
mimo ustawienia IDENTYCZNYCH wartości RGB obiekt renderuje się ~o połowę ciemniej niż
wzorzec. Dwie iteracje "poprawiania palety" nic nie zmieniają (bo problem nie jest w palecie).

## Przyczyna (engine-level)
W projekcie z **Linear color space**:
- **Vertex colors** idą do shadera SUROWE - Unity nie robi na nich żadnej konwersji
  (COLOR semantic = raw float). Shader traktuje je de facto jako wartości liniowe.
- **Kolor property materiału** (typ Color) jest traktowany jako sRGB i przy uploadzie
  na GPU jest konwertowany sRGB->linear (potęga ~2.2).

Efekt: hex 0.42 z siatki = 0.42 liniowo w shaderze; ten sam hex 0.42 wpisany w materiał
= ~0.147 liniowo. Obiekt "z materiału" wychodzi o połowę za ciemny.

## Fix
Przy programowym ustawianiu koloru materiału tak, by GPU dostało dokładnie wartość
z vertex colors wzorca: przypisuj **`targetLinear.gamma`** (Color.gamma = linear->sRGB;
upload odwróci z powrotem do linear):

```csharp
mat.SetColor("_BaseColor", referenceVertexColor.gamma);
```

Diagnoza, która to wykryła: paleta wyciągnięta wprost z mesh.colors wzorca + ten sam
sklonowany shader + ta sama jasność = nadal 2x ciemniej -> jedyną różnicą została
ścieżka wartości do GPU.

## Kiedy pamiętać
Każde "dopasuj flat-color material do obiektu z vertex colors" (i odwrotnie) w projekcie
Linear. Także przy porównywaniu kolorów UI (sRGB) z kolorami świata.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260531-1612-quaternius-lowpoly-nature-urp-import|Importing Quaternius "Stylized Nature MegaKit" (and similar low-poly packs) into URP]] - wspolne: vertex-color, shader, urp
- [[20260702-2140-shader-property-stale-serialized-material-values|Dodanie property do shadera może aktywować STARE, ukryte wartości w materiałach]] - wspolne: shader, materials, urp
- [[20260710-2010-material-props-wrong-shader-inert|Ustawianie właściwości materiału bez sprawdzenia SHADERA = ciche nic + ryzyko nadpisania]] - wspolne: shader, materials, urp
- [[20260720-0940-dwa-niezalezne-bledy-gamma-w-jednym-modelu|"Wszystko za jasne" po podmianie modelu: dwa niezależne błędy gamma dające ten sam objaw]] - wspolne: vertex-color, urp
- [[20260705-0840-blender-linear-color-into-unity-srgb-material|Blender LINIOWE Base Color wpisane wprost w Unity Color property = ~1 gamma za ciemno (projekt Linear)]] - wspolne: shader, materials
- [[20260605-1250-urp-flow-shader-scroll-sign|Scrolling/flow shaders: visual motion runs OPPOSITE to the flow vector]] - wspolne: shader, urp
<!-- /POWIAZANE:auto -->
