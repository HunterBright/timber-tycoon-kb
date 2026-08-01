---
title: 'URP: źle skonfigurowany materiał przezroczysty to CICHA porażka, której wykrywacz magenty nie widzi'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-13'
project: Kerf - Sawmill Tycoon
tags:
- unity
- urp
- materials
- transparency
- build-verification
- silent-failure
- shader-stripping
applies_to: []
source: ''
promoted: '2026-07-30'
---

# URP: źle skonfigurowany materiał przezroczysty to CICHA porażka, której wykrywacz magenty nie widzi

## Kontekst

Projekt miał już sondę buildową z „wykrywaczem magenty": skanuje renderery za materiałami bez shadera i sprawdza, czy shadery szukane przez `Shader.Find` są w buildzie. Powstała po tym, jak `GameObject.CreatePrimitive` + `new Material(Shader.Find(...))` dały różowe placki w buildzie.

Zadanie: zamienić nieprzezroczysty marker na półprzezroczysty słup (URP/Unlit, Surface = Transparent).

## Lekcja

**Wykrywacz magenty łapie tylko materiały BEZ shadera. Materiał przezroczysty, w którym rozjechał się jeden z kilku stanów, ma shader CAŁKOWICIE POPRAWNY - więc przechodzi przez wykrywacz bez słowa, a renderuje się jako LITY.**

To osobna klasa błędu niż magenta:
- magenta = brak shadera = krzykliwa, natychmiast widoczna
- lity zamiast przezroczystego = shader OK = **cicha**, wygląda jak „decyzja projektowa"

W URP przezroczystość to nie jedno pole, tylko komplet stanów, które muszą zajść **naraz**:

| stan | wartość dla Transparent + Alpha |
|---|---|
| `_Surface` | 1 |
| `_Blend` | 0 (Alpha) |
| `_SrcBlend` / `_DstBlend` | 5 (SrcAlpha) / 10 (OneMinusSrcAlpha) |
| `_ZWrite` | 0 |
| słowo kluczowe | `_SURFACE_TYPE_TRANSPARENT` włączone |
| `renderQueue` | 3000 |
| tag | `RenderType = Transparent` |
| przebiegi | `DepthOnly` / `DepthNormals` / `SHADOWCASTER` wyłączone przy ZWrite Off |

Dodatkowo: **`_Surface`/`_ZWrite`/`_Blend` w pliku `.mat` są BEZWŁADNE dla własnych shaderów**, które mają `Blend`/`Queue` zapisane na sztywno w SubShaderze. Woda w tym projekcie ma `_Surface: 0` i mimo to jest przezroczysta - shader ignoruje te pola. Czytanie `.mat` w poszukiwaniu „czy to przezroczyste" jest więc zwodnicze.

## Co z tym zrobić

1. **Nie pisz `.mat` ręcznie w YAML.** Twórz materiał skryptem edytorowym przez API (`SetFloat` / `EnableKeyword` / `SetShaderPassEnabled` / `renderQueue`) - te same wywołania, które robi Inspektor. Skrypt idempotentny: kolejny run odświeża wartości, nie zmienia GUID-a.
2. **Dopisz do sondy buildowej sprawdzenie STANU PRZEZROCZYSTOŚCI**, nie tylko „czy shader nie jest null". Jeden brakujący stan = twardy FAIL z nazwą stanu. Bez tego jedynym detektorem jest ludzkie oko w playteście.
3. **Sprawdź, czy projekt ma JAKIKOLWIEK precedens.** Tutaj nie miał: **każdy** `.mat` w całym repo miał `_Surface: 0` - nawet szyby samochodu i szkło lampy były nieprzezroczyste. Brak wzorca do skopiowania to sygnał ostrzegawczy, nie drobiazg.

## Powiązane

- Gotowy shader Unity (URP/Unlit) wpięty w materiał-asset ma **krótszy łańcuch zależności** niż własny shader: nie trzeba go dopisywać do Always Included Shaders ani pilnować, żeby jakiś materiał go używał. Własny shader, którego nie używa żaden materiał, **wypada z buildu** i `Shader.Find` zwraca null (patrz shader stripping always included).
- Materiał wpięty w ScriptableObject leżący w `Resources/` jest wciągany do buildu razem z shaderem - automatycznie, przez graf zależności. To wystarcza; nie trzeba `Shader.Find`.
- [[build-is-the-only-truth-editor-lies]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260713-2130-shader-find-null-and-createprimitive-magenta-in-build|Magenta w buildzie: Shader.Find zwraca null, a CreatePrimitive daje material, którego build nie ma]] - wspolne: shader-stripping, urp
- [[20260718-0800-particle-visibility-water-sorting|Czasteczki "dzialaja, ale ich nie widac" - trzy niezalezne przyczyny przy wodzie]] - wspolne: transparency, urp
- [[20260702-2140-shader-property-stale-serialized-material-values|Dodanie property do shadera może aktywować STARE, ukryte wartości w materiałach]] - wspolne: materials, urp
- [[20260710-1300-vertex-colors-vs-basecolor-linear|Vertex colors vs _BaseColor w Linear color space - ten sam hex renderuje się INACZEJ]] - wspolne: materials, urp
- [[20260719-1605-paper-shell-culling-seethrough|Prześwity w modelach low-poly: najpierw sprawdź _Cull materiału, nie geometrię]] - wspolne: materials, urp
- [[20260721-1830-linerenderer-flat-on-surface-invisible|LineRenderer lezacy plasko na powierzchni znika, bo material jest jednostronny]] - wspolne: materials, urp
<!-- /POWIAZANE:auto -->
