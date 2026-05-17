---
name: wing-snap-points-modular-fade-in
description: Building expansion via pre-placed wing prefabs (all in scene at start, hidden), revealed with 1.5s material fade-in + dust VFX on purchase
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: genre/tycoon/patterns
  tags: [unity, building, modular, snap-points, fade, expansion]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Wing Snap-Points Modular Instant Fade-In

## When to use
Tycoon games with expandable buildings (sawmill wings, store extensions, farm outbuildings). Player should feel immediate reward on purchase — not "wait for construction animation."

## Steps

**All wings placed in scene at design time:**
```
Scene hierarchy:
  Sawmill/
    Strefa_1 (active=true)   ← always visible
    Strefa_2 (active=false)  ← hidden at start
    Strefa_3 (active=false)  ← hidden at start
```

**Purchase → fade reveal:**
```csharp
public class WingReveal : MonoBehaviour {
    [SerializeField] string wingId;
    [SerializeField] float fadeDuration = 1.5f;
    [SerializeField] GameObject dustVFX;

    void OnEnable() => onUpgradePurchased.Register(OnPurchased);

    void OnPurchased(string id) {
        if (id != wingId) return;
        gameObject.SetActive(true);
        StartCoroutine(FadeIn());
        if (dustVFX) Instantiate(dustVFX, transform.position, Quaternion.identity);
    }

    IEnumerator FadeIn() {
        float elapsed = 0f;
        var renderers = GetComponentsInChildren<Renderer>();
        while (elapsed < fadeDuration) {
            elapsed += Time.deltaTime;
            float alpha = elapsed / fadeDuration;
            foreach (var r in renderers) {
                var mat = r.material;
                mat.color = new Color(mat.color.r, mat.color.g, mat.color.b, alpha);
            }
            yield return null;
        }
        // Restore all materials to full opacity
        foreach (var r in renderers) { var c = r.material.color; r.material.color = new Color(c.r, c.g, c.b, 1f); }
    }
}
```

**Audio:** construction sound + ambient swell during fade. Sells "something just happened" feeling.

**Snap-point workflow:**
1. Designer builds all wing prefabs in Blender
2. All wings placed in scene by level designer at correct positions
3. Strefa_2, Strefa_3 set inactive in Inspector
4. `wingId` field set to matching upgrade ID
5. When upgrade purchased → fade-in reveals. Zero runtime positioning.

**Inspiration:** Tavern Manager (instant building reveal), Schedule 1 (immediate property upgrades).

## Why this works
Pre-placing wings eliminates runtime "where does the wing go?" logic. Fade-in is cosmetic — wing is already at correct position and scale. Purchase = instant gameplay benefit (new workstations accessible), fade = visual polish.

## Trade-offs
- All wings always in scene: memory + rendering cost even for hidden wings. At 3-5 wings with modest poly count, negligible. For 20+ wings, consider async loading.
- Material transparency requirement: shader must support alpha. URP/Lit supports this with alpha transparency mode. Opaque materials can't fade — ensure wing materials use Transparent or Alpha rendering mode.
- Dust VFX position: use wing root transform for VFX spawn. For large wings, consider multiple dust burst points along the edge.

## Variants
- **Dissolve shader:** instead of alpha fade, use a dissolve shader (noise-based disappear/appear). More visually interesting, requires custom shader.
- **Scaffold placeholder:** before purchase, wing position shows scaffolding (thin outline / transparent mesh). Player sees "future expansion" area. Adds anticipation.

See also: [[building-progression-instant-spawn]], [[vfx-trigger-pattern]]
