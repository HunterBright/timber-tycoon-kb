---
title: 'Headless TMP setup: import Essentials + generate SDF font asset from editor script'
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-06-12'
project: Timber_Tycoon
tags:
- unity
- textmeshpro
- tmp
- font
- sdf
- editor-script
- headless
- automation
- localization
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Headless TMP setup: import Essentials + generate SDF font asset from editor script

Fully scripted TextMeshPro setup with no Font Asset Creator window - works via MCP/CLI editor-script execution.

## 1. Import TMP Essential Resources non-interactively

In Unity 6, TMP is merged into `com.unity.ugui` 2.0 - the `TMPro` namespace compiles WITHOUT essentials, but runtime TMP text needs `TMP Settings.asset` + SDF shaders, which ship as a .unitypackage inside the package:

```csharp
var info = UnityEditor.PackageManager.PackageInfo.FindForAssetPath("Packages/com.unity.ugui/package.json");
string pkg = Path.Combine(info.resolvedPath, "Package Resources", "TMP Essential Resources.unitypackage");
AssetDatabase.ImportPackage(pkg, false); // non-interactive
```

`PackageInfo.FindForAssetPath` resolves the physical PackageCache path (hash suffix changes per version - never hardcode). Verify by checking `Assets/TextMesh Pro/Resources` exists before dependent steps (import completes after the call returns).

## 2. Generate a static SDF font asset in code

```csharp
var fa = TMP_FontAsset.CreateFontAsset(ttf, 90, 9, GlyphRenderMode.SDFAA, 2048, 2048,
    AtlasPopulationMode.Dynamic, enableMultiAtlasSupport: false);
fa.TryAddCharacters(charset, out string missing);   // report 'missing'!
fa.atlasPopulationMode = AtlasPopulationMode.Static; // freeze atlas into asset
AssetDatabase.CreateAsset(fa, path);
AssetDatabase.AddObjectToAsset(fa.material, fa);     // material + atlas are sub-assets
AssetDatabase.AddObjectToAsset(fa.atlasTexture, fa); // or the asset breaks on reload
AssetDatabase.SaveAssets();
```

Key points:
- Create as **Dynamic**, add chars, then flip to **Static** - direct static creation has no clean char-add API.
- `multiAtlasSupport: false` makes overflow loud: `TryAddCharacters` returns the characters that didn't fit instead of silently spilling to a second texture.
- **Always log `missing`** - it also catches glyphs absent from the TTF itself (Nunito lacks ĳ and ſ).
- Charset for EU localization: ASCII 0x20-0x7E + Latin-1 0xA0-0xFF + Latin Extended-A 0x100-0x17F (covers Polish ą ę ó ś ź ż ć ń ł) + typographic punctuation - - ' ' „ " … • € **and U+2212 (true minus)** - UI code that prints "−" renders tofu without it. ~330 glyphs fit comfortably in 2048² @ 90pt/9pad.
- Don't touch global `TMP Settings` default font - assign the font asset explicitly (via a skin/style SO) to avoid affecting unrelated systems.

Bonus pattern for code-built UI: a skin ScriptableObject lives in a **Resources** folder (loadable with zero scene references), with all sprite fields optional - null sprite → flat color fallback identical to pre-reskin look, so migration and reskin are decoupled.

Validated in Timber_Tycoon 2026-06-12 (Unity 6000.3, ugui 2.0.0): 329 glyphs, atlas 2048×2048, zero manual steps.
