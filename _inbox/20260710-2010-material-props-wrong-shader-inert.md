---
type: anti-pattern
project: Timber Tycoon
suggested-category: engine/anti-patterns
tags: [unity, materials, shader, urp, editor-scripts]
date: 2026-07-10
status: draft
---

# Ustawianie właściwości materiału bez sprawdzenia SHADERA = ciche nic + ryzyko nadpisania

## Co poszło nie tak
Zadanie: "dodaj maszynom delikatny połysk". Skrypt edytorowy ustawił `_Smoothness`,
`_Metallic`, `_SpecularHighlights` + keywordy URP/Lit na materiałach maszyn.
Efekt: ZERO połysku - materiały używały projektowego `Custom/FlatColorLit`
(czyta tylko `_BaseColor` i `_Brightness`); wszystkie zapisy były martwe
(`SetFloat` na niezadeklarowanej właściwości nie rzuca błędem, keyword ląduje
w `m_InvalidKeywords`). GORZEJ: ten sam skrypt pisał też `_BaseColor` ze starej
tablicy wartości - jedyny zapis, który ZADZIAŁAŁ, i nadpisał zaakceptowaną paletę.

## Zasady
1. PRZED skryptową edycją materiału sprawdź `mat.shader.name` (i zweryfikuj GUID
   shadera w YAML .mat - GUID może należeć do customowego shadera, nie do URP/Lit).
2. W skryptach masowej edycji materiałów dodaj guard: `if (mat.shader.name != oczekiwany) skip`.
3. Skrypt "zmień cechę X" NIE powinien przepisywać innych właściwości (kolorów!)
   ze swojej tablicy "źródła prawdy" - stare tablice wartości potrafią przeżyć
   rework wyglądu i cicho cofnąć akceptowaną paletę.
4. Jednorazowe skrypty look-ów po reworku wyglądu blokuj guardem OBSOLETE -
   przypadkowy re-run z menu nie może nadpisać aktualnego akceptu.
5. Sygnał ostrzegawczy w .mat: właściwość/keyword w `m_InvalidKeywords` lub wartości,
   które "nic nie zmieniają" = piszesz do niewłaściwego shadera.

## Jak wykryto
Adwersaryjny review diffu (multi-agent) porównał GUID shadera z plikiem .shader
i wartości LFS blobów przed/po - ręczny playtest mógł tego nie zauważyć od razu.
