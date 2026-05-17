---
name: vehicle-enter-exit-choreography
description: Vehicle enter/exit sequence — disable player scripts → parent player to vehicle → detach camera → reverse on exit with left-side placement
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: engine/patterns
  tags: [unity, vehicle, enter, exit, choreography, sequence]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Vehicle Enter/Exit Choreography Sequence

## When to use
Any FPS/third-person game where the player enters a vehicle. Without a defined sequence, player scripts fight with vehicle scripts, camera floats to wrong state, or player spawns inside the vehicle mesh on exit.

## Steps

**Enter sequence (called from VehicleInteractable.OnInteract):**
```csharp
void EnterVehicle(Transform vehicleRoot) {
    // 1. Disable player scripts (order matters — disable before re-parenting)
    player.GetComponent<PlayerController>().enabled   = false;
    player.GetComponent<CharacterController>().enabled = false;
    player.GetComponent<PlayerInteraction>().enabled  = false;

    // 2. Parent player to vehicle (hidden inside cabin)
    player.transform.SetParent(vehicleRoot);
    player.transform.localPosition = Vector3.zero; // center in cabin

    // 3. Detach camera from player
    camera.transform.SetParent(null);

    // 4. Attach VehicleCamera to camera object
    camera.gameObject.AddComponent<VehicleCamera>().Init(vehicleRoot);
}
```

**Exit sequence (1s hold E):**
```csharp
void ExitVehicle() {
    // 1. Destroy vehicle camera
    Destroy(camera.GetComponent<VehicleCamera>());

    // 2. Place player BEFORE re-enabling CharacterController
    //    (CC will override position on re-enable)
    Vector3 exitPos = vehicle.position
        + vehicle.right * -1.5f  // left side of vehicle
        + vehicle.up * 0.5f;     // slightly above ground
    player.transform.position = exitPos;
    player.transform.SetParent(null);

    // 3. Re-attach camera to player
    camera.transform.SetParent(player.transform);
    camera.transform.localPosition = eyeLevel; // restore eye height

    // 4. Re-enable player scripts (AFTER position set)
    player.GetComponent<CharacterController>().enabled = true;
    player.GetComponent<PlayerController>().enabled   = true;
    player.GetComponent<PlayerInteraction>().enabled  = true;
}
```

**Critical ordering rules:**
- Disable scripts BEFORE re-parenting (avoid frame where both player and vehicle update transform)
- Set position BEFORE re-enabling CharacterController (CC sets position on enable)
- Destroy VehicleCamera BEFORE re-attaching camera to player (avoid competing transform updates)

## Why this works
Explicit sequence with defined ordering eliminates race conditions between player and vehicle transforms. Each step is deterministic — no frame-order dependency.

## Trade-offs
- Exit position `vehicle.right * -1.5f`: positions player on vehicle's left side. If vehicle is against a wall, player may spawn inside wall — add a clear-space check before exiting
- `hold E 1s` for exit: prevents accidental exit while using E for interaction. Time configurable per game feel
- CharacterController reset: if player exits mid-slope, CC may immediately slide. Add a brief grace period where CC is kinematic after re-enable

## Variants
- **Door animation**: trigger door open anim before EnterVehicle(), door close after
- **Camera cutscene**: briefly fade-to-black, reposition, fade-in — hides abrupt camera transition

See also: [[vehicle-camera-runtime-attach-detach]], [[get-or-add-component-pattern]]
