---
type: anti-pattern
project: Kerf - Sawmill Tycoon
suggested-category: engine/anti-patterns
tags: [testing, procedural-mesh, gates, false-green, geometry]
severity: high
time_lost: "~40 min, twice"
date: 2026-07-27
status: draft
applies_to: [procedural-geometry, automated-testing, any-engine]
---

# A geometry gate that identifies body parts by raw world coordinate is a gate on credit

## The anti-pattern
Automated checks over procedural geometry often need to say "measure the
shoulders" or "measure the hand". The tempting shortcut is to identify the part
by a raw coordinate:

    # "a torso ring is a group of exactly N vertices at the same height"
    grupy = {}
    for (x, y, z) in verts:
        grupy.setdefault(round(z, 7), []).append(x)
    ring = [xs for xs in grupy.values() if len(xs) == SEGMENTS]

    # "the hand is everything past the wrist along X"
    hand = [f for f in faces if all(verts[i][0] > wrist_x for i in f)]

Both worked for months. Both are time bombs.

## Why it fails
The identifying coordinate is not a property of the anatomy, it is a property of
the **current construction**. Change the construction in a way that is otherwise
correct and the gate stops finding anything - and a gate that finds nothing does
not fail loudly, it reports a benign-looking zero.

Two concrete failures on the same project, one week apart in the same file:

1. Torso rings were made to **undulate** (each point of a ring gets its own
   height, so edge loops follow the pectoral and the lat instead of being flat
   hoops). Correct change, big visual win. The "20 vertices at one height" gate
   instantly matched nothing and reported *shoulder width 0.000 m* instead of
   raising an alarm about the geometry.
2. A second gate identified the hand as "everything with x greater than the
   wrist x". The moment the arms rotate into an A-pose, the hand is no longer
   the far end of the X axis and eight hand gates go blind - precisely when the
   hand has just been rebuilt and needs watching most.

Both are false greens in disguise: the check appears to run, produces a number,
and the number is meaningless.

## What to do instead
- **Measure the surface, not the intent.** Intersect the mesh with a plane and
  work on the resulting contours. A cross-section is defined by geometry, not by
  how the geometry happened to be generated, so it survives topology changes.
- **Measure in the part's own frame.** If a limb has a local coordinate system
  in the generator, expose it and transform vertices into it before measuring.
  Then the same gate works at any pose.
- **Detect boundaries by discontinuity, not by a fixed threshold.** "Walk up
  until the width jumps by more than 8% in one step" beats "measure at 25% of
  the way up", because the fixed height silently drifts into the wrong body part
  when a slider moves.
- **Make "found nothing" a failure**, never a zero. Every measurement helper
  should distinguish "measured and it is small" from "could not measure".

## Cost of ignoring it
The first failure produced a red gate, which is cheap. The second would have
produced eight *silently passing* gates on a freshly rebuilt hand - the exact
situation where losing detection is most expensive. That risk was large enough
to justify deferring a whole planned round until the gates could be rewritten.

## Transferability
Any project with automated checks over generated geometry: character
generators, procedural levels, modular kits, terrain, vehicles. The rule
generalises past geometry: *a test that locates its subject by an incidental
property of today's implementation will stop testing, not start failing, when
the implementation changes.*

## Related
- [[loop-matching-ties-in-procedural-stitching]]
