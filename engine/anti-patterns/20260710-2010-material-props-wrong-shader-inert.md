---
title: Ustawianie właściwości materiału bez sprawdzenia SHADERA = ciche nic + ryzyko nadpisania
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-07-10'
project: Kerf - Sawmill Tycoon
tags:
- unity
- materials
- shader
- urp
- editor-script
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Ustawianie właściwości materiału bez sprawdzenia SHADERA = ciche nic + ryzyko nadpisania

## Co poszło nie tak
Zadanie: "dodaj maszynom delikatny połysk". Skrypt edytorowy ustawił `_Smoothness`,
`_Metallic`, `_SpecularHighlights` + keywordy URP/Lit na materiałach maszyn.
Efekt: ZERO połysku - materiały używały projektowego `Custom/FlatColorLit`
(czyta tylko `_BaseColor` i `_Brightness`); wszystkie zapisy były martwe
(`SetFloat` na niezadeklarowanej właściwości nie rzuca błędem, keyword ląduje
w `m_InvalidKeywords`). GORZEJ: ten sam skrypt pisał też `_BaseColor` ze starej
tablicy wartości - jedyny zapis, który ZADZIAŁAŁ, i nadpisał zaakceptowaną paletę.

## Zasady
1. PRZED skryptową edycją materiału sprawdź `mat.shader.name` (i zweryfikuj GUID
   shadera w YAML .mat - GUID może należeć do customowego shadera, nie do URP/Lit).
2. W skryptach masowej edycji materiałów dodaj guard: `if (mat.shader.name != oczekiwany) skip`.
3. Skrypt "zmień cechę X" NIE powinien przepisywać innych właściwości (kolorów!)
   ze swojej tablicy "źródła prawdy" - stare tablice wartości potrafią przeżyć
   rework wyglądu i cicho cofnąć akceptowaną paletę.
4. Jednorazowe skrypty look-ów po reworku wyglądu blokuj guardem OBSOLETE -
   przypadkowy re-run z menu nie może nadpisać aktualnego akceptu.
5. Sygnał ostrzegawczy w .mat: właściwość/keyword w `m_InvalidKeywords` lub wartości,
   które "nic nie zmieniają" = piszesz do niewłaściwego shadera.

## Jak wykryto
Adwersaryjny review diffu (multi-agent) porównał GUID shadera z plikiem .shader
i wartości LFS blobów przed/po - ręczny playtest mógł tego nie zauważyć od razu.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260702-2140-shader-property-stale-serialized-material-values|Dodanie property do shadera może aktywować STARE, ukryte wartości w materiałach]] - wspolne: shader, materials, urp
- [[20260710-1300-vertex-colors-vs-basecolor-linear|Vertex colors vs _BaseColor w Linear color space - ten sam hex renderuje się INACZEJ]] - wspolne: shader, materials, urp
- [[low-poly-water-side-wave|ANTI-PATTERN: sin(X) + sin(Z) Water Shader = Side Waves]] - wspolne: shader, urp
- [[20260606-0930-baked-atlas-texture-foreign-uvs|Don't apply a baked-atlas texture to a mesh whose UVs were authored for a different atlas]] - wspolne: materials, urp
- [[20260807-2330-fallback-shadera-wysypuje-build|20260807-2330-fallback-shadera-wysypuje-build]] - wspolne: shader, urp
- [[20260705-0840-blender-linear-color-into-unity-srgb-material|Blender LINIOWE Base Color wpisane wprost w Unity Color property = ~1 gamma za ciemno (projekt Linear)]] - wspolne: shader, materials
<!-- /POWIAZANE:auto -->
