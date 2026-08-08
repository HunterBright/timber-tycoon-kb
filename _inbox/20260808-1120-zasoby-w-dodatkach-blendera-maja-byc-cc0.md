---
title: Zasoby w dodatkach Blendera maja byc na CC0, a nie na GPL dodatku
type: lesson
status: draft
confidence: high
verified: 2026-08-08
tags: [blender, licencje, assety, dodatki, prawo]
date: 2026-08-08
project: GameDevOS
source: https://docs.blender.org/manual/en/latest/advanced/extensions/licenses.html
applies_to: [blender, pipeline-assetow, licencje]
severity: high
time_lost: dwa dni zablokowanej decyzji
---

# Zasoby w dodatkach Blendera maja byc na CC0, a nie na GPL dodatku

## Objaw

Znalezlismy dodatek do Blendera z biblioteka animacji zwierzat (83 klipy, w tym
21 walkowych) - dokladnie to, czego brakowalo do gry z potworami. Manifest
deklaruje `license = ["SPDX:GPL-3.0-or-later"]`. Pojawilo sie pytanie, ktore
zablokowalo decyzje na dwa dni: **licencja kodu dodatku jest jasna, ale czy pozy
i klatki, ktore z niego wychodza do gry komercyjnej, tez sa objete GPL?**

## Przyczyna

Myslelismy, ze dodatek ma jedna licencje na wszystko. **Nie ma.** Blender rozdziela
te dwie rzeczy formalnie, tylko regula stoi na osobnej, latwej do przeoczenia
stronie podrecznika, a nie w regulaminie sklepu.

## Rozwiazanie

Regula jest trojstopniowa i brzmi doslownie tak:

> **„For assets used in add-ons, the required license is Public Domain (CC0)."**

Czyli: **dodatki** na GPL-3.0-or-later (wymog), **motywy** na dowolnej licencji
zgodnej z GPL, **zasoby wewnatrz dodatkow na CC0** (wymog).

Mechanizmem jest pole `license` w `blender_manifest.toml`, ktore jest **lista**
identyfikatorow SPDX, plus opcjonalne pole `copyright`. Poprawnie zrobiony dodatek
z wlasnymi zasobami wyglada tak:

```toml
license = ["SPDX:GPL-3.0-or-later", "SPDX:CC0-1.0"]
copyright = ["2026 Autor", "2026 Autor (bundled presets, CC0-1.0)"]
```

**Praktyczny sposob sprawdzenia, zanim zaczniesz uzywac czyjegos dodatku
z zasobami:**
1. rozpakuj paczke i przeczytaj `blender_manifest.toml`,
2. jesli w polu `license` jest **jedna** pozycja, a dodatek dostarcza tresc
   tworcza (pozy, modele, tekstury, dzwieki), **to jest niezgodnosc z regula
   platformy**, a nie Twoj problem do rozwiazania w glowie,
3. napisz do autora z prosba o dopisanie drugiej licencji, wskazujac regule
   i przyklad dodatku, ktory robi to poprawnie.

**Cala wartosc tej lekcji jest w kroku 3.** Bez znajomosci reguly prosba brzmi jak
przysluga i latwo ja zignorowac. Z regula jest to zgloszenie niezgodnosci
z warunkami platformy, na ktorej autor sam opublikowal swoja prace.

## Co NIE zadzialalo

- **Szukanie odpowiedzi w oswiadczeniach Blender Foundation o wlasnosci twojej
  pracy.** Zdania w rodzaju *„What you create with Blender is your sole property"*
  dotycza tego, co **Ty** tworzysz, i nie mowia nic o zasobach **dostarczonych**
  przez cudzy dodatek.
- **Szukanie precedensu.** Nie ma ani jednego publicznie rozstrzygnietego
  przypadku biblioteki poz na GPL.

## Dowod

Strona reguly: https://docs.blender.org/manual/en/latest/advanced/extensions/licenses.html
Ta sama regula w podreczniku 4.2 LTS, czyli obowiazywala wstecz:
https://docs.blender.org/manual/en/4.2/advanced/extensions/licenses.html
W katalogu 1329 rozszerzen platformy **dokladnie dwa** deklaruja pare GPL plus
CC0. Katalog dopuszcza tez czysty MIT (7 dodatkow), CC0 (3) i Zlib (2), wiec
autor ma realny wybor.

## Czego ta lekcja NIE rozstrzyga

**Blender nigdzie nie definiuje slowa „asset" w tym zdaniu.** Jesli dodatek trzyma
dane jako opisy symboliczne, a klatki generuje kod w chwili uzycia, nie wiadomo,
czy to jest jeszcze „asset". Egzekwowanie jest slabe: wytyczne dla recenzentow nie
maja ani jednej pozycji o licencjach zasobow.

## Czy to przeniesie sie na inny projekt

Tak, na kazdy, w ktorym Blender dostarcza czesc tresci tworczej. Dotyczy tak samo
bibliotek poz, zestawow materialow, presetow geometrii i paczek dzwiekow.

## Powiazane

- [[20260805-1815-rigify-ma-gotowe-szkielety-zwierzat]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260801-1245-regulamin-uslugi-a-licencja-wag-to-dwa-rozne-swiaty|Regulamin uslugi w chmurze i licencja pobieranych wag to dwa rozne dokumenty - sprawdzaj oba]] - wspolne: prawo, licencje
- [[20260801-1140-licencja-modelu-ai-to-trzy-osobne-dokumenty|Licencja modelu AI to trzy osobne dokumenty i wystarczy, ze jeden zabroni]] - wspolne: prawo, licencje
- [[20260725-0625-ai-model-community-license-excludes-eu|Nie stawiaj pipeline'u assetow na modelu AI z licencja "Community", nie czytajac pierwszej linii LICENSE]] - wspolne: prawo, licencje
- [[20260801-0826-trellis-2-generator-3d-bez-blokady-ue|TRELLIS.2 jako generator 3D bez blokady licencyjnej w UE]] - wspolne: licencje, blender
<!-- /POWIAZANE:auto -->
