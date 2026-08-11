---
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: pipeline/lessons
tags: [blender, fbx, eksport, bramka, skala, unity]
date: 2026-08-02
status: draft
---

# Bramka eksportu ma mierzyć gotowy plik, nie ufać policzonej skali

## Sytuacja

Podmiana czterech modeli świerku w grze. Modele muszą mieć dokładnie te same
wysokości co poprzednie, inaczej drzewa zmieniłyby wielkość na mapie. Skala
liczona była w kodzie: `skala = wysokość_docelowa / wysokość_naturalna`.

## Co poszło źle

Wysokość naturalna policzyła się z NIEPEŁNYCH ustawień generatora - słownik
zbudowano z tabeli suwaków, pomijając listy wyboru (rodzaj i kształt korony).
Model zbudował się z domyślnego rodzaju korony i dał 5,288 m zamiast 5,365 m.
Wszystkie cztery pliki wyjechały o 1,5 procenta za wysokie (16-80 mm).

Sam eksport zgłosił sukces. Log mówił "OK, zapisano, 2102 trójkąty". Nic
w komunikacie nie wskazywało błędu, bo kod wykonał dokładnie to, o co go
poproszono - tylko wzorzec był zły.

## Lekcja

**Wartość policzona w kodzie jest obietnicą, nie dowodem.** Bramka, która
sprawdza zmienną tuż po jej wyliczeniu, mierzy własną definicję i przepuści
każdy błąd popełniony przed nią.

Bramka eksportu ma:
1. wczytać gotowy plik Z DYSKU,
2. zmierzyć go od nowa niezależną drogą,
3. porównać z celem pochodzącym z POMIARU, nie z notatki,
4. mieć próg wyrażony w jednostkach świata (tu 1 cm), a nie w procentach.

## Dowód, że bramka umie oblać

Bramka dostała tryb `--tylko-kontrola` i została puszczona na plikach z błędną
skalą, ZANIM je poprawiono. Dała ZLE na wszystkich czterech, z rozmiarem błędu
w milimetrach. Dopiero potem, na poprawionych plikach, dała zero milimetrów
różnicy. Bramka bez pokazanego trybu porażki nie jest bramką.

## Przenośne poza ten projekt

Dotyczy każdego pipeline'u generator → plik → silnik: eksport siatek, atlasów,
danych konfiguracyjnych. Wszędzie tam, gdzie skrypt liczy przelicznik
i natychmiast go stosuje, warto zapytać: co czyta bramka - moją zmienną czy
plik, który naprawdę powstał?

Powiązane: [[bramka-nie-moze-mierzyc-wlasnej-definicji]], [[sonda-musi-umiec-zawiesc]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260811-1520-cloudrig-daje-czysty-rig-do-gry-bo-odwraca-domyslne-use-deform|CloudRig daje czysty rig do gry, bo odwraca domyslne use_deform Blendera]] - wspolne: fbx, blender
- [[20260531-0934-tripo-polygon-soup-inverted-winding-fix|Tripo / AI-generated meshes import as "polygon soup" - see-through holes under single-sided rendering are a winding problem caused by UNWELDED verts, not interior faces]] - wspolne: fbx, blender
- [[20260626-1203-fbx-pivot-direction-vs-procedural-placement|Pivot/geometry direction of an FBX must match what a procedural placement tool assumes]] - wspolne: fbx, blender
- [[20260629-1145-blender-empties-bake-space-transform-double-axis|FBX with parent EMPTIES imports tipped 90° when exported with bake_space_transform=True]] - wspolne: fbx, blender
- [[20260704-2330-blender-unity-flat-panel-dual-face-texture|Blender flat panel textured on one face renders BLANK in Unity (axis-flip picks the wrong face)]] - wspolne: fbx, blender
- [[20260713-1030-verify-in-target-engine-not-source-tool|Weryfikuj asset w silniku DOCELOWYM, nie w narzędziu źródłowym]] - wspolne: fbx, blender
<!-- /POWIAZANE:auto -->
