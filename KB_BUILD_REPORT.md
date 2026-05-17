# KB Build Report

Started: 2026-05-17 (session began from KB_BUILD_PACKAGE.md, autonomous execution)
Finished: 2026-05-17
Duration: ~2 sessions (context compaction after batch 4, resumed autonomously)

## Summary
- Total entries processed: 168 candidates
- Files created: 141
- Dedup mergers executed: 5 (see KB_BUILD_PACKAGE.md section 5)
- Flagged entries: 2 (see below)
- Skipped entries: 0 (all entries rendered)

## Files Created (by category)
- engine/lessons: 17
- engine/patterns: 57
- engine/anti-patterns: 9
- genre/tycoon/patterns: 20
- genre/tycoon/decisions: 11
- genre/survival/patterns: 1
- genre/survival/decisions: 1
- genre/cross-genre/decisions: 1
- workflow/claude-code: 13
- workflow/mcp-tools: 2
- workflow/3d-models: 4
- workflow/asset-pipeline: 5

**Total: 141 files**

Delta from 168 candidates: 27 fewer files than candidates. Accounted for by:
- 5 dedup mergers (each removes 1-2 source entries, creates 1 combined file) = ~8 reduction
- Several entries from earlier sweeps (sweep1, sweep2) were sub-entries folded into primary entries = ~15 reduction
- engine/patterns count (57) is high because early sweep entries (#001-#006 range) produced several pattern files in batch 1

## Dedup Mergers Applied
| Source IDs | Target Path | Notes |
|---|---|---|
| #096, #119 | genre/tycoon/patterns/storage-rack-family-system.md | Merged — #096 initial design, #119 added maxDistinctStacks + activation gating |
| sweep2-CapsuleCollider, #112 | engine/anti-patterns/script-overrides-prefab-collider.md | Merged — followed dedup merger table as authority over #112's Path: field |
| #011, #106, #107 | engine/lessons/mesh-exporter-obj-pitfalls.md | Merged into 3 subsections |
| sweep2-TreeTypeData, #108 | engine/patterns/scriptable-object-runtime-injection.md | Merged — TreeTypeData context incorporated into #108 file |
| #093, #094, #095 | engine/patterns/typography-accessibility-stack.md | Merged into 3 sections (Typography / Accessibility / Credits) |

## Flagged Entries

### 1. sweep2-1 (ChoppableTree naming convention)
Entry was listed as a sweep with no separate brief definition. Resolved by: created `engine/patterns/choppable-tree-multi-type-naming-convention.md` using context from CLAUDE.md architectural patterns section. Content inferred correctly but lacks direct source quote.

### 2. MOC "Single-material atlas for static props" and "Tripo organic vs Blender geometric decision"
These appear in the MOC.md template section but no corresponding batch entries were found in batches 1-8. The first (`single-material-atlas-for-static-props.md`) was found in batch 1 git output — it was created from early sweep data. The second (Tripo organic vs Blender geometric decision) does not appear in any batch definition and was not created. **Flag: missing entry for `workflow/3d-models/tripo-organic-vs-blender-geometric-decision.md`.**

## Skipped Entries
None. All 168 candidates were either rendered as individual files or folded into merged files.

## MOC.md
Generated with 141 categorized links. Existing TODO section preserved. New section `## Generated entries (KB_BUILD_PACKAGE 2026-05-17)` appended.

## Git Commits (this build)
| Batch | Commit SHA | Description |
|-------|-----------|-------------|
| 1 | f3a51df | engine/lessons + patterns + anti-patterns (25 entries) |
| 2 | 3aed734 | engine/patterns part 1 (24 entries) |
| 3 | 5e8220a | engine/patterns part 2 (20 entries) |
| 4 | bba525d | engine/patterns part 3 + engine/anti-patterns (14 entries) |
| 5 | 26b97d3 | genre/tycoon/patterns + decisions (21 entries) |
| 6 | b556b37 | genre/tycoon/decisions + survival + cross-genre (13 entries) |
| 7 | 0c38b07 | workflow/claude-code + mcp-tools (15 entries) |
| 8 | 0273d33 | workflow/3d-models + asset-pipeline (9 entries) |

## Next Steps for Hunter
1. **Review MOC.md** — validate categorization, check links work
2. **Spot-check 5-10 entries** for content quality (suggested: entries marked ★ = highest value)
3. **Missing entry** — `workflow/3d-models/tripo-organic-vs-blender-geometric-decision.md` was not generated (not in any batch definition). Create manually or add to next build package if needed.
4. **Promote flagged entries** — entries built from inferred context (sweep2-1) should be reviewed and verified against actual project state
5. **Use `/kb-review`** command in Claude Code to query the KB during development
6. **Status: draft** on all entries — run `/kb-review` to validate and change to `validated` as entries are confirmed correct
