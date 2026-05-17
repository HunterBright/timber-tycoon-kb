---
name: cross-project-consistency-stack-reuse
description: Eskimo Simulator uses identical core stack to TT (Unity 6000.3.5f1 + URP + ServiceLocator + GameEventSO + ISaveable). Known toolchain = transferable skills, same workflow, faster Eskimo MVP.
metadata:
  type: lesson
  project: eskimo-simulator
  suggested-category: workflow/claude-code
  tags: [claude-code, architecture, reuse, eskimo, multiproject]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [claude-code-projects, unity-projects]
---

# Cross-Project Stack Consistency (TT → Eskimo)

## Decision

Eskimo Simulator uses the same core technology stack as Timber Tycoon. Not because of inertia — because reuse compounds skill investment.

## Shared stack

| Component | TT | Eskimo |
|-----------|-----|--------|
| Unity version | 6000.3.5f1 | 6000.3.5f1 (locked to TT's tested version) |
| Render pipeline | URP | URP |
| Architecture | ServiceLocator + GameEventSO + ISaveable | Same |
| Dev pipeline | Claude Code | Claude Code |
| 3D modeling | Blender | Blender |
| Audio generation | ElevenLabs | ElevenLabs |
| Version control | Git + LFS | Git + LFS |

## Eskimo additions (not replacements)

- **FishNet** — multiplayer networking layer
- **NetworkBehaviour** — extension of MonoBehaviour for MP state
- **Addressables** — chunk streaming for 6km² map
- **Inuktitut voice** — ElevenLabs native language for Inuit NPCs

These extend the stack, they don't replace it. ServiceLocator still works. GameEventSO still works (as NetworkGameEventSO variant). ISaveable still works (server-authoritative).

## Benefits of stack consistency

1. **Zero toolchain re-learning:** Hunter + Code team already knows this stack
2. **Reusable CLAUDE.md:** ~80% of TT's CLAUDE.md transfers to Eskimo
3. **Reusable Editor scripts:** many utility scripts work across projects
4. **Faster MVP:** 3 months of TT project knowledge doesn't reset for Eskimo

## Anti-pattern avoided

"New game = try new engine / new architecture." This sounds ambitious but means:
- Re-learning Unity → Unreal = 6-month toolchain delay
- New architecture patterns = re-establishing conventions, making the same mistakes again
- Claude Code loses project-specific knowledge built up over TT's development

## Constraint acknowledged

Locked to Unity 6000.3.5f1 until Eskimo ships. No experimenting with Unreal, no Unity 7 migration mid-project. Stable toolchain > latest toolchain.

See also: [[parallel-architecture-pattern]], [[multiplayer-from-mvp-decision]], [[game-state-machine-pattern]]
