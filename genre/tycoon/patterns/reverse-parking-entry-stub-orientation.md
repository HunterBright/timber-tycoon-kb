---
type: pattern
project: timber-tycoon
suggested-category: genre/tycoon/patterns
tags: [unity, npc, parking, kinematic, waypoints, orientation, fbx-forward-axis]
date: 2026-05-22
status: draft
---

# Reverse-parking: orientation on the entry stub (forward-tail)

**Pattern.** In kinematic reversing along a presampled curve, where `path[0]` = the car's stop point (nose-first approach) and `path[1..]` = the reversing waypoints: the first segment `path[0] -> WP_00` is a **forward-tail**, not part of the rear-first maneuver. Applying the rear-first orientation rule (nose perpendicular to the tangent) to this stub forces a ~180° nose rotation that unwinds after WP_00 -> a visible double spin on every slot.

## Symptom
- The car arrives and parks in the correct spot (final state OK), but during reversing the nose "pirouettes" ~2x.
- Happens on EVERY slot, regardless of side -> common geometric cause, not per-slot.

## Root cause
- The stub `path[0]->WP_00` has a tangent aligned with the approach direction (e.g. south). The rear-first rule computes `desiredFwd = Cross(tangent, up)` and, due to the FBX quirk (nose = `-transform.right`, see [[forward-axis-blender-fbx-quirk]]), the nose orientation target lands ~90-180° away from the approach orientation.
- The net rotation is correct (start/end match) -> that's why the endpoints look fine and the spin is easy to miss when only checking endpoints.

## Fix
Apply the rear-first rule only **from WP_00 onward**; on the stub hold `arrivalRot`. Gate on the **curve index**, not on distance -> robust against an arbitrarily short stub.

```csharp
const int SUBDIV = 12;          // WP_00 = curve[SUBDIV]
// currentK = curve segment index this frame (arc-length walk)
Quaternion targetRot;
if (currentK <= SUBDIV)         // entire stub path[0]->WP_00
    targetRot = arrivalRot;     // hold approach orientation, zero rotation
else                            // back-in from WP_00 upward
    targetRot = Quaternion.LookRotation(Vector3.Cross(tangent, Vector3.up), Vector3.up);
transform.rotation = Quaternion.Slerp(transform.rotation, targetRot, 6f * Time.deltaTime);
```

Why index, not distance: the stub can be shorter than one frame on a fast stop; a gate on `targetDist` can skip it. `currentK <= SUBDIV` hits the boundary exactly at WP_00 regardless of stub length. Bonus: arrival and the WP_00 target are convergent (~0-2°), so even skipping the stub in a single frame produces no flip.

## Diagnostics (discriminator)
- Endpoints lie. Log the **cumulative, unwrapped nose yaw** and compare with the net rotation. Here: net ~87°, but total sweep ~460° -> 5x too much = hidden spin. Start/end alone showed ~87° and looked clean.
- "The fix didn't change behavior" = the fix didn't execute (stale assembly / compile error / unsaved edit), NOT "the fix doesn't work". Verify the edit actually made it into the build (Console: red errors? does the file actually contain the new code?) before declaring failure.

## Process (AI rework)
- Give the AI a **clean vs broken** map as ground truth, so it doesn't rewrite working phases. The data proved: approach (physics), pull-past, back-in body from WP_00 = clean; the only broken part = ~12 frames of stub orientation. Without that map, "rewrite from scratch" reproduces the same conflict under a different name.
- Enforce **audit + plan before code**. The AI's first plan had a factual error (claimed arrival and WP_00 are 90° apart -> they are ~0°). Caught at the plan stage, not after implementation.

## See also
- [[forward-axis-blender-fbx-quirk]] -- nose = `-transform.right`, source of the offset in `Cross(tangent,up)`.
- [[npc-parking-pd-controller]] -- approach / pull-past phases (physics), precede this maneuver.
- [[navmesh-plus-kinematic-waypoints]] -- physics->kinematic handoff, construction of `path[]/curve[]`.

File: `Assets/Project/Scripts/NPC/Traffic/NPCVehicle.cs` -> `KinematicReverseAlongPath`.
Diagnostics `[RevDiag START/END]` left in code behind an `if (revDiag)` gate (default `false`) -> resurrect by flipping to `true`.
