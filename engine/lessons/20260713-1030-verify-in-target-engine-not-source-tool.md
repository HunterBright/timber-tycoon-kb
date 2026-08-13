---
title: Weryfikuj asset w silniku DOCELOWYM, nie w narzędziu źródłowym
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-13'
project: Kerf - Sawmill Tycoon
tags:
- blender
- unity
- fbx
- export
- orientation
- verification
- asset-pipeline
applies_to: []
source: ''
severity: high
promoted: '2026-07-30'
---

# Weryfikuj asset w silniku DOCELOWYM, nie w narzędziu źródłowym

## Objaw

Model (plandeka na pakę auta) został w Blenderze zbudowany, dopasowany i **programowo zaudytowany**:
zero kolizji z autem, zero dziur w pokryciu, prześwit od kabiny 8 mm. Po eksporcie FBX agent
zrobił jeszcze **round-trip**: wczytał plik z powrotem do Blendera i porównał bounding box,
skalę, pivot, liczbę wierzchołków. Wynik: **"odchylenie 0.000 na każdej osi, żadna korekta
w Unity nie jest potrzebna"**.

Po wpięciu do Unity model wylądował **obrócony o 180 stopni**: jego płaska krawędź czołowa
(zaprojektowana tak, by minąć słupki kabiny o 8 mm) trafiła na tył, a tylny okap **wjechał
w kabinę na 9 cm**.

## Dlaczego weryfikacja round-trip tego nie złapała

Bo sprawdzała **Blender → FBX → Blender**, a błąd powstaje na **FBX → Unity**.

Konwersja układu współrzędnych (`axis_forward='-Z'`, `axis_up='Y'`, `bake_space_transform=True`)
jest **odwracalna wewnątrz Blendera** - Blender wczytując swój własny plik kompensuje to, co
sam przy zapisie wpiekł. Round-trip zawsze wyjdzie idealnie, nawet gdy plik dla silnika jest
obrócony. Blender potwierdza sam siebie.

## Jak to złapać

**Zmierz asset po stronie silnika, na osi ASYMETRYCZNEJ.**

Ten model był symetryczny w poprzek (±0,815) i asymetryczny wzdłuż (-0,781 / +0,883). Gdyby
patrzeć tylko na rozmiar bounding boxa (1,66 × 1,63) albo na oś symetryczną - obrót o 180
stopni byłby **niewykrywalny**. Widać go dopiero, gdy porówna się **znak asymetrii**:

```
Blender (oczekiwane):  X od -0.781 do +0.883   (więcej materiału z tyłu)
Unity   (zastane):     X od -0.883 do +0.781   (więcej materiału z PRZODU!)
```

Reguła: po imporcie assetu policz jego pozycję w silniku i porównaj z **projektowanym punktem
odniesienia** (tu: krawędź kabiny na X = 0,070 kontra krawędź modelu). Zgodność rozmiaru NIE
jest zgodnością orientacji.

## Reguła

1. **Round-trip w narzędziu źródłowym jest niewystarczający.** Sprawdza serializację, nie
   konwersję do silnika.
2. **Miarodajna weryfikacja to pomiar w silniku docelowym**, na osi, na której model jest
   asymetryczny, względem konkretnego punktu odniesienia sceny.
3. Gdy silnik ma udokumentowany quirk orientacji (tu: przód auta to `-transform.right`, bo
   FBX z Blendera przychodzi obrócony) - **każdy nowy asset dokładany do tego obiektu odziedziczy
   ten sam obrót**. Zakładaj obrót i udowodnij, że go nie ma, a nie odwrotnie.

## Sygnał ostrzegawczy do zapamiętania

Gdy narzędzie/agent mówi **"zweryfikowałem, odchylenie zero, żadna korekta nie jest potrzebna"**,
zapytaj: *zweryfikował względem czego?* Jeśli względem samego siebie - to nie jest weryfikacja,
tylko potwierdzenie własnej serializacji.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[bake-space-transform-linked-duplicates-rotation-bug|bake_space_transform + Linked Duplicates = 90° Rotation Injection]] - wspolne: export, fbx, blender
- [[fbx-export-standard-settings-blender-to-unity|FBX Export Standard Settings (Blender → Unity)]] - wspolne: export, fbx, blender
- [[20260629-1145-blender-empties-bake-space-transform-double-axis|FBX with parent EMPTIES imports tipped 90° when exported with bake_space_transform=True]] - wspolne: orientation, fbx, blender
- [[fbx-long-axis-detect-programmatically|Don't assume an FBX mesh's axis - detect the longest axis programmatically from bounds]] - wspolne: orientation, fbx, blender
- [[20260531-0934-fbx-mesh-only-verification-scan-class-names|Verifying an FBX is "mesh-only" before Mixamo: scan for the real CLASS names, not substrings - `AnimStack` matches the header property `ActiveAnimStackName`]] - wspolne: verification, fbx
- [[20260626-1110-unity-65-material-location-migration-and-runcommand-guard|Unity 6.5: bezpieczna migracja `MaterialLocation.External` + guard w AI-Assistant Run Command]] - wspolne: asset-pipeline, fbx
<!-- /POWIAZANE:auto -->
