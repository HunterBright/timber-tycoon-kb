---
title: Lazy-create singleton wołany z OnDestroy tworzy obiekty przy zamykaniu sceny
type: anti-pattern
status: draft
confidence: low
verified: ''
date: '2026-07-02'
project: Timber_Tycoon
tags:
- unity
- singleton
- lazy-init
- ondestroy
- teardown
- ui
applies_to: []
source: ''
severity: low
suggested-category: engine/lessons
---

# Lazy-create singleton wołany z OnDestroy tworzy obiekty przy zamykaniu sceny

## Problem
Wzorzec `Instance { get { if (null) new GameObject(...).AddComponent<T>() } }`
+ sprzątanie w `OnDestroy()` innego komponentu (`Manager.Instance.Hide(this)`).
Przy wyjściu z Play Mode / zamykaniu sceny singleton był już zniszczony, więc getter
TWORZYŁ nowy GameObject w zamykającej się scenie → błąd Unity:
"Some objects were not cleaned up when closing the scene. (Did you spawn new
GameObjects from OnDestroy?)".

## Zasada
Każdy lazy-create singleton musi mieć drugi, NIETWORZĄCY akcesor do ścieżek teardownu:
```csharp
public static void HideIfExists(Component key)
{
    if (_instance != null) _instance.Hide(key);
}
```
W OnDestroy/OnDisable używać wyłącznie wariantu nietworzącego (albo strażnika
`applicationIsQuitting`). Getter tworzący wolno wołać tylko ze ścieżek "na żądanie"
(Show/Init).

## Transfer
Uniwersalne dla wszystkich lazy singletonów UI/managerów w Unity — błąd pojawia się
dopiero przy wyjściu z Play Mode, więc łatwo przeoczyć w testach funkcjonalnych.
