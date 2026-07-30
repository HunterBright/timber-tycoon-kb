---
title: '`#if UNITY_EDITOR` naprawia build i po cichu zabija system'
type: anti-pattern
status: draft
confidence: low
verified: ''
date: '2026-07-14'
project: Kerf - Sawmill Tycoon
tags:
- unity
- build
- editor-only
- assetdatabase
- prefabutility
- silent-failure
- ifdef
applies_to: []
source: ''
suggested-category: engine/anti-patterns
---

# `#if UNITY_EDITOR` naprawia build i po cichu zabija system

## The trap

Gra nie chce się zbudować, bo w kodzie runtime siedzi wywołanie API edytora
(`AssetDatabase`, `PrefabUtility`, `EditorUtility`). Kompilator player-a nie zna tych typów:

```
error CS0103: The name 'AssetDatabase' does not exist in the current context
```

Odruch, który wygląda na oczywistą naprawę:

```csharp
#if UNITY_EDITOR
    var prefab = AssetDatabase.LoadAssetAtPath<GameObject>(path);
    car = (GameObject)PrefabUtility.InstantiatePrefab(prefab);
#else
    Debug.LogError("tylko editor");
    Destroy(root);
    return;                 // <-- TU UMIERA GRA
#endif
```

Build przechodzi. Bramka świeci na zielono. **A system, którego dotyczył ten kod, w prawdziwej
grze nie istnieje.**

## Why it fails

`#if UNITY_EDITOR` nie jest naprawą - to **przeniesienie błędu z kompilacji do rozgrywki**.
Kompilacja jest głośna (czerwony tekst, build się nie da zrobić). Rozgrywka jest cicha: gra
działa, wygląda normalnie, tylko *czegoś w niej nie ma*.

To jest zamiana błędu, którego NIE DA SIĘ przeoczyć, na błąd, którego prawie NIE DA SIĘ zauważyć.

Wzmacniacz: w Edytorze **wszystko nadal działa idealnie**. Testujesz, grasz, iterujesz - i nic
Ci nie zgrzyta. Dopiero gracz w prawdziwym buildzie zauważa, że czegoś brakuje. A często nawet
nie zauważa, tylko myśli, że tak ma być.

## Symptoms

- W runtime'owym pliku (poza folderem `Editor/`) jest `#if UNITY_EDITOR`, a gałąź `#else`
  zawiera `return`, `Destroy`, `LogError("tylko editor")` albo pustkę.
- Log z buildu (NIE z Edytora) zawiera komunikat w rodzaju "tylko editor", "editor only",
  "not supported in build".
- Coś "nie działa w buildzie", a w Edytorze działa. Zwłaszcza: obiekty się nie pojawiają.

## Correct approach

**Nie pytaj "jak sprawić, żeby to się skompilowało". Pytaj "jak zrobić TO SAMO bez API edytora".**

Prawie zawsze się da, bo API edytora jest tu wygodą, nie koniecznością:

| Edytorowe (nie działa w buildzie) | Runtime'owe (działa wszędzie) |
|---|---|
| `PrefabUtility.InstantiatePrefab(prefab)` | `Instantiate(prefab)` |
| `AssetDatabase.LoadAssetAtPath<T>(path)` | referencja w ScriptableObjekcie / `[SerializeField]` / `Resources.Load<T>` |
| `EditorUtility.SetDirty` | nie potrzebne w runtime |

Kluczowa obserwacja: **zasób i tak jest w buildzie**, jeśli referencuje go ScriptableObject albo
scena. Problem nigdy nie dotyczył zasobu - tylko SPOSOBU jego wczytania.

Zasada nadrzędna: **niech Edytor i build robią DOKŁADNIE TO SAMO.** Każdy `#if UNITY_EDITOR`
w kodzie rozgrywki to rozjazd, w którym zalęgnie się błąd "działa u mnie".

Jeśli API edytora naprawdę musi zostać (np. narzędzie deweloperskie), to gałąź `#else` ma
**dawać sprawną grę bez tej funkcji**, a nie zabijać obiekt i wychodzić.

## Jak to złapać (bo code review tego nie łapie)

Kod wygląda **poprawnie**. Bramka jest jawna, komentarz sensowny, `LogError` obecny.
Recenzent widzi "obsłużony przypadek", a nie "zabity system".

Jedyne, co to łapie, to **sprawdzenie zachowania w prawdziwym buildzie**:

> Czy najprostsza rzecz pod słońcem w ogóle się dzieje? Czy klient przyjeżdża?
> Czy da się cokolwiek sprzedać?

Sonda smoke mierzyła siatki, shadery i okna UI - i przechodziła 62/62 - podczas gdy **gra była
nieprzechodzalna poza samouczkiem**. Bo nikt nie napisał checku na rzecz zbyt oczywistą, żeby ją
sprawdzać.

**Lekcja o testach:** pokrycie techniczne (czy siatka ma collider) nie zastąpi pokrycia
funkcjonalnego (czy da się grać). Dopisz do build-smoke jeden check na każdą pętlę, bez której
gra nie ma sensu.

## Case study (Timber Tycoon, 2026-07-14)

13.07: naprawa błędu kompilacji buildu - `AssetDatabase` w `NPCVehicleTestBootstrap`.
Rozwiązanie: `#if UNITY_EDITOR` + `Destroy(root); return;` w gałęzi buildu.
Build zaczął się kompilować. Ogłoszono sukces ("PIERWSZY DZIAŁAJĄCY BUILD GRY").

14.07: reżyser gra w build i zgłasza, że po questcie tutorialowym nie przyjeżdża żaden klient.

Okazuje się, że **przez cały ten czas w buildzie nie powstawał ANI JEDEN klient**. Zero klientów
= zero sprzedaży = zero pieniędzy i reputacji = zero progresji. Gra nie dawała się przejść, a
jednocześnie właśnie zaczynaliśmy kalibrować jej ekonomię - na kanale sprzedaży, który w buildzie
nie istniał.

Naprawa błędu kompilacji stworzyła błąd rozgrywki. Naprawa zajęła 15 minut; znalezienie -
przypadkowy playtest.
