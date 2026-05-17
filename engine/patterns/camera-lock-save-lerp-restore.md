---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, camera, lerp, minigame, restore, cinematic]
date: 2026-05-17
status: draft
---

# Camera Lock: Save → Lerp → Restore

## When to use
During minigames or cutscenes that move the player camera to a fixed vantage point. Provides smooth entry/exit without code duplication per minigame.

## Steps
Save at minigame start (world coordinates, NOT local — local breaks if parent moves):
```csharp
_savedCamPos = playerCamera.transform.position;
_savedCamRot = playerCamera.transform.rotation;
_savedCamParent = playerCamera.transform.parent;
```

Lerp to target over ~0.3s:
```csharp
while (t < 1f) {
    t += Time.deltaTime / 0.3f;
    playerCamera.transform.position = Vector3.Lerp(_savedCamPos, targetPos, t);
    playerCamera.transform.rotation = Quaternion.Slerp(_savedCamRot, targetRot, t);
    yield return null;
}
```

During lock: `playerController.canMove = false` (see [[universal-camera-lock-canmove-flag]])

Restore on exit:
1. Reverse lerp back to saved world position/rotation
2. `camera.transform.SetParent(_savedCamParent)`

## Why this works
Saving world coordinates is robust — parent transform changes during the minigame don't corrupt the saved state. Lerp gives smooth feel. World coord restore is bulletproof.

## Trade-offs
Manual implementation (no Cinemachine). For projects with Cinemachine available, use virtual cameras instead.

## Variants
Same pattern for: dialogue close-ups (lerp to NPC face), tutorial highlights (lerp to relevant machine), inspection mode (rotate around object).
