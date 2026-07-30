---
title: CharacterController.velocity „zamraża się" gdy przestajesz wołać Move() → audio sterowane ruchem przecieka
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-06-25'
project: Kerf - Sawmill Tycoon
tags:
- unity
- charactercontroller
- audio
- footsteps
- velocity
- minigame
- input-lock
applies_to: []
source: ''
promoted: '2026-07-30'
---

# CharacterController.velocity „zamraża się" gdy przestajesz wołać Move() → audio sterowane ruchem przecieka

## Objaw
Postać wchodzi w interakcję/minigrę (która odcina sterowanie ruchem), a system kroków
gra kroki dalej - w nieskończoność, mimo że gracz stoi.

## Przyczyna (engine-level)
`CharacterController.velocity` zwraca prędkość obliczoną przy OSTATNIM wywołaniu `Move()`.
Jeśli kontroler gracza przestaje wołać `Move()` (bo minigra/dialog/UI przejęły sterowanie),
`velocity` NIE spada do zera - zostaje zamrożona na ostatniej wartości (np. prędkość marszu).
Każdy system, który czyta `characterController.velocity.magnitude` jako „czy gracz idzie"
(np. footstep system, head-bob, particle trails) będzie działał tak, jakby gracz wciąż się ruszał.

Uwaga: to NIE dotyczy normalnego zatrzymania (gdy gracz puszcza klawisze, kontroler nadal
woła `Move(zero)` co klatkę → velocity → 0). Problem występuje tylko gdy `Move()` PRZESTAJE być wołane.

## Rozwiązanie
Audio/efekty sterowane ruchem muszą sprawdzać ten SAM warunek „gracz zajęty", który blokuje ruch -
jedno źródło prawdy. Jeśli ruch jest blokowany przez metodę typu `IsAnyMinigameActive()` /
flagę `canMove` / stan UI, ten sam gating wstaw do systemu kroków:

```csharp
void Update()
{
    if (playerController != null && !playerController.canMove) return;          // istniejąca brama
    if (playerController != null && playerController.IsInteractionActive()) return; // ta sama, co blokuje Move()
    // ... dopiero teraz czytaj velocity i graj kroki
}
```

Alternatywa: zerować własną cache prędkości gdy Move() nie jest wołane - ale gating wspólnym
stanem jest pewniejszy (kroki i blokada ruchu nie rozjadą się przy dodaniu nowej minigry).

## Generalizacja
Każda mechanika czytająca `CharacterController.velocity` poza pętlą, która faktycznie woła `Move()`,
musi zakładać, że ta wartość może być nieaktualna. Traktuj „czy gracz się rusza" jako stan
sterowany przez warstwę inputu/sterowania, nie przez surowy odczyt z kontrolera.
