---
title: AddComponent<Canvas> dokłada RectTransform - kolejny AddComponent<RectTransform> rzuca wyjątkiem i po cichu urywa budowę UI w Awake
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-22'
project: Kerf - Sawmill Tycoon
tags:
- unity
- ugui
- canvas
- recttransform
- addcomponent
- awake
- silent-failure
applies_to:
- unity-6
- ugui
- code-built-ui
source: ''
severity: high
time_lost: ~20 min
promoted: '2026-07-30'
---

# AddComponent<Canvas> dokłada RectTransform - kolejny AddComponent<RectTransform> rzuca wyjątkiem i po cichu urywa budowę UI w Awake

## Problem

Do okien budowanych z kodu dokładałem osobne płótno (`Canvas` + `GraphicRaycaster`),
żeby wymusić kolejność warstw. Wywołanie wstawiłem tuż po utworzeniu obiektu nakładki:

```csharp
overlay = new GameObject("StatisticsOverlay");
overlay.transform.SetParent(canvas.transform, false);
UILayers.Promote(overlay, order);                        // dokłada Canvas
RectTransform rt = overlay.AddComponent<RectTransform>(); // <-- WYJĄTEK
```

Objaw w grze był zupełnie niepowiązany z przyczyną: dwa okna (statystyki, napisy końcowe)
**zostawały widoczne od startu** i na stałe zgłaszały się jako otwarte. Wszystko, co pytało
"czy gracz jest w jakimś oknie", dostawało odpowiedź "tak" przez całą sesję - więc dymki nad
NPC znikały na zawsze, a klawisz ESC zamykałby najpierw statystyki zamiast otwierać pauzę.

## Root cause

`AddComponent<Canvas>()` **automatycznie dokłada `RectTransform`** (Canvas go wymaga).
Następne `AddComponent<RectTransform>()` na tym samym obiekcie rzuca wyjątkiem
"Can't add component 'RectTransform' ... because such a component is already added".

Wyjątek leci w `BuildUI()` wołanym z `Awake()`. Unity **łapie wyjątek z Awake i leci dalej** -
komponent nie jest niszczony, scena wstaje, gra działa. Ale reszta `Awake` po `BuildUI()`
nigdy się nie wykonuje - a tam siedziało `overlay.SetActive(false)`. Okno zostaje włączone.

Dwa mechanizmy złożyły się na ciszę:
1. wyjątek w `Awake` nie zatrzymuje gry, tylko urywa jedną metodę,
2. właściwość `IsOpen => overlay.activeSelf` zwracała prawdę, bo nikt tej nakładki nie wyłączył.

## Solution

Płótno dokładać **po** skonfigurowaniu `RectTransform` nakładki, nie przed:

```csharp
overlay = new GameObject("StatisticsOverlay");
overlay.transform.SetParent(canvas.transform, false);
RectTransform rt = overlay.AddComponent<RectTransform>();
rt.anchorMin = Vector2.zero; rt.anchorMax = Vector2.one;
rt.offsetMin = Vector2.zero; rt.offsetMax = Vector2.zero;
UILayers.Promote(overlay, order);                        // dopiero teraz
```

Odporniejszy wariant helpera: nigdy nie zakładać, że obiekt nie ma jeszcze `RectTransform`,
i zawsze używać `GetComponent<T>() ?? AddComponent<T>()` - także po stronie WOŁAJĄCEGO.

## What didn't work

- Szukanie winnego po objawie ("które okno zgłasza się jako otwarte?") przez czytanie kodu
  okien - kod okien był poprawny, `SetActive(false)` w `Awake` stało tam od zawsze.
- Założenie, że wyjątek w `Awake` byłby widoczny. W buildzie leci do `player.log` i nie
  przerywa niczego - konsola w Edytorze też go tylko odnotowuje.

Co zadziałało od razu: **diagnostyka wypisująca NAZWY** okien zgłaszających się jako otwarte
(`DescribeOpen()`), zamiast samego bool-a. Sonda wypisała "otwarte: statystyki, napisy końcowe"
i przyczyna była na stole w 30 sekund.

## Transferability

Dotyczy każdego projektu Unity budującego uGUI z kodu, niezależnie od gatunku. Ta sama pułapka
występuje przy innych komponentach z `[RequireComponent]` albo z domyślnie dokładaną
zależnością (`Canvas` → `RectTransform`, `Rigidbody2D`/`Collider2D` w niektórych układach).

Ogólniejsza lekcja, ważniejsza od samego API: **wyjątek w `Awake` to cicha awaria częściowa** -
obiekt zostaje w stanie w połowie zbudowanym i kłamie o swoim stanie. Predykaty w rodzaju
`IsOpen => obiekt.activeSelf` zamieniają wtedy błąd budowy w trwałą, mylącą odpowiedź.

## Related
- [[gate-must-have-provable-failure-mode]] (jeszcze nie istnieje)

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260623-0855-unity-layoutelement-requirecomponent-recttransform-null-trap|AddComponent<RectTransform>() zwraca null po wcześniejszym AddComponent komponentu z [RequireComponent(RectTransform)]]] - wspolne: recttransform, addcomponent, awake
<!-- /POWIAZANE:auto -->
