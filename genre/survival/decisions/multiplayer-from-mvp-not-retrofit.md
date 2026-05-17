---
name: multiplayer-from-mvp-decision
description: Eskimo Simulator builds MP from MVP using FishNet + NetworkBehaviour from day 1. Retrofitting SP codebase to MP = rewriting most game logic. Even solo play runs as host-of-1.
metadata:
  type: decision
  project: eskimo-simulator
  suggested-category: genre/survival/decisions
  tags: [game-design, multiplayer, architecture, networking, eskimo]
  severity: high
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Multiplayer from MVP — Not Retrofit

## Decision

**Chosen:** Build Eskimo Simulator with multiplayer architecture from the very first commit. FishNet + NetworkBehaviour from day 1.

## Options considered

| Option | Description | Verdict |
|--------|-------------|---------|
| SP first, MP later | Ship single-player MVP, retrofit multiplayer in v1.5 | REJECTED |
| MP from MVP | All systems NetworkBehaviour-based from day 1 | SELECTED |

## Why retrofit MP always fails

Single-player code makes assumptions that break under networking:

1. **Singletons everywhere** — `GameManager.Instance.AddMoney()` assumes one authority. Networked: who is the authority?
2. **Instant state updates** — `player.health -= 10` assumes no latency. Networked: needs authority check + RPC.
3. **No rollback design** — physics, inventory, crafting all assume single-truth. Networked: needs server reconciliation.
4. **ISaveable attached to clients** — save data must be server-authoritative, not per-client.

Retrofitting = touching EVERY system that has state. Effectively rebuilds the game.

## MP from MVP architecture

```
Stack: Unity 6000.3.5f1 + URP + FishNet + ServiceLocator + GameEventSO + ISaveable + NetworkBehaviour

Listen-server model: host runs server + client simultaneously.
Solo players: run as host-of-1 (same code path as multiplayer — zero branch).
```

`NetworkBehaviour` extends `MonoBehaviour`. All gameplay systems inherit from it. `[ServerRpc]` / `[ClientRpc]` annotations handle authority.

**Compatibility with TT stack:**
- ServiceLocator: unchanged (services are server-authoritative)
- GameEventSO: `NetworkGameEventSO` variant raised server-side, clients subscribe
- ISaveable: server saves state, clients receive on load

## Cost vs. benefit

**Cost:** ~20% additional upfront dev time (NetworkBehaviour more complex than MonoBehaviour).

**Benefit:** No existential refactor at v1.5. The #1 reason indie studios fail to add MP to shipped games is that the codebase fights the addition. This cost pays that debt upfront.

## Cross-project lesson

If multiplayer appears ANYWHERE in a project's roadmap — even "maybe in year 2" — design for it from MVP. The cost of retrofitting grows super-linearly with codebase size.

See also: [[parallel-architecture-pattern]], [[cross-project-stack-reuse]], [[chunk-based-world-loading]]
