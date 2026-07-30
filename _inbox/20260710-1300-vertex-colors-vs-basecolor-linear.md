---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [unity, urp, linear-color-space, vertex-colors, materials, shader]
severity: high
date: 2026-07-10
status: draft
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
