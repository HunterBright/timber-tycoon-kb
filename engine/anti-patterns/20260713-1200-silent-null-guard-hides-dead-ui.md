---
title: Fabryka, która po cichu zwraca obiekt bez wymaganego dziecka
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-07-13'
project: Kerf - Sawmill Tycoon
tags:
- unity
- ui
- factory
- null-guard
- silent-failure
- testing
applies_to: []
source: ''
severity: high
promoted: '2026-07-30'
---

# Fabryka, która po cichu zwraca obiekt bez wymaganego dziecka

## Anty-wzorzec

Fabryka UI tworzy element opcjonalnie, „bo po co tworzyć pusty":

```csharp
public static Image CreateButton(..., string label, ...)
{
    ...
    if (!string.IsNullOrEmpty(label))          // <-- PUŁAPKA
    {
        CreateText("Label", go.transform, ...); // dziecko powstaje TYLKO gdy jest tekst
    }
    ...
}
```

Wywołujący, który chce ustawiać tekst później (bo w chwili budowy okna go jeszcze nie zna),
robi rzecz naturalną:

```csharp
btn = UIFactory.CreateButton("Option1", parent, skin, "", OnClick, green);
label = btn.GetComponentInChildren<TextMeshProUGUI>();   // ---> null, na zawsze
...
if (label != null) label.text = "Załaduj na pakę";       // ---> ciche NIC
```

Efekt: **przycisk to kolorowy pasek bez treści. Na zawsze. Bez jednego błędu w konsoli.**
Cały dialog był w takim stanie **od commita, w którym powstał** - i nikt tego nie zauważył,
bo dało się klikać „z pamięci mięśniowej" (góra = ta akcja, dół = tamta). Awaria wyszła
dopiero, gdy doszedł trzeci przycisk i pozycje się przesunęły.

## Dwa błędy, które się tu spotkały

1. **Fabryka niekompletna**: zwraca obiekt, którego kontraktu (`.text = ...`) **nie da się już
   nigdy spełnić**, i nie mówi o tym ani słowa.
2. **Strażnik nulla jako tłumik**: `if (label != null) label.text = ...` zamienia „mój obiekt
   jest zepsuty" w „nic się nie stało". Strażnik miał chronić przed wyjątkiem, a schował awarię.

## Reguła

- **Fabryka ma być totalna.** Jeśli zwracany obiekt ma dawać się później skonfigurować
  (`SetText`, `SetIcon`, `SetColor`), to elementy tej konfiguracji **muszą powstać zawsze** -
  także w wariancie „pusty". Pusty napis nic nie rysuje, więc nic nie kosztuje.
- **Nie tłum nulla tam, gdzie null znaczy „jestem zepsuty".** `if (x != null)` jest w porządku,
  gdy null jest legalnym stanem. Gdy null oznacza, że konstrukcja obiektu się nie udała -
  ma być głośno (`Debug.LogError`) albo w ogóle nie ma być możliwy.
- **Cache'owanie referencji w czasie budowy jest kruche**: `GetComponentInChildren` w `CreateUI`
  zapisuje null raz i na zawsze. Jeśli musisz cache'ować, sprawdź od razu po pobraniu.

## Dlaczego testy tego nie łapią

Bo testy wołają **logikę wprost**, z pominięciem interfejsu. Ten sam projekt zaliczył tego dnia
**trzy warianty tej samej rodziny**:

| Objaw | Logika | Interfejs |
|---|---|---|
| Witryny nie reagują na E | zdrowa | brak collidera - promień nie ma w co trafić |
| Prompt „[E]" wisi w powietrzu | zdrowa | model 8 m od collidera |
| Menu bez treści | zdrowa | przyciski bez napisów |

**Nazwa klasy: „logika zdrowa, interfejs martwy".** Test, który woła `manager.DoThing()`, przejdzie
w każdym z tych trzech przypadków, a gracz nie może zrobić nic.

## Test, który to łapie

Asercja na **kontrakt widoczności**, nie na wynik logiki:

```csharp
// 1) kontrakt fabryki
var probe = UIFactory.CreateButton("Probe", parent, skin, "", () => { }, Color.white);
Assert(probe.GetComponentInChildren<TextMeshProUGUI>() != null,
       "przycisk z pustym napisem nie ma dziecka 'Label' - zostanie pustym paskiem");

// 2) prawdziwe okno: pokaż je i sprawdź, że napisy SĄ i są DOKŁADNIE te, które podano
dialog.Show(header, piece, "AKCJA_1", ..., "AKCJA_2", ...);
AssertButtonText(canvas, "Option1Button", "AKCJA_1");
```

Analogicznie dla świata 3D: „każdy interaktywny obiekt ma collider na właściwej warstwie",
„collider pokrywa się z modelem".

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260721-1215-ui-fit-check-measuring-rect-instead-of-text|Sprawdzanie "czy tekst sie miesci" przez pomiar RectTransform]] - wspolne: testing, ui
<!-- /POWIAZANE:auto -->
