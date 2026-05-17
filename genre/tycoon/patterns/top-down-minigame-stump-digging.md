---
type: pattern
project: timber-tycoon
suggested-category: genre/tycoon/patterns
tags: [unity, minigame, camera, top-down, stump, mechanic]
date: 2026-05-17
status: draft
---

# Top-Down Camera Minigame (Stump Digging)

## When to use
Tycoon games with a physical interaction mechanic that benefits from a top-down perspective. Stump digging is a spatial task (click multiple points around an object) — top-down makes the target points readable without FPP camera distortion.

## Steps

**Camera transition:**
```csharp
// Position directly above stump, looking down
Vector3 topDownPos = stump.transform.position + Vector3.up * 10f;
Quaternion topDownRot = Quaternion.Euler(90f, 0f, 0f);
StartCoroutine(LerpCameraTo(topDownPos, topDownRot, 0.6f));
```

**Target point generation (4-8 points around perimeter):**
```csharp
List<Vector3> GenerateDigPoints(Transform stump, int count) {
    var points = new List<Vector3>();
    float radius = 0.6f; // stump perimeter radius
    for (int i = 0; i < count; i++) {
        float angle = (i / (float)count) * 360f * Mathf.Deg2Rad;
        Vector3 offset = new Vector3(Mathf.Cos(angle), 0f, Mathf.Sin(angle)) * radius;
        points.Add(stump.position + offset);
    }
    return points;
}
```

**Click interaction:**
- Raycast from screen position to world plane (y = stump.position.y)
- If hit within `clickRadius` of next target point: register "dig"
- Play dig animation (brief down-up motion on point indicator) + spark VFX
- Progress to next point in sequence (or nearest unclicked — player choice)

**Completion:**
```csharp
void OnAllPointsClicked() {
    // 1. Tilt stump (Rigidbody.isKinematic = false + AddForce)
    // 2. Brief fall animation (~0.5s)
    // 3. CollectableStump spawns at stump landing position
    // 4. Lerp camera back to player FPP
}
```

**Minigame catalog in Timber Tycoon (4 distinct mechanics):**
1. Zone-click (tree chop) — strafe + click at correct moment
2. Circular cursor needle (sawmill) — needle in spinning wheel
3. Hold-needle (chipper) — hold E until needle in zone
4. Top-down click-targets (stump digging)

## Why this works
Top-down gives full spatial awareness of the stump. Click-targets are large enough to be forgiving (radius-based hit detection, not pixel-perfect). The stump lifting at the end is the payoff — physical reward visible from the top-down perspective before camera returns to FPP.

## Trade-offs
- Camera transition cost: lerp to top-down and back = ~1.2s total. Acceptable for 5-6 stumps per session. For rapid repeated digging, reduce transition duration to 0.3s.
- Sequence vs. free-click: sequential targeting (must click in order) is harder but more minigame-like; free-click (any order) is easier and more casual. Choose based on target difficulty tier.
- 4 vs 8 points: 4 = easy (casual players), 8 = moderate challenge. Expose as `DiggableStump.clickCount` per stump, designer-set.

## Variants
- **Drag-to-dig:** instead of clicks, player drags from point to point (like connecting dots). More gesture-based, works on mobile.
- **Timed version:** points appear one at a time with a 2s window per point. Miss = point disappears and must wait for it to return. Adds timing pressure.

See also: [[quantity-not-quality-principle]], [[sliding-head-bandsaw-mouse-drag-tempo-minigame]]
