---
type: lesson
project: Timber_Tycoon
suggested-category: engine/lessons
tags: [unity, shader, material, serialization, urp, backwards-compatibility]
date: 2026-07-02
status: draft
---

# Dodanie property do shadera może aktywować STARE, ukryte wartości w materiałach

## Kontekst
Rozszerzanie custom shadera (VertexColorLit) o tint koloru dla nowych koron drzew.
Plan: dodać `_BaseColor` z defaultem białym = "backwards compatible, stare materiały
bez zmian".

## Lekcja
Unity NIE usuwa z pliku .mat wartości properties, których aktualny shader nie ma.
Jeśli materiał kiedyś używał innego shadera (np. URP/Lit), jego stary `_BaseColor`
siedzi zapisany w YAML "na zapas". Gdy shader ZYSKA property o tej samej nazwie,
stara wartość natychmiast się aktywuje — default białego NIE chroni takich materiałów.

Case: `Mat_Debris_Dirt.mat` (VFX kopania pniaka) miał zapisany brązowy `_BaseColor`
z poprzedniego shadera. Dodanie `_BaseColor` do VertexColorLit przyciemniłoby efekt
kopania (vertex color × brąz), mimo defaultu (1,1,1,1).

## Reguła praktyczna
Przed dodaniem property do współdzielonego shadera:
1. `grep` po WSZYSTKICH .mat używających tego shadera (po GUID shadera) w poszukiwaniu
   nazwy property, którą chcesz dodać.
2. Jeśli JAKIKOLWIEK materiał ma ją zapisaną z nie-defaultową wartością → nazwij
   property unikalnie (np. `_LeafTint` zamiast `_BaseColor`). Unikalna nazwa =
   gwarancja, że default zadziała wszędzie.
3. Weryfikacja jest tania: `grep -l <guid-shadera> *.mat`, potem `grep <property>` w wynikach.
