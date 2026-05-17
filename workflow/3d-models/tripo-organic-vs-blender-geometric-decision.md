---
type: decision
project: timber-tycoon
suggested-category: workflow/3d-models
tags: [tripo, blender-mcp, asset-pipeline, decision-record]
date: 2026-05-17
status: draft
---

# Tripo (organic) vs Blender MCP (geometric) — Pipeline Routing Decision

## Context

Two 3D asset generation pipelines are available:
- **Tripo AI** — text-to-3D, fast, produces organic forms with natural surface variation
- **Blender MCP** — Python-scripted procedural geometry, precise, deterministic

Both can theoretically produce any asset, but each has a clear strength profile. Without a routing rule, engineers default to personal preference and waste time fighting the wrong tool.

## Options Considered

**Option A — Tripo for everything:**
Fast iteration, but fails on geometric/architectural assets. Symmetry issues appear unpredictably (see [[tripo-asymmetric-floating-retopo]]). Mechanical parts (machines, vehicles) come out warped or asymmetric, requiring expensive retopo or manual cleanup. Not worth it for geometric shapes.

**Option B — Blender MCP for everything:**
Precise and reproducible, but hand-scripting organic shapes (bark grain, leaf clusters, irregular rock surfaces) takes days compared to Tripo's minutes. Over-engineers natural variation that Tripo handles automatically.

**Option C — Shape-led routing:**
Match the tool to the asset's shape class:
- **Organic** → Tripo (trees, rocks, characters, irregular props, terrain features)
- **Geometric** → Blender MCP (machines, vehicles, buildings, racks, furniture)

## Decision

**Option C — shape-led routing.**

The primary classification question when starting a new asset: *Is this shape primarily organic or primarily geometric?*

**Organic assets** (use Tripo): trees, stumps, logs, rocks, dirt clumps, characters, irregular props, food items, natural terrain features.

**Geometric assets** (use Blender MCP): machines (pelletizer, chipper, plank maker), vehicles (Kei truck, NPC pickup), buildings (sawmill), racks (LogRack, FirewoodRack, StumpRack, BagRack), counters, roads, bridges, straight-edged props.

## Consequences

- Pipeline branches per asset type — onboarding a new asset starts with shape classification, not tool selection
- Hybrid assets (wooden barrel = geometric cylinder + organic wood grain) → Blender MCP for base geometry + Tripo-sourced texture or procedural Cycles material for surface detail
- Tripo output always needs retopo review (see [[tripo-cleanup-pipeline]]) before Unity import
- Blender MCP output always needs UV unwrap + texture bake before FBX export (see [[procedural-textures-need-bake]])

## Revisit when

A new 3D AI tool emerges that handles both shape classes well without retopo overhead, OR Blender MCP gets faster procedural shaders that rival Tripo's organic surface quality.

## See also

[[tripo-asymmetric-floating-retopo]], [[tripo-cleanup-pipeline]], [[procedural-textures-need-bake]], [[blender-headless-python-generation]]
