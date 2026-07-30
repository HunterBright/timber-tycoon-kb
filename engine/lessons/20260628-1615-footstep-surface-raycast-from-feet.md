---
title: 'Detekcja nawierzchni pod graczem: strzelaj raycastem OD STÓP, nie od środka postaci'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-06-28'
project: Kerf - Sawmill Tycoon
tags:
- unity
- footsteps
- raycast
- physics
- meshcollider
- surface-detection
- character-controller
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Detekcja nawierzchni pod graczem: strzelaj raycastem OD STÓP, nie od środka postaci

## Severity
Medium - psuje immersję (zły dźwięk kroków / zła nawierzchnia), trudne do zdiagnozowania bo „wzrokowo wszystko OK".

## Objaw
Na jednolicie wyglądającym podłożu (np. trawa) kroki grały raz trawę, raz żwir, raz ziemię. Tekstura się zgadzała, dźwięk nie.

## Przyczyna (root cause)
System kroków robił `Physics.RaycastAll` z punktu **nad środkiem postaci** (`transform.position + up*0.3`) w dół i brał najbliższy/pierwszy znaleziony kolider. Przy obiektach, których kolider „pływa" nieco nad terenem (drogi/dekale/mosty ~0.3-0.6 m nad gruntem), ten promień trafiał w kolider wiszący **na wysokości kolan**, mimo że **stopy gracza stoją na terenie pod spodem**. Wynik: nawierzchnia z obiektu, którego gracz fizycznie nie dotyka.

## Lekcja / fix
Raycast wykrywający „na czym stoję" MUSI startować tuż nad **podeszwą kapsuły**, nie nad środkiem:

```csharp
float halfH = cc.height * 0.5f;
Vector3 bottom = transform.position + cc.center - Vector3.up * (halfH - cc.skinWidth);
Vector3 origin = bottom + Vector3.up * 0.05f;
RaycastHit[] hits = Physics.RaycastAll(origin, Vector3.down, groundCheckDistance + 0.1f, groundMask);
// pierwszy (najbliższy) trafiony = powierzchnia kontaktu = to, na czym faktycznie stoisz
```

Krótki zasięg (≈ `groundCheckDistance`) + start od stóp = bierzesz dokładnie powierzchnię kontaktu i ignorujesz wszystko wiszące nad podeszwą. To samo dotyczy detekcji „w wodzie": uznawaj zanurzenie tylko gdy NIE ma stałej powierzchni między stopami a taflą (inaczej pomost/most nad wodą = fałszywe „w wodzie").

## Powiązane pułapki
- **Footstep/nawierzchnia ≠ tekstura wizualna.** Dźwięk zależy od kolidera + znacznika, nie od materiału. Trawa-dekoracja narzucona na drogę nadal da dźwięk drogi.
- **Non-convex MeshCollider dodany/zmieniony w EDIT MODE się nie „cookuje"** - raycasty go nie trafiają w edytorze; rejestruje się dopiero w Play Mode. Weryfikuj takie kolidery w grze, nie w edycji.
- Unity 6.5: `Object.GetInstanceID()` jest obsolete → `GetEntityId()`.
