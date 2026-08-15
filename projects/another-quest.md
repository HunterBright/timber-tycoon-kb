---
title: Another Quest - Project Index
type: project-index
status: verified
confidence: high
verified: '2026-08-14'
date: '2026-08-14'
project: Another Quest
tags: [co-op, action-roguelite, rpg, fishnet, urp]
applies_to: []
source: ''
demo: pending
---

# Another Quest - Project Index

Co-op (1-4, solo pelnoprawne) action-roguelite RPG TPP, klimat isekai/gildii poszukiwaczy.
Unity 6000.5.1f1 / URP 17.5 + FishNet (listen-server, server-authoritative, 30 Hz).
Premium ~$20, PEGI 12, Steam. Demo za ~6-12 mies., bez deadline'u.

Repo (monorepo docs + Unity): `D:\Unity\AnotherQuest` — prywatne
`github.com/HunterBright/another-quest`. Utworzone 2026-08-13/14.

Source of truth dla stacku: errata `t` w `docs/errata/00_INDEKS_ERRAT.md`
(GDD v3 i Instrukcje v0.5 niosa stara wersje 6000.4.8f1 = EOL, sa za STALE-pinem).

## Hierarchia zrodel prawdy w tym projekcie

errata > KONFLIKTY.md > Modul_NN_*_LOCKED.md > GDD v3 (tylko sekcja 2) > Instrukcje.
`docs/archive/` NIE jest kanonem. **Nic nie jest kanonem, dopoki nie jest w repo** —
erraty p/q/r raz juz zginely w czacie claude.ai (odzyskane 2026-08-13).

Kanon = 20 modulow LOCKED. Jedyna legalna droga zapisu do modulu: `tools/canon_apply.py`.
Zmiana kanonu inaczej niz erratą jest blokowana przez `permissions.deny`.

## Stan maszynowy (nie duplikowac w prozie)

`state/canon_manifest.json` (hash per modul, `--check` = detekcja STALE),
`state/findings.json`, `state/assumptions.json` (82 decyzje delegowane),
`state/feature_matrix.json`, `state/open_questions.json`.

## Kierunek artystyczny (stan 2026-08-14)

Low-poly stylizowany, **proporcje heroiczne ~5 glow — NIE chibi** (errata `x`).
Faceted low-poly, plaskie baked diffuse, paleta per biom, sprzet widoczny na modelu.

Pipeline referencji → mesh:
1. **ComfyUI lokalnie** (`D:\AI\ComfyUI`): Qwen-Image + Qwen-Edit 2509 + LoRA Multiple-angles.
   Generator w repo: `tools/gen_ref_anchor.py`. **Wymagana flaga `--disable-smart-memory`**
   (patrz lekcja o zakleszczeniu na VRAM).
2. **Hunyuan3D web (Tencent)**, Image-to-3D, tryb **Multiple Images** — pobierane recznie.
3. Rig + animacje lokalnie.

Kanal `unity-mcp` (Unity AI Assistant) **NIE ISTNIEJE** — pakiet `com.unity.ai.assistant`
wypadl z KERF-a 2026-08-05, wpis serwera MCP usuniety 2026-08-14. Kazdy dokument
mowiacy "Hunyuan przez unity-mcp GenerateMesh" jest historyczny.

Anchor stylu: `art/goblin_osilek/refs/` (3 kandydatki, seedy 110814001-3).

## Ryzyko #2: animacje nie-humanoidow

Kimodo animuje **tylko humanoidy**. Czworonogi = Rigify `basic_quadruped` + Animal Animator
(+RootMaker), fallback CloudRig. **Animal Animator i RootMaker nie sa jeszcze zainstalowane**
(stan 2026-08-14). Spike weryfikacyjny: niedzwiedz-boss.

## Roster slice (las) — kanon M05 + errata `u`

Goblin Lucznik / Goblin Osilek (Physical), Jadowy Strzelec / Lesny Pajak (Trucizna),
Mrozny Wisp (Lod) + boss Niedzwiedz (Physical, ~3 m srednicy, 2 fazy, parowalna szarza).
Bandyci przeniesieni na lochy wyzszych rang. Kompozycja (3 dystans / 2 zwarcie,
2T/2P/1L) jest LOCKED — nazwy i wyglad naleza do ART.

## Pamiec projektowa (poza KB)

`%USERPROFILE%\.claude\projects\<katalog repo AQ>\memory\` — stan biezacy, feedback,
decyzje w toku. KB trzyma wylacznie wiedze przenoszalna miedzy projektami.

Powiazane: [[cross-project-stack-reuse]], [[timber-tycoon]]
