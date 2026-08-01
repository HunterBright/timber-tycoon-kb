---
title: 'Anty-wzorzec: test, ktory nie ma jak zawiesc (silnik "naprawia" mierzona wielkosc)'
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-07-14'
project: Kerf - Sawmill Tycoon
tags:
- unity
- tmp
- testing
- build-smoke
- false-green
- ellipsis
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Anty-wzorzec: test, ktory nie ma jak zawiesc (silnik "naprawia" mierzona wielkosc)

## Objaw
Nowy check w build-smoke swieci na zielono od pierwszego uruchomienia i nigdy nie zapala sie na czerwono - nawet gdy cofniesz naprawe, ktorej pilnuje.

## Konkretny przypadek (Unity + TextMeshPro)
Sprawdzenie "czy dlugi tekst miesci sie w swojej ramce":

```csharp
tmp.overflowMode = TextOverflowModes.Ellipsis;   // w kodzie UI
...
float textH = tmp.GetRenderedValues(false).y;    // w tescie
bool ok = textH <= rect.height;                  // ZAWSZE true
```

`GetRenderedValues()` liczy bounds **wygenerowanych znakow**, a przy `Ellipsis` TMP generuje tylko te, ktore sie zmiescily. Mierzysz wiec skutek obciecia, nie potrzebe. Warunek "nie miesci sie" jest **matematycznie nieosiagalny**.

Ta sama pulapka dotyczy kazdej wielkosci, ktora silnik sam przycina/normalizuje przed pomiarem: `Mask`/`RectMask2D` (prostokat dziecka po masce), `ContentSizeFitter` (rozmiar po dopasowaniu), clampy kamery, `Mathf.Clamp` w gettery.

## Poprawnie
Mierz **zapotrzebowanie**, nie wynik:

```csharp
float needed = tmp.GetPreferredValues(text, rect.width, 0f).y;   // przed obcieciem
bool truncated = tmp.textInfo.characterCount < text.Length;      // czy TMP cos zjadl
bool ok = !truncated && needed <= rect.height + tol;
```

## Reguly, ktore z tego wynikaja
1. **Kazdy check musi miec udowodniony tryb porazki.** Dopisz samotest: sztuczny przypadek, ktory MUSI dac FAIL (np. element celowo wiekszy od ramy). Jesli detektor nie zapali sie na nim, cala sekcja testow jest bezwartosciowa.
2. **Porownuj dokladnie (`!=`), nie z zapasem (`<`).** Licznik "zbudowanych wierszy" z luzem jednego wiersza przepusci zgubienie pozycji.
3. **Fail-closed.** Brak elementu / element wylaczony / zerowy rozmiar to NIE jest "nic nie wystaje" (cichy PASS) - tak wlasnie wyglada regresja, w ktorej tresc znika.
4. **Wyjatek w tescie != zielony build.** Wyjatek w korutynie ubija JA, a nie proces - bramka konczyla sie exit 0 i nikt nie zauwazyl, ze sekcja w ogole sie nie wykonala. Ustaw flage "sekcja doszla do konca" i sprawdz ja na koncu.
5. **Testuj tez przypadek MALY, nie tylko skrajny.** Test tylko na maksimum przepusci regresje "zawsze maksymalny rozmiar" (okno zawsze urosniete, kafelki zawsze male).

## Koszt niezauwazenia
Zielony build-smoke, ktory nie chroni przed niczym - czyli gorzej niz brak testu, bo daje falszywe poczucie bezpieczenstwa. U nas zlapal to dopiero adversarialny przeglad kodu, nie autor.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260727-1535-gates-must-not-identify-parts-by-world-coordinate|A geometry gate that identifies body parts by raw world coordinate is a gate on credit]] - wspolne: false-green, testing
- [[20260722-1652-relative-only-test-blind-to-common-mode-error|Test porownujacy instancje MIEDZY SOBA jest slepy na blad wspolny (common-mode)]] - wspolne: false-green, testing
<!-- /POWIAZANE:auto -->
