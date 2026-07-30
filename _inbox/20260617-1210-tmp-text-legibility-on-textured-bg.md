---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [unity, ui, textmeshpro, tmp, legibility, outline, material, canvas, modal]
date: 2026-06-17
status: draft
---

# TextMeshPro: czytelność na teksturowanym tle (drewno) + warstwy modali

## Problem (severity: medium)
Tekst TMP na drewnianym/teksturowanym tle UI „znikał" — był blady, nieczytelny. Reżyser
zgłosił to jako priorytet nr 1.

## Root cause (transferowalny, engine-level)
1. **Komponent `UnityEngine.UI.Outline` / `Shadow` NIE działa na `TextMeshProUGUI`** — działa
   tylko na legacy `UnityEngine.UI.Text`. TMP generuje własny mesh; efekty BaseMeshEffect go
   nie obejmują. (W projekcie HUD używał Outline na legacy Text — i to się myliło z TMP.)
2. **TMP outline ustawia się na MATERIALE**, nie na komponencie. Ustawienie `text.outlineWidth`/
   `outlineColor` rusza WSPÓŁDZIELONY materiał SDF → zmienia WSZYSTKIE teksty tej czcionki.
3. Często jest tylko jeden ciężar fontu (Regular SDF), brak Bold jako osobny asset.

## Rozwiązanie (wzorzec)
Per-instance materiał + faux-bold, jednym helperem (reużywalnym):

```csharp
public static void MakeReadable(TextMeshProUGUI t, Color outlineColor,
                                float outlineWidth = 0.2f, float faceDilate = 0.12f)
{
    var m = t.fontMaterial;            // DOSTĘP do .fontMaterial auto-klonuje materiał per-instancja
    m.SetFloat(ShaderUtilities.ID_OutlineWidth, outlineWidth);
    m.SetColor(ShaderUtilities.ID_OutlineColor, outlineColor);
    m.SetFloat(ShaderUtilities.ID_FaceDilate, faceDilate);  // pogrubienie bez osobnego Bold SDF
    t.UpdateMeshPadding();            // bez tego kontur/dilate się przycina
}
```
- `.fontMaterial` (NIE `.fontSharedMaterial`) → instancja, nie psuje innych tekstów.
- `_FaceDilate` daje „faux-bold" gdy nie ma osobnego ciężaru fontu.
- Tekst KREMOWY + ciemny kontur czyta się na drewnie DOWOLNEJ jasności (recepta jak HUD).
- Koszt: 1 materiał per tekst — OK dla modala (kilkadziesiąt), przy listach rebuildowanych
  materiały giną z GameObjectem.

## Powiązany gotcha — warstwy modali (Canvas sortingOrder)
Modal budowany jako dziecko współdzielonego `MainCanvas` (sortingOrder 0) NIE zakrywa innych
paneli HUD na tym samym canvasie — scrim ich nie przyciemnia, „przebijają". Fix: modal tworzy
WŁASNY `Canvas` (ScreenSpaceOverlay, `sortingOrder = 100`) + CanvasScaler + GraphicRaycaster.
Wtedy scrim zakrywa cały HUD. (Wzorzec już w projekcie: RackTransferUI, ChoppingSelectionUI.)

## When to apply
Każdy runtime-budowany TMP UI na jasnym/teksturowanym tle; każdy modal, który ma przyciemniać
i zakrywać HUD.
