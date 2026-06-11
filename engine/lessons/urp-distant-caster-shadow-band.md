---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [urp, shadows, shadow-acne, terrain, roads, coplanar-meshes, shadow-distance, diagnostics, day-night]
date: 2026-06-11
status: validated
---

# "Dark band that follows the player" = terrain self-shadow leaking onto near-coplanar road meshes

## Symptom
A dark shadow-like band sits on the road tens of meters ahead of the camera and moves with the
player. Present at normal sun heights (confirmed at 40.6° elevation), strongest on roads, not on
open terrain.

## Root cause (confirmed by live in-game A/B toggle)
Road meshes were face-copies of the terrain raised only ~2 cm above it. The TERRAIN had
`Cast Shadows = On`. Shadow-map texel precision falls with distance from the camera (later
cascade + map stretched over the full shadow distance), so beyond some camera-relative distance
the terrain's own shadow depth gets imprecise enough to leak onto the geometry layered millimeters
above it. The leak boundary is camera-relative → the band follows the player. Only the
near-coplanar layer (roads) shows it; everything else has clearance.

Distant mountain casters were only a **secondary** contributor: giant skyline meshes outside the
shadow distance can produce clipped-window artifacts, but only at very low sun (<8° elevation).
Their `Cast Shadows = Off` fix was valid hygiene, not the cause of the band.

## Fix
`Cast Shadows = Off` on the terrain renderer (+ backdrop mountains as hygiene). The terrain still
RECEIVES shadows normally; its own cast shadows on gentle low-poly relief were imperceptible.
(Alternatives if terrain shadows are needed: raise depth/normal bias, raise the co-planar layer,
or shrink shadow distance — all with worse trade-offs.)

## Two transferable diagnostic lessons
1. **Editor-view captures are NOT ground truth for shadow bugs.** A Scene-View A/B capture test
   confidently pointed at the distant mountain casters. The Game view told a different story —
   see [[scene-view-ab-false-positive-game-view-ground-truth]]. Also: offscreen RenderTexture
   renders in Edit Mode showed NO realtime shadows at all — dead end.
2. **The decisive tool was a runtime group-toggle harness**: an editor-only, self-bootstrapping
   MonoBehaviour (RuntimeInitializeOnLoadMethod inside #if UNITY_EDITOR — scene file untouched,
   nothing persists) with hotkeys toggling Cast Shadows per renderer group (terrain / roads /
   trees / props / EVERYTHING-except-player as the null test) + an overlay showing states and live
   sun elevation. The user plays, presses keys at the bug spot, and the culprit group is found in
   minutes — with the sun elevation recorded for the report.

## Rule of thumb
When stacking a mesh a few centimeters above another (roads, decals-as-geometry, rugs, paths):
the LOWER mesh's cast shadows will eventually leak onto the upper one at distance. Either the
lower mesh shouldn't cast, or the gap must exceed the far-cascade depth error.

## See also
[[scene-view-ab-false-positive-game-view-ground-truth]], [[urp-shadow-cascade-tuning]], [[debugging-search-first-trust-render-check-upstream]]
