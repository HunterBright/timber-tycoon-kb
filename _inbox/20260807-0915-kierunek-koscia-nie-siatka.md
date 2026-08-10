---
type: pattern
project: Kerf - Sawmill Tycoon
suggested-category: engine/patterns
tags: [unity, humanoid, retarget, rig, dlonie, animacja, mocap]
date: 2026-08-07
status: draft
---

# Ksztalt w siatce, kierunek w kosci

## Problem

Postacie z generatorow 3D maja dlonie ustawione "po swojemu" (kciuk w gore, wnetrze
do przodu). Naturalny odruch: obrocic wierzcholki dloni w pliku modelu. Po tygodniu
iteracji wychodzi paradoks - jeden wariant ma dobry KIERUNEK i zepsuty KSZTALT,
drugi odwrotnie.

## Dlaczego obracanie siatki nie moze zadzialac

Obrot wierzcholkow wokol kosci trzeba wygasic przy nadgarstku (inaczej powstaje
szew), wiec stosuje sie wage: `v = lerp(v, obrocone, waga_kosci * k)`. Kazdy taki
obrot **rozmazuje sie na przejsciu** - im wiekszy kat, tym mocniej skrecony
nadgarstek. Przy 90-170 stopniach dlon przestaje byc dlonia.

Drugi powod: obrot w siatce jest slepy na to, co z nadgarstkiem robi KLIP.
Mocap niesie wlasna pronacje przedramienia; korekta strojona w pozie T rozjezdza
sie w ruchu.

## Wzorzec

Rozdziel odpowiedzialnosci:
- **KSZTALT** = geometria pliku, obrotow nie stosujemy wcale;
- **KIERUNEK** = sztywny obrot KOSCI po ewaluacji animatora (LateUpdate):

```csharp
var os = (dlon.position - przedramie.position).normalized;
dlon.rotation = Quaternion.AngleAxis(kat, os) * dlon.rotation;
```

Obraca cala dlon jak bryle - zero deformacji, ten sam efekt w kazdym klipie,
strojenie na zywo w trybie gry (liczba w inspektorze zamiast regeneracji modelu).
Ten sam komponent obsluguje stopy (kat wokol pionu) i przechyl tulowia.

## Czego NIE uzywac

- **Poza wzorcowa awatara** (humanDescription/skeleton): humanoid Unity przycina
  duze skrety do zakresu miesni - zadane 75 stopni weszlo jako ~25 (zmierzone).
  Nadaje sie do drobnych korekt (do ~20 st.), nie do zmiany orientacji dloni.
- **Korekta w klipie**: dziala na WSZYSTKICH humanoidach naraz, w tym na
  postaciach juz zatwierdzonych.

## Miernik (bez niego to zgadywanie)

Kciuka na rekawiczkowej dloni nie da sie wiarygodnie ocenic z renderu. Trwaly
miernik: w Blenderze zapisz INDEKS/POZYCJE wierzcholka kciuka (najwyzszy punkt
dloni w pozie T), w Unity odnajdz go w bind pose i mierz kierunek w POZIE Z KLIPU
(skladowa przod/bok/gora). Dopiero wtedy A/B ma sens - inaczej dwie rozne wady
wygladaja na renderze tak samo.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260717-0010-generated-rig-bone-axis-defect-skeleton-transplant|Rigi z generatorów AI (Hunyuan): osie kości rozjechane z frontem modelu = wykrzywiona stopa w retargecie; lek = przeszczep szkieletu w Blenderze]] - wspolne: retarget, humanoid
- [[20260802-1620-humanoid-retarget-poza-wzorcowa|20260802-1620-humanoid-retarget-poza-wzorcowa]] - wspolne: animacja, humanoid
- [[20260531-1530-unity-humanoid-autorig-mirrored-foot|Crooked foot under Unity Humanoid = auto-rig copied the foot bind pose instead of mirroring it]] - wspolne: rig, humanoid
- [[20260725-1015-ai-autorig-proportions-crush-humanoid|Szkielet z auto-rigu AI ma inne proporcje niz siatka: postac w grze skladasie w harmonijke]] - wspolne: rig, humanoid
- [[20260726-1420-humanoid-sloty-opcjonalne-vs-wymagane|Humanoid: sloty OPCJONALNE zwracaja null na poprawnym awatarze - fallback po nazwach nie moze byc pod jednym `!isHuman`]] - wspolne: rig, humanoid
<!-- /POWIAZANE:auto -->
