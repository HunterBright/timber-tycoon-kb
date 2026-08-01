---
title: Sprawdzanie "czy tekst sie miesci" przez pomiar RectTransform
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-07-21'
project: Kerf - Sawmill Tycoon
tags:
- unity
- ui
- textmeshpro
- testing
- probe
- layout
- tautology
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Sprawdzanie "czy tekst sie miesci" przez pomiar RectTransform

## The trap

Automat pilnujacy ukladu UI mierzy, czy element miesci sie w ramie okna:

```csharp
FitsInside(hintRect, contentRect, tol, out float over);   // wyglada solidnie
```

Wydaje sie, ze to chroni przed rozjezdzajacym sie tekstem - w koncu "sprawdzamy, czy podpowiedz
miesci sie w papierze". Nie chroni. Ten check **nie moze zawiesc** dla napisu o stalej ramce.

## Why it fails

`RectTransform` napisu ma rozmiar nadany przez uklad (np. rezerwa 44 px), a nie przez TRESC.
TextMeshPro rysuje tekst dluzszy niz ramka **poza nia** i przy `overflowMode = Overflow`
(domyslnie) nie zmienia przy tym ani rozmiaru, ani pozycji `RectTransform`.

Wynik: ramka o stalej wysokosci zawsze siedzi w papierze, wiec check zawsze daje PASS - a gracz
widzi trzy linijki tekstu wchodzace na zawartosc pod spodem.

Blizniacza pulapka po drugiej stronie: pomiar `GetRenderedValues()` na napisie z
`overflowMode = Ellipsis`. Ten mierzy tekst **juz przyciety**, wiec tez nigdy nie przekroczy
rezerwy. Check-tautologia w obie strony.

## Symptoms

- automat swieci na zielono, a na zrzucie ekranu tekst nachodzi na inny element,
- check "czy X miesci sie w Y" nigdy w historii projektu nie zapalil sie na czerwono,
- nie potrafisz wskazac konkretnej regresji, ktora ten check by wylapal,
- rezerwa na napis jest stala liczba w kodzie, a tresc pochodzi z tlumaczen albo z danych gracza
  (nazwy, kwoty, liczniki) - czyli moze urosnac bez zmiany ani jednej linijki ukladu.

## Correct approach

Mierz **ile tekst POTRZEBUJE**, nie ile zajmuje jego ramka:

```csharp
// napis bez zawijania -> w poziomie
needed = tmp.GetPreferredValues(tmp.text, 0f, 0f).x;   have = rect.width;

// napis zawijany -> w pionie, PRZY SZEROKOSCI swojej ramki
needed = tmp.GetPreferredValues(tmp.text, rect.width, 0f).y;   have = rect.height;

if (needed <= 0.01f) return false;   // FAIL-CLOSED: 0 px to nieudany pomiar, nie "miesci sie"
return needed <= have + tol;
```

Trzy zasady, ktore ratuja te rodzine checkow:

1. **Pomiar potrzeby, nie stanu po przycieciu** (`GetPreferredValues`, nie `GetRenderedValues`).
2. **Fail-closed** - brak elementu, zerowy rozmiar albo nieudany pomiar to FAIL, nie cichy PASS.
   Inaczej regresja, w ktorej tresc ZNIKA, wyglada identycznie jak sukces.
3. **Ustaw `NoWrap` tam, gdzie zawijanie i tak byloby bledem** (napisy na przyciskach). Wtedy za
   dlugi napis WYSTAJE i automat go widzi; przy domyslnym zawijaniu rozlalby sie na druga linijke
   po cichu i tylko brzydko wygladal.

Dodatkowo: sprawdzaj nachodzenie elementow WEWNATRZ kafelka, nie tylko kafelka wzgledem okna.
Kazdy element z osobna moze miescic sie i w oknie, i we wlasnym polu, a mimo to napis moze
lezec na przycisku.

## Related
- [[gate-must-have-provable-failure-mode]]
- [[20260721-1210-screencapture-mid-frame-captures-previous-frame]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260723-2140-silent-loc-fallback-antipattern|Cichy fallback lokalizacji ukrywa nieprzetłumaczoną treść]] - wspolne: probe, testing
- [[20260721-1210-screencapture-mid-frame-captures-previous-frame|ScreenCapture.CaptureScreenshotAsTexture w ciele korutyny lapie POPRZEDNIA klatke]] - wspolne: probe, ui
- [[20260713-1200-silent-null-guard-hides-dead-ui|Fabryka, która po cichu zwraca obiekt bez wymaganego dziecka]] - wspolne: testing, ui
- [[20260617-1210-tmp-text-legibility-on-textured-bg|TextMeshPro: czytelność na teksturowanym tle (drewno) + warstwy modali]] - wspolne: textmeshpro, ui
- [[typography-accessibility-stack|Typography + Accessibility Stack]] - wspolne: textmeshpro, ui
<!-- /POWIAZANE:auto -->
