---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [unity, ugui, contentsizefitter, 9-slice, layout, hud, code-built-ui]
date: 2026-06-16
status: draft
severity: medium
---

# 9-slice Image na obiekcie layoutu zaniża ContentSizeFitter (panel rośnie do sumy borderów)

## Objaw
Kod-budowane panele UI (drewniane plakietki HUD) miały rosnąć do treści przez
`ContentSizeFitter` (PreferredSize) + `HorizontalLayoutGroup`/`VerticalLayoutGroup`.
Zamiast tego wszystkie wychodziły o stałym rozmiarze **192×192 px** (a powinny ~108×77),
niezależnie od krótkiej treści. Szerokość tam, gdzie tekst był długi, była OK — ale rozmiar
nigdy nie schodził poniżej 192 na żadnej osi.

## Root cause
Tło 9-slice (`Image`, typ `Sliced`) siedziało na TYM SAMYM GameObjekcie co `LayoutGroup`
i `ContentSizeFitter`. Slicowany `Image` implementuje `ILayoutElement` i zgłasza
**minimalny/preferowany rozmiar = suma borderów 9-slice** (tu border 96 px → 96+96 = 192).
`ContentSizeFitter` bierze `max(preferowane_layoutu, preferowane_Image)` → podłoga 192 na obu osiach.
Kluczowe: ten zgłaszany rozmiar **ignoruje `pixelsPerUnitMultiplier`** — mimo że renderowana
ramka miała ~16 px (multiplier 6), zgłoszone „preferred" to surowe 192.

## Fix
Przenieść tło 9-slice do **osobnego dziecka**:
- dziecko `Background` z `Image` (sliced), anchory stretch (0,0)–(1,1), offsety 0 → wypełnia rodzica,
- `LayoutElement.ignoreLayout = true` → LayoutGroup je pomija, a jego preferred nie wpływa na rodzica,
- ustawić jako pierwszy sibling (renderuje się za treścią).
Obiekt-rodzic (root layoutu) ma wtedy TYLKO LayoutGroup + ContentSizeFitter → wymiaruje się
czysto do treści. Tło i tak skaluje się z rodzicem przez anchory.

## Reguła ogólna
Każdy kod-budowany panel uGUI, który ma `ContentSizeFitter` (grow-to-content) ORAZ tło 9-slice,
musi mieć tło jako osobne dziecko z `ignoreLayout=true`. Nigdy nie kładź slicowanego Image na tym
samym obiekcie co layout root.

## Tip diagnostyczny
Zamiast polegać na zrzucie ekranu — w Play Mode odpal mały skrypt logujący
`rectTransform.rect.size` paneli i ich dzieci. Podejrzanie równa liczba (= suma borderów sprite'a)
wskazuje od razu na ten problem.
