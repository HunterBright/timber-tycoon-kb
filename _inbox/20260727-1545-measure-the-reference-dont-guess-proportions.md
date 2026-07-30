---
title: Measure the reference from pixels, then gate against the measurement
type: pattern
status: draft
confidence: low
verified: ''
date: '2026-07-27'
project: Kerf - Sawmill Tycoon
tags:
- reference
- proportions
- measurement
- art-direction
- low-poly
- silhouette
applies_to:
- 3d-art
- procedural-characters
- any-engine
source: ''
severity: medium
suggested-category: workflow/patterns
time_lost: ''
---

# Measure the reference from pixels, then gate against the measurement

## Context
A stylised character kept being described as "reads like a sack" through
several rebuild rounds. Every round changed something plausible and the
complaint survived. The rounds were driven by opinion, so they could not
converge.

## The pattern
1. **Get the reference as images.** Orthographic-ish screenshots are enough.
   Legally this matters too: measuring proportions from a preview is learning,
   copying vertex positions is not.
2. **Extract the silhouette programmatically.** Threshold against the
   background colour, then apply a binary closing wider than the wireframe
   lines. Skipping the closing splits the silhouette wherever a dark edge line
   crosses it and produces confident nonsense - one shoulder measured as three
   separate 5 cm blocks instead of one 25 cm span.
3. **Normalise by stature**, never by pixels, so the numbers survive any image
   size and any character height.
4. **Store the table as code**, with provenance comments, not as a doc. The
   gates then compare against a number that provably did not come from the
   generator, so the generator cannot agree with itself.
5. **Gate on shape signatures, not absolute sizes.** Absolute widths are what
   the user's sliders are for. Ratios are what anatomy is for:
   - width at each level divided by chest width
   - **depth divided by width at each level**

## The payoff
Two numbers explained several months of vague complaints at once:

    shoulder / chest   reference 1.50   ours 1.06
    depth / width      reference varies 0.66 -> 0.88 -> 0.74 by height
                       ours constant 0.69 - 0.72 at every height

A torso whose cross-section keeps the same proportions all the way up *is*
"one section stretched along a body" - which is the literal definition of the
thing everyone had been calling a sack, and it is trivially measurable.
The fix followed directly: narrow the ribcage, let the deltoid carry the
shoulder width, and vary the depth/width ratio by height.

The same method found a hand 2.4x thicker than a human hand (68 mm against
28 mm), a defect that seven existing hand gates all missed because every one of
them measured in the width-length plane and none measured thickness.

## Fitting the constants
When several constants each affect several measured heights (because the
profile is smoothed between control nodes), hand-tuning does not converge.
A brute-force sweep over a coarse grid, scored by the summed deviation from the
reference table across all heights at once, lands it in seconds. Then verify
the extremes still pass a wide sanity band - fitting the default figure must
not make the sliders illegal.

## Watch out
- Only enforce a tight match on the **default** setting. Sliders exist so the
  user can leave the reference; a gate that punishes using them punishes using
  the tool. Give every other setting a wide sanity band instead.
- Check what a borrowed reference number actually measures. A previous round
  doubled a hand's thickness chasing "hand thickness 0.079" copied from an
  in-game asset; that number was almost certainly the hand *including the
  thumb*, not the palm plate.

## Related
- [[20260727-1535-gates-must-not-identify-parts-by-world-coordinate]]
