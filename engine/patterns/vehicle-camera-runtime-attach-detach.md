---
name: vehicle-camera-runtime-attach-detach
description: Third-person orbit camera added/removed as component at runtime on vehicle enter/exit — mouse orbit, scroll zoom, smooth Lerp follow
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: engine/patterns
  tags: [unity, camera, third-person, orbit, runtime, vehicle]
  severity: high
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# VehicleCamera Third-Person Orbit (Runtime Attach/Detach)

## When to use
Games where the player enters and exits a vehicle, and the camera should switch from first-person (foot) to third-person orbit (vehicle). Using a separate camera GameObject leads to state mismatch bugs; runtime component attachment eliminates them.

## Steps

**Enter vehicle — attach VehicleCamera:**
```csharp
// Called from VehicleInteractable.OnInteract()
var vehicleCam = playerCamera.gameObject.AddComponent<VehicleCamera>();
vehicleCam.Init(vehicleRoot.transform);
```

**VehicleCamera.Update() (orbit + zoom + follow):**
```csharp
void Update() {
    // Orbit
    yaw   += Input.GetAxis("Mouse X") * orbitSensitivity;
    pitch -= Input.GetAxis("Mouse Y") * orbitSensitivity;
    pitch = Mathf.Clamp(pitch, -45f, 85f); // prevent flip

    // Zoom
    currentDist = Mathf.Clamp(
        currentDist - Input.GetAxis("Mouse ScrollWheel") * scrollSensitivity,
        minDist, maxDist); // 4–15m range

    // Follow (smooth Lerp)
    Vector3 goalPos = target.position
        + Quaternion.Euler(pitch, yaw, 0f) * Vector3.back * currentDist;
    transform.position = Vector3.Lerp(transform.position, goalPos, followSpeed * Time.deltaTime);
    transform.LookAt(target.position + Vector3.up * lookAtHeightOffset);
}
```

**Exit vehicle — remove VehicleCamera:**
```csharp
Destroy(playerCamera.GetComponent<VehicleCamera>());
// Reset camera to player first-person state (see [[vehicle-enter-exit-choreography]])
```

**No separate camera GameObject:** the single player camera gets `VehicleCamera` added/removed. One camera object = one state.

## Why this works
`AddComponent` at runtime vs. pre-existing camera object: there's no "camera mode" state to synchronize. When VehicleCamera is present, it overrides transform. When destroyed, control returns to whatever ran before (PlayerController camera).

## Trade-offs
- Pitch clamped -45° to +85°: cannot look straight down or fully upward (prevent gimbal flip). Adjust clamps if your camera needs more range
- `Lerp` follow: at low `followSpeed`, camera lags noticeably behind fast vehicle — tune per vehicle speed
- `LookAt`: camera always looks at vehicle center. For large vehicles, `lookAtHeightOffset` should point at driver seat height, not wheel level

## Variants
- **Cinemachine VirtualCamera swap:** activate a VC in Cinemachine instead of component attach. More features, higher complexity. Good if already using Cinemachine
- **Fixed chase camera:** no orbit, just offset follow. Simpler for mobile/controller

See also: [[vehicle-enter-exit-choreography]], [[universal-camera-lock-canmove-flag]], [[camera-lock-save-lerp-restore]]
