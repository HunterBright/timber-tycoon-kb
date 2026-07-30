---
title: Collider z materiałem `null` NIE ma zerowego tarcia - ma DOMYŚLNE 0.6
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-14'
project: Kerf - Sawmill Tycoon
tags:
- unity
- physics
- physicsmaterial
- friction
- editor-only
- assetdatabase
- silent-failure
- vehicle
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Collider z materiałem `null` NIE ma zerowego tarcia - ma DOMYŚLNE 0.6

## Severity
Wysoka. Cichy błąd fizyki: nic nie krzyczy, nic nie loguje, a pojazd/obiekt zachowuje się
zupełnie inaczej niż w Edytorze.

## Kontekst

Pojazd arcade'owy: kadłub `BoxCollider` uniesiony nad ziemię, cztery `SphereCollider` na kołach.
Auto jest pchane **siłą przyłożoną do kadłuba**, a koła mają się tylko **toczyć i ślizgać** -
dlatego wszystkie collidery dostają materiał zerowego tarcia.

Materiał wczytywany z assetu:

```csharp
PhysicsMaterial zeroFriction = null;
#if UNITY_EDITOR
    zeroFriction = AssetDatabase.LoadAssetAtPath<PhysicsMaterial>("Assets/ZeroFriction.physicMaterial");
#endif

box.material = zeroFriction != null ? zeroFriction : new PhysicsMaterial(...) { /* fallback */ };
...
sphere.material = zeroFriction;      // <-- BEZ fallbacku. W buildzie: null.
```

## Lekcja

**`collider.material = null` nie znaczy "brak tarcia". Znaczy "weź domyślny materiał Unity",
czyli dynamicFriction 0.6 / staticFriction 0.6 / frictionCombine Average.**

Nie ma ostrzeżenia, nie ma wyjątku, nie ma czerwonego tekstu. Collider po prostu dostaje
przyzwoite, realistyczne tarcie - czyli **dokładnie to, czego ten pojazd nie może mieć**.

W Timber Tycoon: `AssetDatabase` istnieje wyłącznie w Edytorze, więc w buildzie `zeroFriction`
było `null`. Kadłub miał fallback, **koła nie**. Koła są JEDYNYM kontaktem auta z ziemią - więc
w buildzie auto przyklejało się do drogi i nie mogło ruszyć:

> `NPC STUCK` → teleport awaryjny → po którymś teleporcie auto wypadało pod mapę (Y = −955).

Klient nigdy nie dojeżdżał do sklepu. **Cały kanał sprzedaży nie istniał w prawdziwej grze,
a w Edytorze wszystko działało bez zarzutu.**

## Drugi haczyk: `frictionCombine`

Nawet gdy ustawisz `dynamicFriction = 0` i `staticFriction = 0`, to **nie wystarczy**, jeśli
zapomnisz o `frictionCombine`.

Domyślna kombinacja to **Average**: efektywne tarcie pary = średnia z obu materiałów.
Twoje 0 uśrednione z drogą (0.6) daje **0.3** - czyli znowu tarcie tam, gdzie go nie ma być.

Materiał "zerowego tarcia" musi mieć **`frictionCombine = Minimum`** ("weź NAJNIŻSZE z pary").
Bez tego zerowanie liczb jest iluzoryczne.

W YAML assetu to pole `m_FrictionCombine: 2` (0=Average, 1=Multiply, 2=Minimum, 3=Maximum).
Ręcznie sklejany fallback `new PhysicsMaterial(...)` **nie kopiuje tego pola**, jeśli o nim
nie pamiętasz - i wtedy "fallback z tymi samymi parametrami tarcia" jest cichym kłamstwem.

## Correct approach

**Nie trzymaj materiału fizycznego jako assetu, jeśli to pięć liczb.** Twórz go w kodzie:

```csharp
private static PhysicsMaterial cached;

private static PhysicsMaterial ZeroFriction()
{
    if (cached == null)
    {
        cached = new PhysicsMaterial("ZeroFriction")
        {
            dynamicFriction = 0f,
            staticFriction  = 0f,
            bounciness      = 0f,
            frictionCombine = PhysicsMaterialCombine.Minimum,   // BEZ TEGO zerowanie nic nie da
            bounceCombine   = PhysicsMaterialCombine.Minimum,
            hideFlags       = HideFlags.HideAndDontSave,        // jeden na sesję, nie śmieci w scenie
        };
    }
    return cached;
}
```

Wtedy **Edytor i build robią dokładnie to samo** - a to jest jedyna prawdziwa obrona przed tą
rodziną błędów. Użyj `sharedMaterial`, nie `material` (to drugie tworzy instancję per collider).

## Jak wykryć

- `grep` po `\.material = ` i `\.sharedMaterial = ` w kodzie runtime. Każde przypisanie, którego
  źródłem może być `null`, jest podejrzane.
- Każde `AssetDatabase` w kodzie runtime (nawet pod `#if UNITY_EDITOR`) to sygnał, że w buildzie
  jakaś zmienna zostaje `null` - **sprawdź, co się z nią dalej dzieje**.
- Objaw w grze: obiekt zachowuje się "ciężej"/"lepiej się trzyma" w buildzie niż w Edytorze.
  Pojazdy grzęzną, obiekty nie zjeżdżają po pochyłościach, ragdolle się zaklejają.

## Powiązane

[[20260714-2320-if-unity-editor-fixes-the-build-and-kills-the-game]] - to jest jego bliźniak. Ta sama łatka
`#if UNITY_EDITOR` zabiła w tym samym pliku DWIE rzeczy: powstawanie auta (naprawione rano)
i tarcie kół (naprawione wieczorem). **Gdy zakładasz taką bramkę, przejdź CAŁY plik i sprawdź
KAŻDĄ zmienną, która może zostać `null` w buildzie** - nie tylko tę, na której wywalał się
kompilator.
