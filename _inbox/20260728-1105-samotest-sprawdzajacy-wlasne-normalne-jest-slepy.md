---
title: Samotest sprawdzajacy WLASNE normalne jest slepy na odwrocona scianke
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-28'
project: Kerf - Sawmill Tycoon
tags:
- unity
- mesh
- normals
- winding
- testing
- self-test
- geometry
applies_to: []
source: ''
suggested-category: engine/lessons
---

# Samotest sprawdzajacy WLASNE normalne jest slepy na odwrocona scianke

## Objaw

Skrypt tnie siatke i zaslepia miejsce ciecia. Samotest swieci na zielono: siatka
szczelna, pole zaslepki co do drugiego miejsca po przecinku, normalne skierowane
tam, gdzie trzeba. W grze zaslepki NIE MA - widac przez nia wnetrze bryly.

## Przyczyna

O tym, ktora strona trojkata jest widoczna, decyduje **KOLEJNOSC WIERZCHOLKOW**
(nawiniecie), a nie normalna zapisana w wierzcholku. Przy domyslnym odrzucaniu
tylnych scianek karta graficzna liczy kierunek z iloczynu wektorowego:

```
normalna_rysowania = cross(b - a, c - a)
```

Kod budujacy zaslepke wpisywal normalna recznie (`Vector3.up` / `Vector3.down`)
i osobno wybieral kolejnosc wierzcholkow z kierunku obiegu petli. Kolejnosc
wyszla odwrotnie. Test sprawdzal normalne - czyli **liczby, ktore sam wpisal** -
wiec nie mial szansy tego zobaczyc.

## Regula

**Test nie moze sprawdzac danych, ktore sam wyprodukowal jako zalozenie.**
Sprawdzaj to, co faktycznie decyduje o wyniku:

```csharp
// ZLE: to tylko odczytuje z powrotem wartosc wpisana przez kod
Vector3 n = usredniona_normalna(mesh);
Assert(n.y > 0.99f);

// DOBRZE: to liczy karta graficzna
Vector3 nawiniecie = Vector3.zero;
for (int i = 0; i < tri.Length; i += 3)
    nawiniecie += Vector3.Cross(poz[tri[i+1]] - poz[tri[i]],
                                poz[tri[i+2]] - poz[tri[i]]).normalized;
Assert(nawiniecie.normalized.y > 0.99f);
Assert(Vector3.Dot(nawiniecie.normalized, usredniona_normalna) > 0.9f);  // zgodnosc obu
```

## Poprawka w samym generatorze

Zamiast liczyc kierunek obiegu petli i wybierac kolejnosc "z gory", sprawdzaj
kazdy trojkat osobno i odwracaj, gdy wyszedl w druga strone. Jest to odporne na
kierunek obiegu, dziury i petle wieloczesciowe:

```csharp
bool dobraStrona = Vector3.Dot(Vector3.Cross(p1 - srodek, p2 - srodek), zadanaNormalna) > 0f;
if (dobraStrona) Trojkat(srodek, p1, p2);
else             Trojkat(srodek, p2, p1);
```

## Powiazany trop

Kontrola szczelnosci (kazda krawedz w dwoch trojkatach) TEZ nie zlapie odwroconej
scianki - odwrocony trojkat ma te same krawedzie. Szczelnosc i nawiniecie to dwie
osobne wlasnosci i trzeba sprawdzac obie.
