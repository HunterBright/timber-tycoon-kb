---
title: AddComponent<RectTransform>() zwraca null po wcześniejszym AddComponent komponentu z [RequireComponent(RectTransform)]
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-06-23'
project: Kerf - Sawmill Tycoon
tags:
- unity
- ui
- recttransform
- layoutelement
- requirecomponent
- addcomponent
- None
- awake
- startup-softlock
applies_to:
- unity-ui-runtime-construction
source: ''
severity: high
time_lost: ~20 min (zlokalizowane szybko przez log Unity)
promoted: '2026-07-30'
---

# AddComponent<RectTransform>() zwraca null po wcześniejszym AddComponent komponentu z [RequireComponent(RectTransform)]

## Problem
Runtime'owy kod budujący UI rzucał `NullReferenceException` w linii ustawiającej
`rectTransform.anchorMin`. Objaw w grze był nieproporcjonalnie duży: panel ustawień
budowany w `Awake` wywalał się w środku → `overlay.SetActive(false)` na końcu `Awake`
nie wykonywał się (okno zostawało otwarte na starcie), a wyjątek propagował przez
`AddComponent<SettingsUI>()` do `UIBootstrap.Awake` i przerywał tworzenie WSZYSTKICH
kolejnych podsystemów UI → gra niesterowalna na starcie ("startup soft-lock").

## Root cause
```csharp
GameObject go = new GameObject("X");              // ma zwykły Transform
go.AddComponent<LayoutElement>();                 // LayoutElement: [RequireComponent(typeof(RectTransform))]
                                                  //   → Unity SAM dodaje RectTransform tutaj
RectTransform rt = go.AddComponent<RectTransform>(); // null! GameObject ma już RectTransform,
                                                  //   drugi komponent transformu jest niedozwolony
rt.anchorMin = ...                                // NRE
```
Komponent z atrybutem `[RequireComponent(typeof(RectTransform))]` (LayoutElement,
GridLayoutGroup, większość komponentów uGUI) przy dodaniu automatycznie tworzy
RectTransform. Późniejsze `AddComponent<RectTransform>()` NIE zwraca istniejącego -
zwraca `null` (GameObject może mieć tylko jeden komponent typu Transform).

## Solution
Utwórz GameObject od razu z RectTransform i pobieraj go przez GetComponent, a komponenty
uGUI dodawaj POTEM:
```csharp
GameObject go = new GameObject("X", typeof(RectTransform));
RectTransform rt = go.GetComponent<RectTransform>();   // pewne, nie-null
go.AddComponent<LayoutElement>();                       // RectTransform już jest → brak konfliktu
```
Dodatkowo (obrona przed klasą błędów): budowę UI w `Awake` owinąć w
`try { Build(); } catch (log) finally { overlay.SetActive(false); }` - pojedynczy błąd
budowy panelu nie może wtedy zostawić okna otwartego ani przerwać bootstrapu reszty UI.

## What didn't work
- Zakładanie, że `AddComponent<RectTransform>()` zwróci istniejący RectTransform (jak
  `GetComponent`) - NIE, zwraca null gdy transform już istnieje.
- Szukanie przyczyny w ostatnio zmienionym kodzie - wyjątek był w innej, starszej linii;
  log Unity (stack trace) wskazał plik+linię od razu (zawsze najpierw czytaj log/stack).

## Transferability
Dotyczy KAŻDEGO projektu Unity budującego uGUI z kodu (nie z prefabów). Pułapka jest
uniwersalna dla wszystkich komponentów z `[RequireComponent(typeof(RectTransform))]`.
Druga lekcja (try/finally wokół budowy UI w Awake, by uniknąć startup soft-locka) jest
ogólnym wzorcem odporności bootstrapu UI.

## Related
- [[20260623-0840-unity-cjk-cyrillic-fonts-tmp-and-legacy-text]] (ta sama funkcja lokalizacji)
