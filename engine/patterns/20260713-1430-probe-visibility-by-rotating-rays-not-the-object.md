---
title: 'Sonda widoczności: obracaj PROMIENIE, nie obiekt'
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-07-13'
project: Kerf - Sawmill Tycoon
tags:
- unity
- raycast
- occlusion
- visibility
- physics
- minigame
- ux
applies_to:
- unity
source: ''
promoted: '2026-07-30'
---

# Sonda widoczności: obracaj PROMIENIE, nie obiekt

## Kontekst

Trzeba odpowiedzieć na pytanie: „czy gracz zobaczy tę część obiektu, jeśli obróci go o X stopni?" - żeby np. samo ustawić najlepszy kąt, wykryć części zasłonięte albo zweryfikować, czy cel jest w ogóle osiągalny myszą.

Naiwna implementacja: obróć obiekt → `Physics.SyncTransforms()` → wystrzel promienie → przywróć obrót. Powtórz dla 12 kątów.

## Problem naiwnego wariantu

- Wymaga `Physics.SyncTransforms()` po każdym obrocie (przy `autoSyncTransforms = 0`, czyli w domyślnej konfiguracji nowszych projektów, bez tego promienie trafiają w collidery sprzed obrotu).
- Musi przywrócić obrót **przed końcem klatki**, inaczej gracz widzi migotanie. To zamyka drogę do rozłożenia sondy na kilka klatek - a sonda bywa droga.
- Przerwanie w środku (ESC, wczytanie zapisu, wyjątek) zostawia obiekt w losowym obrocie.
- Kłóci się z każdą inną korutyną, która akurat rusza tym obiektem (animacje, przenoszenie, skalowanie).

## Rozwiązanie

Obrót obiektu o kąt t przy nieruchomej kamerze jest **matematycznie tożsamy** z obrotem promienia o −t wokół tej samej osi. Więc sonduj „wirtualną kamerą" na okręgu, a obiekt zostaw w spokoju:

```csharp
static Ray RayAtYaw(Ray ray, Vector3 axisPoint, Vector3 axisDir, float yawDelta)
{
    var q = Quaternion.AngleAxis(-yawDelta, axisDir);
    return new Ray(axisPoint + q * (ray.origin - axisPoint), q * ray.direction);
}

// prostokąt ekranowy części po hipotetycznym obrocie: punkt świata obracamy o +yawDelta
Vector3 PointAtYaw(Vector3 p, Vector3 axisPoint, Vector3 axisDir, float yawDelta)
    => axisPoint + Quaternion.AngleAxis(yawDelta, axisDir) * (p - axisPoint);
```

Zyski: zero `SyncTransforms` w pętli, zero migotania, zero ryzyka zostawienia obiektu w złym obrocie, sondę można spokojnie rozłożyć na klatki (jeden kąt na klatkę).

## Miara widoczności - dwie liczby, nie jedna

Sam procent zasłonięcia kłamie na małych częściach. Nóżka szafy „widoczna w 25%" to kilka pikseli spod cokołu; cokół „widoczny w 20%" to szeroki pas przez pół obiektu. Dlatego mierz oba:

- **procent zasłonięcia** = promienie, w których badana część wygrała / promienie, które w nią w ogóle trafiły (mianownik ignorujący zasłony - odporne na chude kształty, gdzie prostokąt bounds jest w większości pusty);
- **widoczna powierzchnia** jako ułamek ekranu = „czy jest w co celować myszą".

Reguła „schowana" = (procent poniżej progu) LUB (powierzchnia poniżej progu), z wyjątkiem: część widoczna w ~100% nigdy nie jest „za mała" - to po prostu mała część, której nic nie zasłania.

Trzymaj tę regułę w JEDNEJ funkcji, używanej i przez gameplay, i przez testy - inaczej test mierzy coś innego niż gra.

## Kiedy stosować

Montaż / demontaż / malowanie / naprawa / inspekcja obiektów złożonych z części; auto-kadrowanie kamery na „element, którym gracz się teraz zajmuje"; audyt assetów („czy w każdą część każdego modelu da się kliknąć?").

## Koszt

Pętla `Collider.Raycast` po N colliderach × M promieni. Rząd wielkości: 12 kątów × 25 promieni × 10 colliderów ≈ 3 tys. zapytań ≈ kilka ms - mieści się w klatce. Wariant „siatka na każdą część osobno × każdy kąt" (60 tys. zapytań) **nie mieści się** - to 30-120 ms, czyli widoczne szarpnięcie. Jeśli potrzeba więcej: osobna warstwa dla części + `Physics.Raycast` z maską (jedno zapytanie natywne zamiast pętli po colliderach) tnie koszt o rząd wielkości.

## Related
- [[20260713-1420-convex-meshcollider-swallows-hollow-interiors]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260702-0651-batch-machine-loop-pattern|Wzorzec: pętla serii maszyny z minigrą (batch-machine loop)]] - wspolne: ux, minigame
- [[20260713-1420-convex-meshcollider-swallows-hollow-interiors|Convex MeshCollider na skorupie połyka jej wnętrze - promienie nigdy nie trafią w części w środku]] - wspolne: minigame, raycast, physics
- [[diegetic-3d-button-raycast|Diegetic 3D Button Raycast Pattern]] - wspolne: minigame, raycast
- [[mesh-collider-convex-for-clickable-minigame-objects|Convex MeshCollider for Irregular Clickable Objects]] - wspolne: minigame, raycast
- [[20260722-2050-unstuck-nearest-valid-ground-ring-search|Unstuck / reset: szukaj najbliższego POPRAWNEGO gruntu zamiast teleportu do bazy]] - wspolne: raycast, physics
- [[20260629-1917-diegetic-buttons-frame-minigame|Console buttons FRAME a skill minigame (they're flow-control, not the mechanic)]] - wspolne: minigame, raycast
<!-- /POWIAZANE:auto -->
