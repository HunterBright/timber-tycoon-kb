---
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: engine/lessons
tags: [urp, ssao, ambient-occlusion, postacie, oswietlenie, unity6, pomiar]
date: 2026-08-07
status: draft
---

# Promien SSAO w URP jest w METRACH i domyslnie mikroskopijny - przy twarzach decyduje o wszystkim

## Objaw

Twarze postaci wygladaja "niedomalowane": ciemne przyciemnienie pod oczami, w oczodolach,
przy skrzydelkach nosa. Podejrzenie pada na teksture z generatora AI albo na cienie w grze.

## Co sie okazalo

`ScreenSpaceAmbientOcclusion.Radius` w URP ma **domyslna wartosc 0.035** i jest wyrazony
w **metrach swiata**. W projekcie stalo 0.3, czyli 8,6 raza wiecej. Do tego URP mnozy promien
przez **1.5 dla metody BlueNoise** (`ScreenSpaceAmbientOcclusionPass.cs`), wiec efektywny
promien wynosil **0.45 m** przy glowie ludzkiej szerokiej na ~0.2 m.

Skutek: przyciemnienie nie lapie zagłębien twarzy (oczodol ma glebokosc 2-4 cm), tylko
obejmuje **cala glowe jednym gradientem**. Wyglada jak brud na skorze, nie jak cieniowanie.

Pomiar na modelu z gry (rozbicie w stopach jasnosci, kontrast na twarzy):

| warstwa | wklad |
|---|---|
| tekstura (wypalone w PNG) | 0.307 |
| ksztalt bryly + swiatlo | 1.317 |
| mapa wypuklosci | 0.003 |
| cien slonca | 0.033 |
| **przyciemnianie zakamarkow (promien 0.3)** | **0.134** |
| to samo z promieniem domyslnym 0.035 | 0.010 |

Czyli **~93% wkladu tej warstwy bralo sie z samego zawyzonego promienia**.

Dodatkowo `DirectLightingStrength` (domyslnie 0.25) sprawia, ze przyciemnianie tlumi takze
**swiatlo bezposrednie**, nie tylko rozproszone - wiec dziala nawet w pelnym sloncu.

## Reguly do zapamietania

1. Promien SSAO dobieraj do **skali detalu**, ktory ma byc przyciemniony, nie "na oko".
   Dla twarzy to centymetry, nie decymetry.
2. Sprawdz, czy pipeline nie mnozy promienia (URP: x1.5 dla BlueNoise).
3. Zanim uznasz teksture z generatora AI za winna ciemnych plam - **wyrenderuj model
   materialem bez oswietlenia** (URP/Unlit z ta sama mapa koloru). Jesli plama znika,
   tekstura jest niewinna. To jeden render i zamyka dyskusje.
4. "Nie odbieraj cieni" (`receiveShadows = false`) **nie dotyczy SSAO** - to nie jest cien
   w rozumieniu Unity. Zmierzone: wplyw dokladnie 0.000 stopa.

## Anty-wniosek

Nie zakladaj, ze objaw wystepujacy tylko na nowych modelach musi miec przyczyne w nowych
modelach. Tu zawyzony promien SSAO uderzal **mocniej w stare modele** (1.398 stopa) niz
w nowe (0.134) - roznica brala sie z czego innego. Mierz, zanim przypiszesz wine.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260807-2230-splaszczanie-oswietlenia-per-material|20260807-2230-splaszczanie-oswietlenia-per-material]] - wspolne: oswietlenie, ssao, postacie
- [[20260807-2330-fallback-shadera-wysypuje-build|20260807-2330-fallback-shadera-wysypuje-build]] - wspolne: unity6, urp
<!-- /POWIAZANE:auto -->
