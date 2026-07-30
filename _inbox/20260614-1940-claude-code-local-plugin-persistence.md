---
title: Loading a LOCAL Claude Code plugin permanently (marketplace, not --plugin-dir)
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-06-14'
project: Kerf - Sawmill Tycoon
tags:
- claude-code
- plugins
- marketplace
- settings
- local-plugin
- gotcha
applies_to: []
source: ''
severity: medium
suggested-category: tooling/lessons
---

# Loading a LOCAL Claude Code plugin permanently (marketplace, not --plugin-dir)

## Context
Had a local plugin folder (`.claude-plugin/plugin.json` + `skills/`, no marketplace.json) loaded only via the dev flag `claude --plugin-dir ./folder`. Wanted it to load every session without the flag, configured in project `.claude/settings.json`.

## Lessons (each cost time to verify)

1. **There is NO `pluginDirs` settings key.** An AI "guide" confidently recommended `pluginDirs` — it does not exist in the official settings JSON schema (`json.schemastore.org/claude-code-settings.json`). Writing it fails silently (plugin never loads). **Always verify a settings key against the schema or docs before writing it — don't trust a single summary.**

2. **A local plugin loads only two ways** (per official docs): `--plugin-dir` (duration of one session) OR through a **marketplace** (future sessions). Persistent loading therefore REQUIRES a `marketplace.json` catalog file somewhere — a bare plugin folder is not enough.

3. **Relative plugin source rule.** In `marketplace.json`, a plugin's relative `source` must start with `./` and resolve INSIDE the marketplace root (the dir containing `.claude-plugin/`). To register a pre-existing folder you don't want to move, put `marketplace.json` at an ANCESTOR dir — e.g. repo-root `.claude-plugin/marketplace.json` with `"source": "./that-folder"`. Symlinks pointing outside the marketplace are skipped by design, so a sibling/sub folder can't reach it.

4. **Settings shapes (schema-verified):**
   - `extraKnownMarketplaces`: object keyed by marketplace name → `{ "source": { "source": "directory", "path": "<dir containing .claude-plugin/marketplace.json>" } }` (local variants: `"directory"` or `"file"`).
   - `enabledPlugins`: object keyed by `"plugin-name@marketplace-name"` → `true`.

5. **GOTCHA — `/reload-plugins` ≠ install.** `/reload-plugins` only refreshes ALREADY-installed plugins. It does NOT register a freshly-declared `extraKnownMarketplaces` nor install its plugin (verified: `known_marketplaces.json` and `installed_plugins.json` were untouched after reload, no new skills appeared). To actually load a newly-declared marketplace plugin: **restart** Claude Code (startup processes `extraKnownMarketplaces`, may prompt trust/install) OR run `/plugin marketplace add .` → `/plugin install name@marketplace` → `/reload-plugins`.

6. **Marketplace plugins are copied to cache** (`~/.claude/plugins/cache`) — a SNAPSHOT, not the live in-place behavior of `--plugin-dir`. To pick up later edits to the source folder: `/plugin marketplace update <name>` + `/reload-plugins`.

## Takeaway
"Make a local plugin permanent" = wrap it in a local marketplace (`extraKnownMarketplaces` + `enabledPlugins`), put the catalog at an ancestor of the plugin folder, and remember that activation needs a restart/install — a reload alone won't do it. Verify every settings key against the schema; AI guides hallucinate plugin-config keys.
