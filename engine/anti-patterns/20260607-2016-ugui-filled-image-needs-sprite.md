---
title: A UGUI `Image` with `type = Filled` but no sprite ignores `fillAmount`
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-06-07'
project: Kerf - Sawmill Tycoon
tags:
- unity
- ugui
- ui
- image
- fillamount
- progress-bar
- urp
applies_to: []
source: ''
promoted: '2026-07-30'
---

# A UGUI `Image` with `type = Filled` but no sprite ignores `fillAmount`

## What we tried
Built a runtime progress bar (reputation HUD) entirely in code:

```csharp
fillBar = fillObj.AddComponent<Image>();
fillBar.color = new Color(0.773f, 0.910f, 0.322f, 1f);
fillBar.type = Image.Type.Filled;
fillBar.fillMethod = Image.FillMethod.Horizontal;
fillBar.fillAmount = 0f;
// NOTE: fillBar.sprite was never assigned
```

The data/logic side was correct: the widget subscribed to the change event, the
fill value was recomputed correctly on every update (`Mathf.Clamp01((current - floor) / gap)`),
yet the bar **never moved visually** - it rendered as a constant full-width quad.

## Why it doesn't work
Unity's `Image.OnPopulateMesh` short-circuits when there is no sprite:

```csharp
protected override void OnPopulateMesh(VertexHelper toFill)
{
    if (activeSprite == null)
    {
        base.OnPopulateMesh(toFill); // plain full-rect Graphic quad
        return;                      // type & fillAmount are NEVER consulted
    }
    switch (type) { ... case Type.Filled: GenerateFilledSprite(...); ... }
}
```

With `activeSprite == null`, the component falls back to the base `Graphic` mesh - a
solid rectangle filling the whole RectTransform, using only `color`. `type`,
`fillMethod`, and `fillAmount` are completely ignored. So a "Filled" image with no
Source Image always looks 100% full and never animates.

In the Editor this is easy to miss: dragging a UGUI Image in usually auto-assigns the
default `UISprite`, so designers rarely hit it. **Code-created** Images start with
`sprite == null`, which is where the trap lives.

## Fix
Assign any sprite before relying on `fillAmount`:

```csharp
fillBar.sprite = Resources.GetBuiltinResource<Sprite>("UI/Skin/UISprite.psd");
// or a 1x1 white sprite:
// fillBar.sprite = Sprite.Create(Texture2D.whiteTexture, new Rect(0,0,1,1), new Vector2(0.5f,0.5f));
fillBar.type = Image.Type.Filled;
```

## Rule of thumb
Any code-built UGUI `Image` that uses `Filled` / `Sliced` / `Tiled`, or that you intend
to animate via `fillAmount`, **must** have a sprite assigned. A bare colored Image only
works for the `Simple` look (a solid quad). If a "filled" bar appears stuck at full,
check the Source Image field first - before suspecting the binding or the math.
