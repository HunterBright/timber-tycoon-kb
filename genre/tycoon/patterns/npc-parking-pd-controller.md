---
type: pattern
project: timber-tycoon
suggested-category: genre/tycoon/patterns
tags: [unity, npc, parking, pd-controller, vehicle, physics]
date: 2026-05-17
status: draft
---

# NPC Parking PD Controller

## When to use
NPC vehicles that need to maneuver into specific parking slots, including back-in (reverse) parking. "Drive forward until slot, then stop" breaks down for any non-trivial slot geometry.

## Steps

**PD controller — applies force/torque toward slot target:**
```csharp
public class NPCParkingPDController : MonoBehaviour {
    [SerializeField] Transform slotTarget;
    [SerializeField] bool invertSlotForward = true; // true for back-in parking
    const float Kct = 5f;   // proportional correction gain
    const float Kd  = 0.4f; // derivative damping

    Rigidbody rb;

    void FixedUpdate() {
        // Angular error from target orientation
        Vector3 forward = invertSlotForward ? -slotTarget.forward : slotTarget.forward;
        float angErr = Vector3.SignedAngle(transform.forward, forward, Vector3.up);

        // PD torque
        float correctionTorque = Kct * angErr;
        float dampingTorque    = -Kd * rb.angularVelocity.y;
        rb.AddTorque(Vector3.up * (correctionTorque + dampingTorque));

        // Drive toward slot position
        Vector3 toSlot = slotTarget.position - transform.position;
        rb.AddForce(toSlot.normalized * driveForce);

        // Stop condition
        if (toSlot.magnitude < 0.3f && Mathf.Abs(angErr) < 5f && rb.velocity.magnitude < 0.1f)
            FinalizeParking();
    }

    void FinalizeParking() {
        rb.isKinematic = true;
        transform.position = slotTarget.position;
        transform.rotation = slotTarget.rotation;
        // Notify ParkingManager: slot occupied
    }
}
```

**Constants (tuned by trial for TT NPCPickup01):**
- `Kct = 5` — correction gain: higher = snappier turn, too high = oscillation
- `Kd = 0.4` — damping: prevents overshoot
- `invertSlotForward = true` for back-in slots (NPC reverses into slot)

**Forward axis fix:** NPC vehicle's "forward" = `-transform.right` (FBX export quirk, see [[forward-axis-blender-fbx-quirk]]). Compensate in PD controller by using `-transform.right` as the vehicle's heading vector.

**Stop → kinematic:** once parked, `isKinematic = true` snaps vehicle to exact slot position/rotation, stops physics simulation. Prevents post-park drift.

## Why this works
PD controller handles any starting approach angle — no "follow a script" sequence needed. The derivative term prevents oscillation (car swinging side to side). Kinematic snap on finish ensures visual precision.

## Trade-offs
- Constants tuned per vehicle type — NPCPickup01 constants won't work for a large truck (different mass/wheelbase). Per-vehicle `PDConfig` SO recommended
- Angular error near ±180° can cause wrap-around torque: add `angErr = Mathf.Clamp(angErr, -90, 90)` for safety
- Multiple vehicles parking simultaneously: each has independent PD, no inter-vehicle awareness — they may clip if two slots are close

## Variants
- **Pure kinematic waypoint parking:** no physics, just lerp from "approach point" → "parked position" — simpler, less realistic
- **Ackermann steering:** model turning radius accurately for realistic parking sequences

See also: [[forward-axis-blender-fbx-quirk]], [[pipeline-style-npc-spawn]]
