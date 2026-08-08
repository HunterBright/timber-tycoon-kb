---
type: pattern
project: Kerf - Sawmill Tycoon
suggested-category: engine/patterns
tags: [urp, shader, postacie, oswietlenie, ssao, unity6, styl-graficzny]
date: 2026-08-07
status: draft
---

# Splaszczanie oswietlenia PER MATERIAL, nie globalnie

## Problem, ktory to rozwiazuje

Rezyser oglada postacie i mowi: "chce, zeby wygladaly plasko, jak namalowana tekstura,
bez tego brudu na twarzy". Naturalny odruch to podkrecic swiatlo rozproszone i sciszyc
slonce - ale te ustawienia sa GLOBALNE. Splaszczysz twarze i przy okazji splaszczysz
budynki, teren i cala reszte, ktora akurat wygladala dobrze.

## Wzorzec

Nie ruszaj ustawien sceny. Napisz osobny shader dla tej jednej rodziny obiektow
i splaszcz oswietlenie **w nim**. Reszta swiata zostaje nietknieta.

Trzy dzwignie w shaderze, w kolejnosci wplywu:

1. **Plaskie swiatlo rozproszone.** Zamiast `SampleSH(normal)` uzyj `SampleSH(float3(0,1,0))`
   - czyli pobierz oswietlenie tak, jakby cala powierzchnia patrzyla w gore.
   ```hlsl
   float3 plaskie    = SampleSH(float3(0, 1, 0));
   float3 kierunkowe = SampleSH(normal);
   float3 rozproszone = lerp(plaskie, kierunkowe, _Kierunkowosc);  // 0.15 = prawie plasko
   ```
   To jest **najwiekszy** zysk, jesli scena ma ustawienie Trilight z ciemnym kolorem od ziemi:
   wtedy kazda powierzchnia skierowana w dol (oczodol, spod brwi, spod nosa, spod brody)
   dostawala ciemny kolor i wygladala na ubrudzona.

2. **Oslabione slonce.** `mainLight.color * NdotL * shadowAttenuation * _SilaSlonca` z sila
   rzedu 0.35. Twarz przestaje byc rzezba, a nadal przyjmuje **kolor** slonca, wiec zyje
   z pora dnia. To jest roznica miedzy "plasko" a "martwo".

3. **Pominiecie przyciemniania zakamarkow.** Wystarczy NIE czytac
   `_ScreenSpaceOcclusionTexture`. Swiat dalej dostaje SSAO, ta rodzina obiektow nie -
   i nie trzeba ruszac ustawien renderera ani kombinowac z warstwami.

## Czego nie wolno zgubic

Wlasny shader latwo napisac tak, ze zabiera rzeczy, ktorych nikt nie zamawial:

- **pass ShadowCaster** - bez niego postac przestaje rzucac cien i wyglada, jakby unosila sie
  nad ziemia. To jest bardzo widoczny blad, a latwo go przeoczyc w renderze bez podlogi.
- **pass DepthOnly i DepthNormals** - bez nich obiekt wypada z bufora glebi i normalnych.
  Skutek: efekty czytajace glebie widza przez postac, a SSAO reszty sceny robi obwodki
  wokol jej sylwetki.
- **petla swiatel dodatkowych** - inaczej lampy przestaja oswietlac postacie w nocy.
  W trybie Forward+ petla wymaga zadeklarowanej struktury `InputData` (pozycja w swiecie
  + wspolrzedne ekranowe), inaczej shader sie nie kompiluje.
- **odwracanie normalnej dla tylnych scianek** (`facing : VFACE`), jesli material bywa
  dwustronny - bez tego tylne scianki wychodza czarne.
- **uklad CBUFFER zgodny z SRP Batcher** - wszystkie wlasciwosci materialu w jednym
  `CBUFFER_START(UnityPerMaterial)`. Inaczej kazda postac to osobne wywolanie rysowania.

## Kiedy NIE stosowac

Gdy splaszczenia potrzebuje cala scena - wtedy zmiana ustawien jest prostsza i uczciwsza.
Wzorzec oplaca sie dopiero, gdy jedna rodzina obiektow ma wygladac inaczej niz reszta.

## Dowod, ze dziala

Zmierzone na 4 postaciach (kontrast na twarzy, w stopach jasnosci, przed -> po):
1.197 -> 0.737, 1.530 -> 0.838, 1.477 -> 0.850, 1.637 -> 0.855. Udzial przyciemniania
zakamarkow na postaciach: 0.134 -> 0.000. Wyglad swiata: bez zmiany, bo nic globalnego
nie ruszono.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260807-2130-urp-ssao-promien-twarze|20260807-2130-urp-ssao-promien-twarze]] - wspolne: oswietlenie, ssao, postacie
- [[20260807-2330-fallback-shadera-wysypuje-build|20260807-2330-fallback-shadera-wysypuje-build]] - wspolne: unity6, shader, urp
- [[four-phase-weighted-smoothstep-day-night|4-Phase Weighted Smoothstep Day/Night Transition]] - wspolne: shader, urp
- [[procedural-skybox-sun-moon-trick|Procedural Skybox Sun/Moon Trick]] - wspolne: shader, urp
- [[20260710-2010-material-props-wrong-shader-inert|Ustawianie właściwości materiału bez sprawdzenia SHADERA = ciche nic + ryzyko nadpisania]] - wspolne: shader, urp
- [[low-poly-water-side-wave|ANTI-PATTERN: sin(X) + sin(Z) Water Shader = Side Waves]] - wspolne: shader, urp
<!-- /POWIAZANE:auto -->
