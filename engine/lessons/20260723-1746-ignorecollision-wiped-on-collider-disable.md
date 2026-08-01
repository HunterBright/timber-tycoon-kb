---
title: Physics.IgnoreCollision znika przy wyłączeniu collidera - dla przełączanych colliderów używaj par warstw
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-23'
project: Kerf - Sawmill Tycoon
tags:
- unity
- physics
- ignorecollision
- layers
- charactercontroller
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Physics.IgnoreCollision znika przy wyłączeniu collidera - dla przełączanych colliderów używaj par warstw

## Problem

Unity kasuje WSZYSTKIE punktowe wyjątki `Physics.IgnoreCollision(a, b)` w momencie, gdy
którykolwiek z colliderów zostanie wyłączony (`enabled = false`) albo jego GameObject
dezaktywowany. Wyjątek trzeba by nakładać od nowa po każdym włączeniu.

W praktyce: kapsuła gracza (CharacterController) jest rutynowo wyłączana przy wsiadaniu
do pojazdu i przy teleportach. Każdy system, który raz w `Start()` zawołał
`IgnoreCollision(playerCC, cośtam)`, cicho przestawał działać po pierwszej jeździe autem.
W Kerf tak umarła "furtka" w ścianie granicznej (BoundaryFootGate) i tak samo umarłby
planowany wyjątek gracz-auta NPC.

## Rozwiązanie

Dla par, w których którykolwiek collider bywa wyłączany: `Physics.IgnoreLayerCollision`
(para warstw) zamiast wyjątków per-collider. Ignorowanie na poziomie warstw NIE jest
kasowane przy wyłączaniu colliderów. Koszt: osobna warstwa (np. PlayerFoot tylko dla
pieszego gracza, pojazd gracza zostaje na Default, żeby dalej kolidował z tym, z czym ma).

Wzorzec aplikacji: statyczna klasa z `[RuntimeInitializeOnLoadMethod(BeforeSceneLoad)]`
wołająca `IgnoreLayerCollision` - przeżywa restarty edytora, działa w buildzie, zero sceny.

## Kiedy per-collider jest OK

Gdy collidery żyją stabilnie (nigdy nie są wyłączane), a wyjątek jest naprawdę punktowy -
np. piesi NPC z poolingu, gdzie `IgnoreCollision` i tak jest nakładane od nowa w każdym
`Init()` (pooling = dezaktywacja = i tak by znikło).

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260628-1615-footstep-surface-raycast-from-feet|Detekcja nawierzchni pod graczem: strzelaj raycastem OD STÓP, nie od środka postaci]] - wspolne: charactercontroller, physics
- [[20260716-1812-charactercontroller-depenetration-thin-mesh-terrain|CharacterController + cienki jednostronny teren-siatka = gracz pod mapą (i jak się przed tym bronić)]] - wspolne: charactercontroller, physics
- [[20260722-2055-raycast-w-gore-nie-widzi-tafli-wody|"Czy jestem pod wodą?" - promień w GÓRĘ nic nie zobaczy (backface)]] - wspolne: layers, physics
<!-- /POWIAZANE:auto -->
