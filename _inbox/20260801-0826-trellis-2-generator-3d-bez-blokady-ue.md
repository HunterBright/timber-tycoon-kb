---
title: TRELLIS.2 jako generator 3D bez blokady licencyjnej w UE
type: pattern
status: draft
confidence: medium
verified: 2026-08-01
tags: [low-poly, generatory-3d, licencje, ue, blender, pipeline]
date: 2026-08-01
project: GameDevOS
source: https://github.com/microsoft/TRELLIS.2/blob/main/LICENSE
applies_to: [generowanie-modeli, low-poly, potwory, pipeline-blender-unity]
---

# TRELLIS.2 jako generator 3D bez blokady licencyjnej w UE

## Kiedy stosować

Gdy potrzebujesz **lokalnego** generatora modeli 3D do zasobów komercyjnej gry
wydawanej w Unii Europejskiej, a dotychczasowy kandydat odpadł na licencji.

To jest wpis o **wyborze bramkowanym licencją, nie jakością**. Punkt wyjścia:
w rodzinie mocnych generatorów obraz-na-3D większość albo działa wyłącznie
w chmurze na subskrypcji, albo ma licencję społecznościową z wykluczeniem
terytorialnym. TRELLIS.2 jest dziś jedynym znanym nam modelem tej klasy,
który przechodzi obie bramki naraz: działa lokalnie i ma czyste MIT.

## Kroki

1. **Zweryfikuj licencję u źródła**, zanim cokolwiek pobierzesz - zgodnie
   z [[20260801-0825-licencja-marki-nie-jest-licencja-produktu]].
   Sprawdzone 2026-08-01: `microsoft/TRELLIS.2/LICENSE` to czyste MIT,
   bez klauzul o zakresie użycia i bez ograniczeń terytorialnych.
2. Pobierz wagi lokalnie (4 mld parametrów,
   [microsoft/TRELLIS.2-4B](https://huggingface.co/microsoft/TRELLIS.2-4B)).
3. Wygeneruj sylwetkę i wyeksportuj do GLB lub OBJ.
4. **Zredukuj siatkę w Blenderze wbudowanym QuadriFlow**, ze skryptu,
   bezokienkowo - bez sięgania po płatne narzędzia do retopologii.
5. Dopiero na zredukowanej siatce zakładaj szkielet.

## Dlaczego to działa

MIT nie zawiera ani progów przychodowych, ani listy dozwolonych terytoriów,
ani klauzuli o dopuszczalnym zastosowaniu. Jedyny obowiązek to dołączenie
noty o prawach autorskich. Znika więc cała klasa pytań, która blokuje
generatory na licencjach społecznościowych.

Praca lokalna dokłada drugą korzyść: zasoby własnej gry nie wychodzą
na serwer dostawcy. Przy usługach chmurowych to jest osobna decyzja,
nie tylko techniczna.

## Koszty i kompromisy

- **Jakość siatki jest deklaracją, nie faktem.** Opisy "topologia w większości
  czworokątna, około 50 tys. trójkątów, gotowe pod rigowanie" pochodzą ze stron
  promujących narzędzie. Niezależnego testu brak.
- 50 tys. trójkątów to **nie jest low poly**. Krok redukcji jest obowiązkowy,
  nie opcjonalny.
- Model wydany w grudniu 2025 - dojrzały, ale nie najnowszy. Warto sprawdzać,
  czy nie pojawił się następca na równie czystej licencji.
- Wymaga własnego GPU i czasu na postawienie środowiska. Chmurowa
  konkurencja jest szybsza w starcie i droższa w utrzymaniu.

## Warianty

- **Redukcja:** QuadriFlow w Blenderze (darmowy, w standardzie, skryptowalny)
  zamiast Tripo AI czy 3D AI Studio - te są w chmurze i na subskrypcję.
- **Odrzucone:** Hunyuan3D - licencja wyklucza UE, stan potwierdzony 2026-08-01.
  Nie mylić ze zmianą licencji modelu **językowego** Hy3.

## Dowód, że zadziałało u nas

**Brak. To jest hipoteza do sprawdzenia, nie sprawdzony wzorzec.**
Wpis powstał z przeglądu rynku, nie z testu. Zweryfikowana jest wyłącznie
licencja - i to bezpośrednio w pliku źródłowym.

Test rozstrzygający, jeden wieczór: wygenerować jedną sylwetkę nieludzką,
zredukować QuadriFlow, spróbować założyć szkielet. Wynik zamienia ten wpis
w sprawdzony wzorzec albo skreśla go w całości. Do tego czasu
`confidence: medium` jest uczciwe.

## Powiązane
- [[20260801-0825-licencja-marki-nie-jest-licencja-produktu]]
- [[MAPA-LOW-POLY]]
- [[MAPA-PIPELINE-BLENDER-UNITY]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260801-0825-licencja-marki-nie-jest-licencja-produktu|Licencja marki nie jest licencja produktu]] - wspolne: ue, generatory-3d, licencje
- [[20260725-0625-ai-model-community-license-excludes-eu|Nie stawiaj pipeline'u assetow na modelu AI z licencja "Community", nie czytajac pierwszej linii LICENSE]] - wspolne: licencje, pipeline
- [[20260725-1830-plaskie-tekstury-z-plam-referencji|Plaskie tekstury dla modelu z generatora 3D: tozsamosc elementu bierz z REFERENCJI, nie z koloru]] - wspolne: generatory-3d, blender
- [[20260719-1605-mesh-seethrough-audit-pattern|Wzorzec audytu prześwitów w siatkach: render 3-przebiegowy > heurystyki geometryczne]] - wspolne: pipeline, blender
- [[character-pipeline-tripo-mixamo-unity|Character pipeline: Tripo mesh → Mixamo rig → Unity (clean, working recipe)]] - wspolne: pipeline, blender
- [[blender-mcp-interactive-remodel-loop|Blender-MCP Interactive Remodel Loop (GUID-Preserving In-Place Replace)]] - wspolne: pipeline, blender
<!-- /POWIAZANE:auto -->
