---
name: tripo-firewood-log-vocab
description: Tripo AI interprets "firewood log" as full tree trunk. For TT firewood pieces, use "wedge-shaped piece of wood, triangular cross section, quarter of a log." Vocab table for common mistakes.
metadata:
  type: lesson
  project: timber-tycoon
  suggested-category: workflow/3d-models
  tags: [tripo, ai-3d, prompt-engineering, vocabulary]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [blender-pipelines]
---

# Tripo Vocab: Firewood = Wedge-Shaped Piece

## What happened

Prompt: "firewood log" → Tripo generates a 3-meter cylinder (full tree trunk shape).
Expected: a small wedge-shaped piece — the "product to sell" version of firewood in TT.

Tripo is trained on common internet images. "Firewood log" → most images are actual logs. TT's use is niche (already-split firewood piece), requiring description-first prompts.

## Vocabulary translation table

| Game term | Tripo prompt |
|-----------|-------------|
| Firewood | "wedge-shaped piece of wood, triangular cross section, quarter of a log, freshly split" |
| Plank | "rectangular wooden board, flat and smooth, lumber board" |
| Bark | "tree bark fragment, curved, rough fibrous outer layer, brown" |
| Pellet bag | "small sealed sack of compressed wood pellets, rectangular, dense" |
| Stump | "tree stump, circular cross section, flat top, roots at base" |
| Sapling | "small young tree seedling, 30cm tall, few leaves, thin trunk" |
| Kei truck | "small Japanese kei truck, compact pickup, flatbed, boxy shape" |

## Why first-try accuracy matters

Each Tripo generation costs API credits + 2-5 minutes. Wrong vocab:
- Wrong model on first attempt
- Delete + re-prompt: +2-5 minutes
- Possible 2nd wrong attempt: +2-5 minutes again

Correct vocab = ~80% first-try success rate. Table pays for itself in first 5 uses.

## General rule

Tripo trained on common usage. When your concept is niche, abstract, or industry-specific: don't use the common name. Describe the VISUAL SHAPE first, then add context. "What does it look like?" > "What is it called?"

See also: [[tripo-cleanup-pipeline]], [[tripo-asymmetric-floating-retopo]], [[iterative-checkpoint-workflow-generated-assets]]
