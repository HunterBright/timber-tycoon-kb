---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [unity, static-constructor, resources-load, type-initializer, ui-bootstrap, startup-crash, lifecycle]
severity: high
time_lost: "~25 min (mylące: błąd w klasie, której nie zmienialiśmy)"
date: 2026-06-23
status: draft
applies_to: [unity-runtime-init]
---

# Resources.Load w inicjalizatorze pola statycznego psuje cały start UI (TypeInitializationException zapadkowana)

## Problem
Po niezwiązanych zmianach UI gra startowała bez HUD i menu (HudUI/SettingsUI/PauseMenuUI
nie powstawały), choć inne systemy (lokalizacja, świat) działały. W logu:
`UnityException: Load is not allowed to be called from a MonoBehaviour constructor (or
instance field initializer)` — w klasie `QuestUI`, której NIE dotykaliśmy. Centralny
bootstrap UI tworzy podsystemy po kolei; wyjątek przy tworzeniu QuestUI przerywał `Awake`
bootstrapu, więc wszystko PO QuestUI nie powstawało (ta sama kaskada co przy każdym wyjątku
w pętli inicjalizacyjnej).

## Root cause
```csharp
// QuestUI
private static readonly Color ColorDone = ResolveProgressFill();   // inicjalizator pola statycznego
private static Color ResolveProgressFill() {
    UISkinSO skin = UISkinSO.Default;            // -> Resources.Load<UISkinSO>("UISkin_Default")
    return skin != null ? skin.progressFill : fallback;
}
```
Statyczny konstruktor klasy (cctor) odpala się przy PIERWSZYM dotknięciu typu. Jeśli to
pierwsze dotknięcie wypada podczas KONSTRUKCJI MonoBehaviour (np. `AddComponent<QuestUI>()`
albo deserializacja sceny), to `Resources.Load` w cctorze jest ZAKAZANY → cctor rzuca. Co
gorsza, .NET ZAPADKUJE (cache'uje) wyjątek inicjalizatora typu: po jednym rzuceniu KAŻDY
kolejny dostęp do typu rzuca `TypeInitializationException` aż do przeładowania domeny
(rekompilacja). Dlatego błąd „przykleja się" i jest mylący (czasem działa, czasem nie —
zależnie od tego, w jakim kontekście typ pierwszy raz się zainicjalizował).

## Solution
Nie wołaj `Resources.Load` (ani niczego ciężkiego z Unity API) w inicjalizatorze pola
statycznego. Zrób to LENIWE — rozwiąż przy pierwszym UŻYCIU (które wypada już w Awake/build,
gdzie Resources.Load jest dozwolony):
```csharp
private static Color? _colorDoneCache;
private static Color ColorDone {
    get {
        if (_colorDoneCache == null) _colorDoneCache = ResolveProgressFill();
        return _colorDoneCache.Value;
    }
}
```
Po naprawie wymagane jest przeładowanie domeny (rekompilacja), żeby wyczyścić zapadkowany
wyjątek — sama zmiana skryptu to robi.

## What didn't work
- Restart Play Mode bez rekompilacji — jeśli „Reload Domain" jest wyłączony, statyczny stan
  (w tym zapadkowany wyjątek) PRZETRWA między sesjami Play; błąd wraca.
- Szukanie winy w ostatnio zmienianym kodzie — wyjątek był w nietkniętej klasie; trzeba było
  przeczytać log/stack (wskazał `..cctor` + `Resources.Load`).

## Transferability
Uniwersalny anti-pattern Unity: ŻADNE `Resources.Load`, `FindObjectOfType`, instancjonowanie
itp. w inicjalizatorach pól statycznych ani w konstruktorach MonoBehaviour. Rób w Awake/Start
albo leniwie. Dotyczy każdego projektu Unity. Bonus-lekcja: pojedynczy wyjątek w pętli
„twórz-podsystemy" bootstrapu kładzie WSZYSTKO po nim — warto owijać pojedyncze tworzenia
w try/catch, by jeden zły podsystem nie wyłączał całego UI.

## Related
- [[20260623-0855-unity-layoutelement-requirecomponent-recttransform-null-trap]] (ta sama klasa: wyjątek w bootstrapie UI kładzie resztę)
- [[20260623-0840-unity-cjk-cyrillic-fonts-tmp-and-legacy-text]] (ta sama funkcja lokalizacji)
