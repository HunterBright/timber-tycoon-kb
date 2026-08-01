---
title: uGUI toast/plaque with a fixed LayoutElement.preferredHeight clips multi-line and localized text
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-07'
project: Kerf - Sawmill Tycoon
tags:
- unity
- ugui
- layoutelement
- contentsizefitter
- tmp
- localization
- auto-height
applies_to:
- unity
source: ''
severity: low
time_lost: ''
promoted: '2026-07-30'
---

# uGUI toast/plaque with a fixed LayoutElement.preferredHeight clips multi-line and localized text

## Problem
A bottom-left notification/toast stack (each entry = one plaque) showed only one line tall. When a message needed two lines (e.g. "quality" + "+2 planks +1 bark", or a longer translation), the extra line was clipped instead of growing the plaque. The container (VerticalLayoutGroup + ContentSizeFitter) auto-fit the stack correctly, but every entry still reported a single-line height.

## Root cause
Each entry's root had `LayoutElement.preferredHeight = 46` (a hardcoded single-line value) with `flexibleHeight = 0`. That pins the entry height regardless of text content, so the parent VerticalLayoutGroup lays out every entry at 46px. Separately, the entry's inner HorizontalLayoutGroup had `childControlWidth = false`, so the TMP text was never width-constrained - with no wrap width, `TextWrappingModes.Normal` produced a single overflowing line and reported a single-line preferred height.

## Solution
Let the text drive height, with a floor instead of a fixed value:
- On the entry root LayoutElement: use `minHeight = 46` (single-line floor) and DO NOT set `preferredHeight` (leave it -1). The layout then takes the HorizontalLayoutGroup's computed preferred height, which comes from the TMP text.
- On the entry's HorizontalLayoutGroup: set `childControlWidth = true` so the text rect is constrained to (container width − padding). Only then does TMP wrap and report the correct multi-line preferred height.
- Keep the parent VerticalLayoutGroup(childControlHeight) + ContentSizeFitter(verticalFit = PreferredSize) to grow the stack.
Result: single-line messages stay ~46px; two-line and long-translation messages grow automatically. Explicit `\n` in the message also works.

## What didn't work
- Putting a ContentSizeFitter directly on the child while the parent VerticalLayoutGroup has childControlHeight=true fights the parent - prefer driving the child preferred height via minHeight + the child's own layout group instead.
- Leaving childControlWidth=false and hoping wrap kicks in: TMP needs a constrained rect width first, or 2-line text still measures as 1 line.

## Transferability
Any localized uGUI game/app: fixed pixel heights on text-bearing layout children are a recurring localization bug (German/Polish/CJK run longer). The rule "minHeight as a floor, never a fixed preferredHeight on text elements; constrain width so the text can report its true wrapped height" applies to toasts, tooltips, list rows, dialog bodies, quest cards.

## Related
- [[20260616-1532-unity-9slice-contentsizefitter-pollution]]
- [[20260623-0855-unity-layoutelement-requirecomponent-recttransform-null-trap]]
- [[20260613-0625-9slice-ppu-must-scale-to-target-rect-not-stay-100]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260616-1532-unity-9slice-contentsizefitter-pollution|9-slice Image na obiekcie layoutu zaniża ContentSizeFitter (panel rośnie do sumy borderów)]] - wspolne: contentsizefitter, ugui
- [[20260612-1815-headless-tmp-sdf-font-generation|Headless TMP setup: import Essentials + generate SDF font asset from editor script]] - wspolne: tmp, localization
<!-- /POWIAZANE:auto -->
