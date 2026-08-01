---
title: Blender flat panel textured on one face renders BLANK in Unity (axis-flip picks the wrong face)
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-04'
project: Kerf - Sawmill Tycoon
tags:
- blender
- unity
- fbx
- uv
- texture
- forward-axis
- flat-panel
- low-poly
applies_to: []
source: ''
severity: medium
promoted: '2026-07-30'
---

# Blender flat panel textured on one face renders BLANK in Unity (axis-flip picks the wrong face)

## Objaw
Płaski panel z Blendera (tablica, znak, ekran) z teksturą/rysunkiem wypalonym na jednej dużej ścianie
importuje się do Unity i widoczna ściana jest PUSTA (goły materiał bazowy), mimo że atlas/UV/bake są poprawne.

## Przyczyna
Cienki panel = pudełko z DWIEMA dużymi ścianami (przód i tył). Bake trafia na jedną z nich. Przy eksporcie
FBX Blender→Unity następuje flip osi forward (ten sam rodzaj quirku co „forward auta = -transform.right"
z FBX Blendera) - przez co to, KTÓRĄ z dwóch dużych ścian Unity pokazuje graczowi, jest niejednoznaczne.
W efekcie gracz patrzy na PUSTĄ (tylną) ścianę, a rysunek jest po drugiej stronie.

Diagnostyczne: jeśli ściana jest w 100% jednolita (żadnych akcentów kolorystycznych) - to nie „za słaby
kontrast" ani „UV frontu w złym regionie atlasu", tylko pokazywana jest przeciwna ściana.

## Rozwiązanie (pewne)
Pomaluj/UV-uj OBIE duże ściany panelu tym samym rysunkiem; dla tylnej odbij UV w poziomie (u → 1-u), żeby
też czytała się od lewej do prawej z własnej strony. Wtedy niezależnie od tego, którą ścianę Unity pokaże,
jest czytelna (ukryta ściana przy ścianie i tak nie jest widoczna). Zero dodatkowej geometrii.

Alternatywy: (a) zweryfikować w Unity którą ścianę widać i obrócić model o 180°, ale to kruche przy re-eksporcie;
(b) dla mesha bez tyłu - upewnić się że normalna dużej ściany + kierunek eksportu dają front do gracza.

## Weryfikacja przed „gotowe"
Round-trip realnego FBX i próbkowanie wypalonego PNG w UV0 OBU dużych ścian (jak próbkuje Unity) - obie muszą
zwracać wysoką wariancję koloru (PAINTED), nie płaski kolor. Sam render w Blenderze nie wystarczy (Blender może
pokazywać właściwą stronę, Unity przeciwną).

## Koszt gdy pominięte
Wygląda jak błąd materiału/UV → można stracić czas na reimport tekstury i reassign materiału (bezskutecznie),
zanim się zrozumie, że to wybór ściany przez flip osi.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[forward-axis-blender-fbx-quirk|Forward Axis = -transform.right (Blender FBX Quirk)]] - wspolne: forward-axis, fbx, blender
- [[20260702-1400-bark-atlas-vs-tile-strategy|Rodzina assetów współdzieląca teksturę: wybierz strategię (atlas vs kafel) PRZED budową]] - wspolne: uv, low-poly, blender
- [[20260704-1322-blender-file-image-scale-bake-revert|Blender: image.scale() na file-backed image nie trzyma się podczas bake - użyj images.new()]] - wspolne: texture, blender
- [[20260725-1930-blender-pixels-buffer-not-converted-to-srgb-on-png-save|Blender nie przelicza `image.pixels[]` na sRGB przy zapisie PNG]] - wspolne: texture, blender
- [[20260606-0930-baked-atlas-texture-foreign-uvs|Don't apply a baked-atlas texture to a mesh whose UVs were authored for a different atlas]] - wspolne: uv, blender
- [[cycles-bake-for-solid-colors|ANTI-PATTERN: Cycles Bake for Solid Color Regions]] - wspolne: uv, blender
<!-- /POWIAZANE:auto -->
