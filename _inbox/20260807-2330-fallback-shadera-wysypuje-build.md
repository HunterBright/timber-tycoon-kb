---
type: anti-pattern
project: Kerf - Sawmill Tycoon
suggested-category: engine/anti-patterns
tags: [urp, shader, fallback, motion-vectors, skinnedmesh, crash, build, unity6]
date: 2026-08-07
status: draft
---

# FallBack we wlasnym shaderze URP wysypuje build na siatkach ze szkieletem

## Co sie stalo

Wlasny shader URP dla postaci (ForwardLit + ShadowCaster + DepthOnly + DepthNormals)
z linijka na koncu:

```
FallBack "Universal Render Pipeline/Lit"
```

W Edytorze wszystko wygladalo poprawnie. Build tez powstal. Sonda build-smoke
potwierdzila, ze shader jest w buildzie i dziala (zero magenty na 2535 rendererach).
A mimo to gra **wysypywala sie** przy pierwszym kontakcie z postaciami ze szkieletem
(naruszenie ochrony pamieci, kod wyjscia -1073741819 / 0xC0000005), i to tak wczesnie,
ze plik wynikowy sondy w ogole nie powstawal.

## Dlaczego

Shader nie mial passu **MotionVectors**. Dla obiektow ze szkieletem (SkinnedMeshRenderer)
URP liczy wektory ruchu. Skoro shader tego passu nie ma, Unity siega po niego do shadera
z `FallBack` - czyli do URP/Lit - i probuje go nakarmic **wlasciwosciami naszego
materialu**. Uklad pamieci stalych (`UnityPerMaterial`) naszego shadera jest inny niz
URP/Lit. Efekt: czytanie poza buforem i wysypka natywna.

To wyjasnia, czemu ten sam wzorzec shadera dzialal w projekcie od miesiecy na drzewach
i rekwizytach: **statyczne obiekty nie licza wektorow ruchu**, wiec nigdy nie siegaly
po brakujacy pass.

## Naprawa (dwie rzeczy, obie potrzebne)

1. **Usun `FallBack`** z wlasnego shadera URP. Brak fallbacku znaczy tyle, ze brakujacy
   pass po prostu sie nie rysuje - a to jest bezpieczne. Fallback do shadera o innym
   ukladzie wlasciwosci bezpieczny nie jest.
2. **Wylacz pass wektorow ruchu na materiale**: `m.SetShaderPassEnabled("MotionVectors", false)`
   (w pliku .mat widac to jako `disabledShaderPasses: - MOTIONVECTORS`). Warto sprawdzic,
   czy materialy, ktore podmieniasz, juz tego nie mialy - u nas oryginalne materialy NPC
   mialy to wylaczone i przeoczenie tego bylo prawdziwym zrodlem bledu.

## Reguly do zapamietania

- **Wlasny shader URP: albo miej wszystkie passy, ktorych pipeline moze zazadac, albo nie
  dawaj FallBacku.** Nigdy polowicznie.
- Passy, o ktorych sie zapomina: `DepthOnly`, `DepthNormals` (potrzebne, gdy pipeline ma
  wlaczone RequireDepthTexture albo SSAO liczone z normalnych), `MotionVectors`
  (dla wszystkiego, co sie rusza lub ma szkielet), `ShadowCaster`.
- **Objaw myli:** "shader jest w buildzie i nie jest magenta" NIE znaczy "shader jest
  poprawny". Wykrywacz magenty lapie brak shadera, nie brak passu.
- Sprawdzenie w Edytorze niczego tu nie dowodzi. To klasyczna rodzina "Edytor klamie" -
  buduj natychmiast po dodaniu wlasnego shadera.

## Jak to zostalo ustalone (metoda warta powtorzenia)

Nie zgadywanie po logu, tylko **test rozstrzygajacy**: zdjac zmiane przelacznikiem,
zbudowac, puscic sonde (kod 0), zalozyc zmiane z powrotem, zbudowac, puscic sonde
(wysypka). Dwa przebiegi w kazda strone, bo mierzona sekcja miala udokumentowana
historie niestabilnosci. Dopiero to pozwolilo powiedziec "wina jest moja" zamiast
"chyba cos z shaderem".

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260807-2230-splaszczanie-oswietlenia-per-material|20260807-2230-splaszczanie-oswietlenia-per-material]] - wspolne: unity6, shader, urp
- [[20260807-2130-urp-ssao-promien-twarze|20260807-2130-urp-ssao-promien-twarze]] - wspolne: unity6, urp
- [[20260710-2010-material-props-wrong-shader-inert|Ustawianie właściwości materiału bez sprawdzenia SHADERA = ciche nic + ryzyko nadpisania]] - wspolne: shader, urp
- [[low-poly-water-side-wave|ANTI-PATTERN: sin(X) + sin(Z) Water Shader = Side Waves]] - wspolne: shader, urp
- [[20260531-1612-quaternius-lowpoly-nature-urp-import|Importing Quaternius "Stylized Nature MegaKit" (and similar low-poly packs) into URP]] - wspolne: shader, urp
- [[20260605-1250-urp-flow-shader-scroll-sign|Scrolling/flow shaders: visual motion runs OPPOSITE to the flow vector]] - wspolne: shader, urp
<!-- /POWIAZANE:auto -->
