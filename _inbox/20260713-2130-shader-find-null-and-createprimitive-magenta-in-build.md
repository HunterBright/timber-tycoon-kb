---
title: 'Magenta w buildzie: Shader.Find zwraca null, a CreatePrimitive daje material, którego build nie ma'
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-13'
project: Kerf - Sawmill Tycoon
tags:
- unity
- urp
- shader-find
- always-included-shaders
- createprimitive
- magenta
- build
- shader-stripping
applies_to:
- unity
source: ''
severity: high
time_lost: ~1h (diagnoza + fix + weryfikacja w buildzie)
suggested-category: engine/lessons
---

# Magenta w buildzie: Shader.Find zwraca null, a CreatePrimitive daje material, którego build nie ma

## Problem

W Edytorze wszystko wygląda poprawnie. W **buildzie** obiekty świecą na **magenta** (różowo), albo
jakaś funkcja po prostu **nie działa i nikt nie wie dlaczego** (bez crasha, bez błędu na ekranie).

Konkretnie u nas:
- 12 znaczników dostawy przy domkach = wielkie **różowe placki** na trawie.
- Podświetlanie celów questów (obwódka wokół obiektu) **w ogóle nie działało** w buildzie.

## Root cause

Build **wycina shadery**, których nie używa żaden materiał w projekcie. To celowa optymalizacja
(shader variants potrafią ważyć setki MB). Edytor niczego nie wycina, bo ma cały AssetDatabase.

Stąd dwie pułapki:

### 1. `Shader.Find("X")` zwraca **null** w buildzie

...jeśli shadera „X" nie używa **żaden materiał** ani nie ma go na liście
**Project Settings → Graphics → Always Included Shaders**.

```csharp
var shader = Shader.Find("Custom/QuestOutline");   // Edytor: znajdzie. Build: NULL.
if (shader == null) { Debug.LogError("..."); return; }   // <- null-check jest, wiec bez crasha
```
Efekt: **cicha śmierć funkcji.** Kod nawet ma poprawny null-check, więc nic się nie wywala — funkcja
po prostu nie istnieje w wydanej grze. To najgorszy rodzaj błędu: nie zostawia śladu.

Materiał `new Material(null)` → obiekt renderuje się **magentą** (`Hidden/InternalErrorShader`).

### 2. `GameObject.CreatePrimitive()` przypisuje **domyślny materiał Unity**

W projekcie na URP ten domyślny materiał (built-in `Default-Material` / Standard) **nie jest przez
nic używany**, więc build go wycina → prymityw tworzony w runtime jest **magenta**.

```csharp
var ring = GameObject.CreatePrimitive(PrimitiveType.Cylinder);
var r = ring.GetComponent<Renderer>();
r.SetPropertyBlock(mpb);   // ustawiamy _BaseColor... na materiale, ktorego w buildzie NIE MA
```
**MaterialPropertyBlock nie ratuje sytuacji** — koloruje materiał, którego nie ma.

## Solution

**Dla shadera wołanego przez `Shader.Find`:** dopisz go do **Always Included Shaders**
(`ProjectSettings/GraphicsSettings.asset`, lista `m_AlwaysIncludedShaders`). To jedyny sposób, by
shader nieużywany przez żaden materiał trafił do buildu.

**Dla `CreatePrimitive`:** ZAWSZE przypisz własny materiał, nigdy nie licz na domyślny.

```csharp
Shader s = Shader.Find("Universal Render Pipeline/Lit")
           ?? Shader.Find("Universal Render Pipeline/Unlit");
if (s != null) r.material = new Material(s) { color = gold };
else Debug.LogError("Brak shadera URP - obiekt bedzie MAGENTA.");
```

Shadery URP/Lit i URP/Unlit są bezpieczne, **o ile** używa ich jakikolwiek materiał w projekcie
(a w projekcie URP zawsze używa). Shadery własne (`Custom/*`) i legacy (`Standard`, `Unlit/Color`,
`Particles/Standard Unlit`) — **sprawdź, nie zakładaj.**

## Automatyczny wykrywacz (to jest sedno)

Ręczne pilnowanie tego nie zadziała. Wpisz to w sondę, która **uruchamia się w prawdziwym buildzie**:

```csharp
// (a) czy cokolwiek WIDOCZNEGO renderuje sie shaderem bledu?
foreach (Renderer r in FindObjectsByType<Renderer>(FindObjectsInactive.Include, FindObjectsSortMode.None))
{
    if (!r.enabled || !r.gameObject.activeInHierarchy) continue;   // ukrytego gracz nie zobaczy
    foreach (Material m in r.sharedMaterials)
        if (m == null || m.shader == null || m.shader.name.Contains("InternalErrorShader"))
            Fail($"MAGENTA: {PathOf(r.transform)}");
}

// (b) czy kazdy shader szukany przez kod faktycznie JEST w buildzie?
foreach (string name in CriticalShaders)
    if (Shader.Find(name) == null) Fail($"BRAK shadera: {name}");
```

Podziel listę shaderów na **krytyczne** (brak = zepsuta żywa funkcja → FAIL) i **opcjonalne**
(używane tylko jako fallback albo przez martwy kod → tylko informacja). Inaczej sonda utonie w szumie:
u nas z 6 „brakujących" shaderów realnie szkodził **jeden**.

Listę shaderów do sprawdzenia wyciągnij z kodu, nie z pamięci:
```bash
grep -rn 'Shader\.Find("' --include=*.cs Assets | grep -v "/Editor/" \
  | sed -E 's/.*Shader\.Find\("([^"]+)"\).*/\1/' | sort -u
```
**Uwaga na własną pomyłkę:** przy `grep -h` (bez nazw plików) `| grep -v "/Editor/"` **nic nie filtruje**,
bo w strumieniu nie ma już ścieżek. Dwa z sześciu „braków" okazały się skryptami edytorowymi.

## Transferability

Każdy projekt Unity używający **SRP (URP/HDRP)** — a więc praktycznie każdy nowy. Ryzyko rośnie wszędzie
tam, gdzie obiekty i materiały powstają **w kodzie w runtime** (znaczniki, podświetlenia, wizualki minigier,
debugowe kształty) zamiast być assetami w scenie.

Reguła kciuka: **jeśli shader nie jest użyty przez żaden materiał w projekcie, dla builda on nie istnieje.**
Ta sama zasada dotyczy `Resources.Load` po nazwie i wszystkiego, co szuka assetu stringiem — build widzi
tylko to, do czego prowadzi łańcuch referencji.

## Related
- [[20260713-1900-build-early-never-built-project-hides-editor-only-bugs]]
- [[20260713-1830-runtime-meshcollider-needs-readwrite-and-editor-cannot-prove-it]]
