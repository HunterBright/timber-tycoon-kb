---
type: anti-pattern
project: Timber Tycoon
suggested-category: engine/lessons
tags: [unity, ugui, image, fillamount, sprite, builtin-resources, runtime]
date: 2026-06-12
status: draft
---

# Resources.GetBuiltinResource zwraca NULL dla sprite'ów UI (builtin-EXTRA) — pasek Filled rysuje się jako pełny quad

## Anti-pattern
W kodzie runtime (budowanie UI z kodu):

```csharp
fillBar.sprite = Resources.GetBuiltinResource<Sprite>("UI/Skin/UISprite.psd");
fillBar.type = Image.Type.Filled;
fillBar.fillAmount = 0.1f; // ignorowane!
```

Wygląda poprawnie, kompiluje się, nie rzuca wyjątku — a pasek zawsze renderuje się w 100% wypełniony.

## Dlaczego nie działa (dwa nakładające się fakty silnikowe)
1. **uGUI Image bez sprite'a ignoruje tryb Filled** — rysuje pełny prostokąt w kolorze `color`, niezależnie od `fillAmount`. Bez żadnego warninga.
2. **`Resources.GetBuiltinResource` czyta tylko "unity default resources"** (meshe, LegacyRuntime.ttf). Sprite'y UI (`UISprite.psd`, `Background.psd`, `Knob.psd`, `Checkmark.psd`) leżą w **`unity_builtin_extra`** — dostępne wyłącznie edytorowym API `AssetDatabase.GetBuiltinExtraResource<Sprite>(...)`. W runtime `GetBuiltinResource` zwraca **cicho NULL** (bez wyjątku, bez loga).

Połączenie obu = "naprawa" która wygląda na zrobioną, a bug zostaje. Zweryfikowane empirycznie w Unity 6000.3.5f1 (runtime API → NULL, editor API → UISprite OK).

## Poprawny wzorzec (runtime, działa też w buildzie, zero assetów)
```csharp
fillBar.sprite = Sprite.Create(Texture2D.whiteTexture, new Rect(0f, 0f, 4f, 4f), new Vector2(0.5f, 0.5f));
fillBar.type = Image.Type.Filled;
fillBar.fillMethod = Image.FillMethod.Horizontal; // fillOrigin 0 = Left
```

`Texture2D.whiteTexture` to wbudowana biała tekstura 4×4, zawsze dostępna w runtime. Sprite tintuje się przez `Image.color`.

## Jak diagnozować
Pierwsze pytanie przy "pasek Filled zawsze pełny": `Debug.Log(image.sprite == null)` w runtime. Editorowy inspector / edytorowe API mogą kłamać — sprawdzaj w tym samym kontekście, w którym kod faktycznie działa.

## Kontekst odkrycia
Timber Tycoon — ReputationHUDWidget (UI budowane w 100% z kodu, bez prefabu). Pierwsza "naprawa" przypisała sprite przez GetBuiltinResource i bug przetrwał commit.
