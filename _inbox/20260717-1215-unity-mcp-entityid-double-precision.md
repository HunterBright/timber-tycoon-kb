---
type: lesson
project: Timber_Tycoon
suggested-category: engine/lessons
tags: [unity-mcp, entityid, json, float64, ai-generators, hunyuan, precision, mcp]
date: 2026-07-17
status: draft
---

# Unity 6.5 EntityId (64-bit) gubi precyzję w kanale MCP/JSON - generator dostaje CUDZY asset

## Symptom
Generacja 3D (image-to-3D, Hunyuan przez unity-mcp `GenerateAsset`) zwraca model
NIEZWIĄZANY z obrazkiem referencyjnym, za to podejrzanie podobny do INNYCH assetów
projektu (w grze o tartaku: kupka kłód). Powtarzalnie, nie losowo. Żadnego błędu -
narzędzie raportuje sukces.

## Korzeń
1. Unity 6000.5 używa 64-bitowych EntityId (GetInstanceID jest deprecated); wartości
   sesyjne rzędu 5.7e17.
2. Odpowiedzi/parametry MCP przechodzą przez JSON z konwersją przez float64 (double).
   W zakresie ~5.7e17 odstęp między reprezentowalnymi liczbami (ulp) wynosi 64.
   Identyfikator "568105584918909716" wraca/dochodzi jako "...700".
3. Serwer (GenerateAssetTool w com.unity.ai.assistant) robi EntityIdToObject na
   zaokrąglonej liczbie. Jeśli trafi w ISTNIEJĄCY obiekt (w dużym projekcie łatwo,
   bo identyfikatory sesyjne są gęste i rosną sekwencyjnie), generacja dostaje
   ten obiekt jako referencję - bez żadnego ostrzeżenia.

## Wykrywanie (smoking gun)
Dwa RÓŻNE assety zwracają w odpowiedziach narzędzia IDENTYCZNE FileInstanceID
(np. dwa wygenerowane obrazki oba "...800") - kolizja niemożliwa przy prawdziwych
identyfikatorach, pewny dowód zaokrąglania w kanale.

## Obejście (sprawdzone)
1. Zrób N kopii pliku referencyjnego NA DYSKU (nie przez AssetDatabase - narzędzia
   modyfikujące assety przez MCP mogą być zablokowane "user interaction required"),
   niech Unity je zaimportuje.
2. Skryptem w edytorze (np. unity-mcp `Unity_RunCommand`, tylko odczyt) znajdź kopię,
   której EntityId przeżywa round-trip: `(ulong)(double)id == id`. W zakresie 5.7e17
   trafia się ~1 na kilkadziesiąt kopii (ulp=64, przyrosty id 4-100).
3. Podaj TEN identyfikator w wywołaniu MCP. Zaokrąglenie w kanale (w obie strony)
   nie zmienia wartości, bo każda liczba w promieniu +/-32 parsuje się do tego samego
   double. Zapis dziesiętny w odpowiedzi może wyglądać "inaczej" (shortest repr,
   np. ...592 wyświetla się jako ...600) - to TEN SAM double, nie błąd.
4. Preferuj parametry ŚCIEŻKOWE tam, gdzie istnieją (`targetAssetPath` - np.
   RemoveImageBackground) - ścieżki nie mają tego problemu.

## Uwagi powiązane
- Wymóg Hunyuan w unity-mcp: obrazek referencyjny MUSI mieć przezroczyste tło
  (`RemoveImageBackground` z `photoroom-bg-removal` robi to w miejscu, po ścieżce).
- Diagnoza wymagała czytania źródeł pakietu w Library/PackageCache
  (GenerateAssetTool.cs) - ścieżka pojedynczej referencji przy nieistniejącym id
  rzuca błąd, więc "sukces" oznacza, że id TRAFIŁO w jakiś obiekt.
