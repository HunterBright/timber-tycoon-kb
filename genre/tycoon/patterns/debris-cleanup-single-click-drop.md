---
name: debris-cleanup-single-click-drop
description: Tutorial debris interaction — single E click, pile despawns, 1 plank drops. No minigame, no hold, no animation. First gameplay moment must be instant and satisfying.
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: genre/tycoon/patterns
  tags: [unity, tutorial, debris, cleanup, visual-progression]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Debris Cleanup — Single-Click Drop Materials Visual

## When to use
Tycoon games with a tutorial cleanup task that introduces interaction. The first gameplay moment must feel immediate and rewarding — not frustrating. Use when you need a visible "I did something" payoff with zero friction.

## Steps

**DebrisCollectible script:**
```csharp
public class DebrisCollectible : MonoBehaviour, IInteractable {
    [SerializeField] GameObject plankPrefab;
    [SerializeField] QuestEventSO onDebrisCollected;

    public void Interact(PlayerController player) {
        Instantiate(plankPrefab, transform.position + Vector3.up * 0.3f, Quaternion.identity);
        onDebrisCollected?.Raise(gameObject);
        Destroy(gameObject);
    }
}
```

**Scene setup:**
- 6 DebrisCollectible GameObjects in starting sawmill area
- Each: BoxCollider (trigger) + Interactable + DebrisCollectible
- Plank prefab drops at pile position (visible on ground)

**Quest integration:**
- `OnDebrisCollected` event increments quest counter (0/6 → 6/6)
- Quest highlight (#047) draws player attention to nearest uncollected pile
- On 6/6: quest complete → unlock next step (repair Counter_Broken with planks)

**Why no minigame:** Tutorial moment. Player is learning the E key and basic interaction. Multi-step minigame here = confusion. Single click = immediate agency.

## Why this works
First interaction teaches the E-to-interact convention. Material reward (plank appearing on ground) immediately makes the player feel productive. Quest tracker reinforces that each pile matters. Zero chance of failing = confidence building before harder mechanics.

## Trade-offs
- No challenge: entirely skill-free. Intentional — tutorial must not gate players on execution.
- Plank physics: if plank drops with physics and bounces away, player may not find it. Either: use kinematic spawn + place directly in inventory, or spawn nearby with short physics burst + sleep.
- 6 piles: too few = anticlimactic, too many = tedious before game starts. 6 is calibrated for ~90 seconds of early engagement.

## Variants
- **Hold interact:** replace E-click with hold-E 1s for slightly more weight. Not recommended for first interaction.
- **Sweep animation:** pile animates "swept to side" before despawn. Visual polish, not functionally different.
- **Random loot:** pile drops random material (1-2 planks, sometimes bark). Adds surprise but complicates quest tracking.

See also: [[quest-highlight-pattern]], [[wing-snap-points-modular-fade-in]], [[multi-step-quest-checklist]]
