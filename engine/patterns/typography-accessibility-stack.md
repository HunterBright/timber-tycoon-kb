---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, ui, typography, accessibility, textmeshpro, font, sdf]
date: 2026-05-17
status: draft
---

# Typography + Accessibility Stack

Merger of #093 (TextMeshPro rule), #094 (style guide), #095 (dynamic sizing).

## When to use
Any Unity UI with text. TextMeshPro is the Unity standard — legacy `Text` is EOL. Apply this stack at project start; retrofitting 50 screens is painful.

---

## Part 1 — TextMeshPro Rule (#093)

**NEVER use Unity legacy `Text` component — TextMeshPro mandatory on all UI.**

**Font requirements for internationalization:**
- Must include full extended Latin set for Polish characters (ą ę ó ś ź ż ć ń ł)
- Recommended fonts (OFL license): Noto Sans, Inter, Roboto
- TT uses: **Nunito** from Google Fonts (OFL)

**Generate SDF atlas:**
1. Window → TextMeshPro → Font Asset Creator
2. Assign `.ttf` or `.otf` file
3. Atlas Resolution: **2048×2048** (for all Latin + Polish + symbols)
4. Sampling Point Size: **90**
5. Generate Font Atlas → save as `.asset`
6. Assign in `TMP_Settings` as default font

**Fallback font chain:** assign in TMP Settings for CJK or other extended character sets if future localization is planned.

---

## Part 2 — Style Guide (#094)

**Font size hierarchy** (base scale, before accessibility multiplier):

| Role | Size |
|------|------|
| Header | 32 pt |
| Subheader | 24 pt |
| Body | 18 pt |
| Caption | 14 pt |
| Tooltip | 16 pt |

Define sizes in `UIStyleGuideSO` ScriptableObject — not hardcoded in prefabs.

**Color palette:**
```
Primary text:    #FFFFFF  (white)
Secondary text:  #B5B5B5  (light gray)
Accent:          brand color
Positive:        #5BAA47  (green)
Negative:        #C75F50  (red)
Disabled:        #666666  (dark gray)
```

Apply via `TMP_StyleSheet` or reference from `UIStyleGuideSO`.

---

## Part 3 — Accessibility Dynamic Sizing (#095)

**AccessibilityManager** provides `FontSizeMultiplier` (0.8 / 1.0 / 1.3):

```csharp
public class AccessibilityManager : MonoBehaviour, ISaveable {
    public float FontSizeMultiplier { get; private set; } = 1.0f;
    List<(TMP_Text text, float baseSize)> scalableTexts = new();

    public void RegisterScalableText(TMP_Text t) =>
        scalableTexts.Add((t, t.fontSize));

    public void SetFontSizeMultiplier(float m) {
        FontSizeMultiplier = m;
        RefreshAllFontSizes();
    }

    void RefreshAllFontSizes() {
        foreach (var (t, baseSize) in scalableTexts)
            t.fontSize = baseSize * FontSizeMultiplier;
    }
}
```

**All TMP_Text components register in Awake:** `AccessibilityManager.Instance.RegisterScalableText(this.GetComponent<TMP_Text>())`.

**SDF rendering ensures crispness at all sizes** — no rasterization artifacts at 0.8× or 1.3×. SDF material can include outline and shadow without quality loss.

**ISaveable:** persist multiplier setting across sessions.

---

## Why this works
SDF font rendering = crisp at any size. Style guide enforced via SO = designers use consistent sizes without memorizing values. Accessibility multiplier applied globally = one setting change updates all 50+ screens.

## Trade-offs
- SDF atlas generation: 2048×2048 takes ~30s per font — one-time cost
- `FindObjectsOfType<TMP_Text>` scan on startup is expensive in complex scenes — use registration pattern (as above) instead
- Multiplier 1.3× may cause text overflow in fixed-size UI containers — test all screens at max size during development

## Variants
- **Simpler version**: single font size scale, no registration pattern — just `TMP_Settings` override
- **High contrast mode**: separate `IHighContrastable` interface, inverts palette for accessibility
