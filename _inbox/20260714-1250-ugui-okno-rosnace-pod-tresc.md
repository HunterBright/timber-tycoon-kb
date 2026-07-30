---
title: 'Lekcja: okno UI rosnace pod tresc - clamp do ekranu NIE zmniejsza tresci'
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-14'
project: Kerf - Sawmill Tycoon
tags:
- unity
- ugui
- layout
- canvas-scaler
- ultrawide
- modal
applies_to: []
source: ''
suggested-category: engine/lessons
---

# Lekcja: okno UI rosnace pod tresc - clamp do ekranu NIE zmniejsza tresci

## Kontekst
Modal budowany z kodu (uGUI): panel 9-slice + tresc kotwiczona od gory + przycisk kotwiczony do dolu. Liczba elementow rosnie z danymi (kafelki gatunkow, nisze magazynu, wiersze paragonu). Klasyczne rozwiazanie "panel rosnie pod zawartosc".

## Trzy pulapki, ktore kosztowaly czas

### 1. Clamp wysokosci panelu do ekranu tylko OBCINA papier
```csharp
panelH = Mathf.Clamp(contentH + 2 * PAD, MIN, MAX);   // MAX = 992 (bo ekran 1080)
```
Wyglada bezpiecznie i jest **falszywym poczuciem bezpieczenstwa**: clamp zmniejsza PANEL, ale tresc jest kotwiczona od gory i ma wlasna wysokosc - wiec dalej wylatuje poza (obcinasz rame, a nie zawartosc). Bug widac dopiero na innym formacie ekranu.

**Poprawnie:** dobierz rozmiar elementow tak, zeby tresc **naprawde sie zmiescila** - policz uklad dla kilku skal i wez PIERWSZA, ktora wchodzi w kanwe:
```csharp
foreach (float s in new[] { 1f, 0.86f, 0.74f }) {
    ...policz gridH, queueH, panelH dla skali s...
    if (panelH <= MaxPanelHeight()) break;   // ta skala sie miesci
}
```

### 2. Sufit "92% z 1080" jest FALSZYWY na 21:9
`CanvasScaler` (ScaleWithScreenSize, ref 1920x1080, match 0.5) na ekranie 2560x1080 daje `scale = (2560/1920)^0.5 = 1.155`, czyli kanwa ma **935 px referencyjnych**, nie 1080. Kazda stala typu "max 992 bo ekran ma 1080" jest wtedy klamstwem.

**Poprawnie:** licz sufit z FAKTYCZNEJ kanwy:
```csharp
float cap = DESIGN_MAX;
var canvasRT = canvas.transform as RectTransform;
if (canvasRT != null && canvasRT.rect.height > 1f)
    cap = Mathf.Min(cap, canvasRT.rect.height - 2f * MARGIN);
```

### 3. Przebudowa siatki W SRODKU obslugi klikniecia
Jesli ksztalt siatki zalezy od zawartosci, a zawartosc zmienia sie klikiem (transfer towaru), to `OnContentsChanged -> Refresh -> Rebuild` leci **synchronicznie wewnatrz handlera klika**: element pod kursorem jest niszczony i odtwarzany w innym miejscu/rozmiarze. Bez histerezy dochodzi migotanie na granicy (20 -> 21 -> 20 elementow przebudowuje okno tam i z powrotem), a drugi szybki klik trafia w INNY obiekt.

**Poprawnie:** ksztalt ukladu w obrebie jednego otwarcia okna **tylko rosnie**, nigdy nie maleje (reset przy `Show()`).

## Bonus: pomiar rezerwy pod tekst zmienny
Jesli wysokosc panelu zalezy od dlugosci tekstu, ktory zmienia sie przy kazdym kliknieciu - NIE przeliczaj panelu co klik (elementy uciekaja spod kursora). Zmierz **najgorszy mozliwy tekst** raz przy otwarciu (`GetPreferredValues`) i zarezerwuj tyle miejsca. Kazdy realny stan jest jego podzbiorem, wiec nachodzenie staje sie niemozliwe.

## Reguly
- Rosnacy panel: licz uklad DLA KILKU SKAL i wez pierwsza, ktora sie miesci; nie ratuj sie samym clampem.
- Sufit okna licz z `canvasRT.rect`, nie z zalozonego 1080.
- Uklad zalezny od danych zmienianych klikiem: tylko rosnie w obrebie sesji okna.
- Rezerwa pod tekst: mierz worst-case raz, nie stan biezacy co klatke.
