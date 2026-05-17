---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, interactable, kiosk, placeholder, ui]
date: 2026-05-17
status: draft
---

# KioskInteractable + Cube Placeholder Pattern

## When to use
Any game with diegetic UI interaction points (shop, upgrade station, quest board) that need to be playable immediately before final art exists. Using a cube placeholder + IInteractable = positioned and playable in minutes.

## Steps

**Cube placeholder setup:**
1. Create → 3D Object → Cube → rename to `UpgradeKiosk_Placeholder`
2. Position at intended location (e.g., near sales counter)
3. Remove default MeshRenderer's default material — assign a visible placeholder material
4. Add `BoxCollider` (already has one by default — keep it as raycast target)
5. Add `KioskInteractable` script (IInteractable)

**KioskInteractable script:**
```csharp
public class KioskInteractable : MonoBehaviour, IInteractable {
    [SerializeField] string tooltipKey = "UI.Kiosk.Tooltip"; // "Open Upgrade Shop"

    public string GetTooltip() => LocalizationManager.Get(tooltipKey);

    public void OnInteract(PlayerInteraction player) {
        UpgradeShopUI.Instance.Open();
        playerController.canMove = false; // lock movement (see [[universal-camera-lock-canmove-flag]])
    }
}
```

**UpgradeShopUI.Instance.Open()** → panel slides in, blocks gameplay input.

**Art replacement workflow:**
1. Model proper kiosk (notice board, terminal, etc.) in Blender → FBX → Unity
2. Create `UpgradeKiosk.prefab` with new mesh
3. In scene: copy world position/rotation from cube placeholder
4. Replace Cube primitive GameObject with new prefab
5. Remove placeholder

**Debug shortcut** (editor-only, remove after kiosk is functional):
```csharp
#if UNITY_EDITOR
void Update() {
    if (Input.GetKeyDown(KeyCode.U)) OnInteract(null); // test without walking to kiosk
}
#endif
```

## Why this works
`IInteractable` interface separates the interaction logic from the mesh representation. Cube has a BoxCollider — the raycasting system can find it immediately. Art replacement is purely a visual swap, behavior unchanged.

## Trade-offs
- Cube placeholder is visible in screenshots: use a colored, semi-transparent material to signal "work in progress," or hide with a camera layer in demo builds
- `playerController.canMove = false`: must be re-enabled when the UI closes. Wire `OnPanelClose` event → `canMove = true`
- Multiple kiosks: if more than one kiosk type is added, generalize `KioskInteractable` with a serialized `System.Action onInteract` or `UnityEvent`

## Variants
- **Quest board kiosk:** same pattern, `OnInteract → QuestBoardUI.Open()`
- **Notice board:** same pattern, `OnInteract → DialogueUI.StartDialogue(noticeSO)`

See also: [[diegetic-3d-button-raycast]], [[universal-camera-lock-canmove-flag]]
