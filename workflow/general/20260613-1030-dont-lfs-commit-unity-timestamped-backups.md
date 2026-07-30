---
title: Don't commit timestamped Unity scene/asset backups into git-LFS
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-06-13'
project: Kerf - Sawmill Tycoon
tags:
- git
- git-lfs
- unity
- storage
- quota
- backups
- gitignore
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Don't commit timestamped Unity scene/asset backups into git-LFS

## The trap
Unity work generates lots of "safety backup" copies - `Demo_Scene.backup_2026-06-07_pre-x.unity`,
`Roads_backup_pre_v9_*.blend`, `_Backup_YYYY-MM-DD/…`. They're large binaries, so `.gitattributes`
LFS rules (`*.unity`, `*.blend`, `*.asset`) sweep them into LFS automatically, and they get committed
alongside real work. It feels safe ("everything's versioned!"), and locally it's invisible.

## Why it fails
git-LFS bills on **cumulative storage of every version ever pushed**, not the current tree. Each
~4-5 MB scene/blend backup is a near-duplicate of an asset already tracked, so backups multiply
storage fast. GitHub's free tier is only **1 GB storage + 1 GB/month bandwidth**; a project quietly
crosses it from *backups alone*. Measured on Timber Tycoon (2026-06-13): live tree 817 MB but full
LFS history 1.1 GB, with **~200 MB of it being timestamped `.unity`/`.blend` backups** (14 of the
top-25 largest LFS objects). That's the difference between staying free and needing a paid data pack.

## Symptoms
- `git lfs ls-files -s` / a size sweep shows many `*backup*` / `_Backup_*` entries near the top.
- Total LFS history materially larger than the current working tree (old versions piling up).
- LFS storage creeping toward 1 GB despite a modestly-sized actual project.

## Correct approach
- **gitignore backups** so they never enter LFS: e.g. `*.backup_*.unity`, `*_backup_*.blend`,
  `*_Backup_*/`, plus whatever timestamp convention the project uses. Keep backups locally, out of git.
- Git already *is* the version history - rely on commits/tags for rollback, not committed file copies.
- A normal feature push is tiny (Timber Tycoon: ~26 MB for 15 commits), so day-to-day pushing is never
  the problem - cumulative backup storage is. Audit with a per-extension LFS size breakdown periodically.
- To reclaim storage already in history: a deliberate history rewrite (git-filter-repo / BFG) to strip
  the backup paths - careful, coordinate with any collaborators, separate task.
