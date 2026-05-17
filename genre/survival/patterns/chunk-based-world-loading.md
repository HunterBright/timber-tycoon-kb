---
type: pattern
project: eskimo-simulator
suggested-category: genre/survival/patterns
tags: [unity, world-streaming, chunks, survival, eskimo]
date: 2026-05-17
status: draft
---

# Chunk-Based World Loading (Eskimo Simulator)

## When to use
Large open-world survival games where the map exceeds single-scene capacity (>1km² / >3 min load time). Use when: map has multiple distinct zones, player doesn't need all zones loaded simultaneously, memory budget < all-zones-loaded.

## Context: TT vs Eskimo

| | Timber Tycoon | Eskimo Simulator |
|--|-------------|-----------------|
| Map size | ~600×650m | ~6km² (6 zones × 1km²) |
| Architecture | Single scene (`Demo_Scene.unity`) | Chunk-based streaming |
| Load strategy | Full map in memory always | 16 chunks visible at any time |
| Memory | ~2GB for all content | ~2GB for 16 active chunks |

Single-scene works for TT. Eskimo's 6km² single-scene = ~8GB RAM + 3-minute loads = unacceptable.

## Steps

**Chunk definition:**
- 256m × 256m grid tile
- Each chunk is an Addressable asset (Unity Addressables system)
- Contents: terrain mesh, scattered props, ambient zones, NPC spawn points
- Save persistence: chunk saves independently (edits to one chunk don't bloat global save)

**Streaming manager:**
```csharp
public class ChunkStreamingManager : NetworkBehaviour {
    const float ChunkSize = 256f;
    const int VisibleRadius = 2; // 4×4 = 16 active chunks

    void Update() {
        Vector2Int playerChunk = WorldToChunk(player.position);
        LoadChunksAround(playerChunk, VisibleRadius);
        UnloadChunksOutside(playerChunk, VisibleRadius + 1);
    }

    Vector2Int WorldToChunk(Vector3 pos) =>
        new Vector2Int(Mathf.FloorToInt(pos.x / ChunkSize),
                       Mathf.FloorToInt(pos.z / ChunkSize));
}
```

**LOD per chunk:**
- Near chunks (<256m): full detail models
- Mid-range (256-512m): medium-poly variants
- Far chunks (>512m): low-poly impostors (billboard sprites)

**Addressables workflow:**
- Each chunk as Addressable group
- Async load/unload (`Addressables.LoadAssetAsync`)
- Progress callback for loading screen (zone transition ~0.5s)

## Trade-offs

- **Complexity:** Addressables has learning curve; chunk boundary edge cases (object spans two chunks). Worth it at 6km².
- **Multiplayer complication:** different players in different chunks = different things loaded. Server must track which clients have which chunks loaded before sending events.
- **Not needed for TT:** TT's single scene is correct for its scale. This pattern is overkill at <1km².

## Variants
- **Hub-and-spoke without streaming:** small central hub + 6 distinct zones loaded on-demand (scene transition, loading screen). Simpler, less seamless. Valid for zone-boundary game design.

See also: [[multiplayer-from-mvp-decision]], [[cross-project-stack-reuse]]
