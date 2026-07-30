---
type: anti-pattern
project: Timber Tycoon
suggested-category: engine/anti-patterns
tags: [unity, particles, renderer, bounds, getcomponentsinchildren, regression]
date: 2026-07-18
status: draft
---

# GetComponentsInChildren<Renderer> + Encapsulate(bounds) psuje sie po dodaniu czasteczek

## Anty-wzorzec
Liczenie obrysu obiektu ("gora maszyny", "srodek", wysokosc pierscienia UI) petla po
`GetComponentsInChildren<Renderer>()` z `bounds.Encapsulate`. Dziala latami - do dnia,
w ktorym ktos doda dziecko z ParticleSystem: **ParticleSystemRenderer TEZ jest Rendererem**,
jego bounds zyja i puchna z czasteczkami, a `.enabled` jest true nawet gdy emiter nie gra.

## Objaw z praktyki
Dodanie emitera kurzu pod maszyne przesunelo "gore maszyny" -> pasek HUD odplynal od
przyciskow, a animowany pniak przestal wpadac do wlotu (cel wskoku liczony z obrysu).
Druga instancja tej samej klasy bledu: pierscien postepu NPC nad maszynami.

## Regula
W KAZDEJ petli zbierajacej `Renderer`y do obrysu/bounds dodawaj:
```csharp
if (r is ParticleSystemRenderer) continue;
```
(analogicznie warto rozwazyc TrailRenderer/LineRenderer, gdy licza sie tylko "twarde" meshe).
Przy dodawaniu emiterow czasteczek do istniejacych obiektow: grep projektu po
`GetComponentsInChildren<Renderer>` i przejrzyj kazde trafienie POD KATEM obrysow.
