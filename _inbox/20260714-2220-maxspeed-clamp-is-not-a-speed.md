---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [unity, physics, rigidbody, vehicle, drag, upgrades, balance]
date: 2026-07-14
status: draft
---

# maxSpeed to KLAMRA, nie prędkość - pojazd i tak stanie na (napęd / tłumienie)

## Severity
Średnia - nie wywala gry, ale **cicho unieważnia kupowane ulepszenia**, więc gracz płaci za coś,
czego nie dostaje. Trudne do zauważenia, bo wszystko "działa".

## Kontekst

Typowy arcade'owy kontroler pojazdu w Unity:

```csharp
rb.AddForce(forward * input * acceleration * rb.mass, ForceMode.Force);
rb.linearDamping = drag;                                   // liniowe tłumienie
if (rb.linearVelocity.magnitude > maxSpeed)                // "ogranicznik"
    rb.linearVelocity = rb.linearVelocity.normalized * maxSpeed;
```

Do tego dane pojazdu w ScriptableObjekcie (`maxSpeed: 25 / 30 / 35`) i ulepszenia w sklepie:
"Auto II - szybsze", "Auto III - jeszcze szybsze".

## Lekcja

Przy sile napędu proporcjonalnej do masy (`ForceMode.Force` z `* rb.mass`) przyspieszenie w m/s^2
równa się polu `acceleration`, niezależnie od masy. Tłumienie liniowe (`rb.linearDamping`) daje
równanie:

```
v(t+dt) = (v(t) + a*dt) / (1 + d*dt)
```

którego stan ustalony to **dokładnie `v = a / d`**.

Czyli: **prędkość maksymalna pojazdu to napęd podzielony przez tłumienie - i nic więcej.**
`maxSpeed` jest tylko klamrą, która przycina prędkość *jeśli* pojazd ją przekroczy.

Jeśli `a/d < maxSpeed`, to klamra **nigdy się nie włącza** i `maxSpeed` nie robi absolutnie nic.
Podnoszenie samego `maxSpeed` (np. przez ulepszenie pojazdu) podnosi wtedy sufit, do którego pojazd
i tak nie dojeżdża.

W Timber Tycoon: `a=22`, `d=2` -> prędkość ustalona **11 m/s (~40 km/h)**, a `maxSpeed` w danych
wynosił 25/30/35. Menedżer pojazdu ustawiał **tylko** `MaxSpeed` przy zakupie ulepszenia.
Efekt: Auto II (700 zł) i Auto III (2000 zł) dawały **wyłącznie pojemność paki**. Prędkość została
11 m/s na zawsze, przez cały rok prac, i nikt tego nie zauważył - bo auto przecież jeździło.

## Jak wykryć

- Policz `acceleration / drag` i porównaj z `maxSpeed`. Jeśli iloraz jest MNIEJSZY - klamra jest
  martwa, a `maxSpeed` to fikcja.
- Objaw w grze: pojazd rozpędza się do jakiejś prędkości i twardo na niej stoi, a zmiana `maxSpeed`
  nic nie daje. Zmiana `drag` daje bardzo dużo (i "przypadkiem" naprawia problem, co myli trop).

## Naprawa

Wyprowadź napęd Z ŻĄDANEJ prędkości, zamiast ustawiać obie liczby niezależnie:

```csharp
public void ApplyDrivetrainForTopSpeed(float topSpeed)
{
    maxSpeed = topSpeed;
    acceleration = topSpeed * Mathf.Max(0.01f, drag);   // bo v_ustalone = a / d
}
```

Wtedy `maxSpeed` w danych pojazdu jest PRAWDZIWĄ prędkością szczytową i ulepszenia realnie działają.

**Zachowaj odczucie startowego pojazdu:** wpisz do danych startowego pojazdu tę prędkość, którą
faktycznie osiągał do tej pory (u nas: 11, nie 25). Inaczej "naprawa" podwoi prędkość auta, którym
gracz jeździł przez cały rozwój gry, i zepsuje prowadzenie, kolizje i ruch NPC.

## Pułapka przy okazji

`FindAnyObjectByType<VehicleController>()` przy synchronizacji danych pojazdu potrafi trafić w **auto
NPC** zamiast w auto gracza (oba mają ten sam komponent). Szukaj auta gracza po tym, co je wyróżnia
(u nas: pakę - `VehicleStorage`), a nie po typie kontrolera.
