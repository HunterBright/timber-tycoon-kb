---
title: Blender ma w standardzie gotowe szkielety zwierzat (Rigify), zanim siegniesz po AI
type: pattern
status: draft
confidence: medium
verified: ''
date: 2026-08-05
project: GameDevOS
tags: [rigging, blender, rigify, stworzenia, czworonogi, low-poly, unity]
applies_to: [blender-5.2, unity-6, rigging]
source: 'zmierzone lokalnie: C:\Program Files\Blender Foundation\Blender 5.2\5.2\scripts\addons_core\rigify\metarigs\Animals\'
suggested-category: engine/patterns
---

# Blender ma w standardzie gotowe szkielety zwierzat (Rigify), zanim siegniesz po AI

## Kiedy stosowac

Gdy potrzebujesz szkieletu dla postaci **nieludzkiej**: czworonoga, ptaka
ze skrzydlami, stworzenia z ogonem. Czyli wtedy, gdy Mixamo odpada, bo robi
wylacznie humanoidy.

## Kroki

1. Wlacz dodatek **Rigify**. Jest w Blenderze od lat i **nie trzeba niczego
   instalowac**.
2. Wstaw gotowy metarig: Add / Armature. W Blenderze 5.2 sa dostepne
   **bird, cat, horse, shark, wolf** oraz **basic_quadruped** i **basic_human**.
3. Dopasuj kosci metarigu do swojej siatki. To jest praca reczna w trybie edycji
   i **to jest caly koszt tej sciezki**.
4. Wygeneruj rig. Rigify tworzy pelny szkielet kontrolny z kinematyka odwrotna.
5. Przed eksportem do Unity **sprowadz rig do samych kosci deformujacych**,
   inaczej wywieziesz do silnika kilkadziesiat kosci sterujacych, ktore w grze
   sa balastem.

## Dlaczego to dziala

Bo problem "szkielet dla czworonoga" ma dwie zupelnie rozne czesci, ktore latwo
pomylic:
- **rozmieszczenie kosci na siatce** (trudne do zautomatyzowania, dlatego
  wlasnie tym zajmuja sie modele AI),
- **zbudowanie dzialajacej hierarchii z kinematyka odwrotna, ograniczeniami
  i sterownikami** (rozwiazane od dawna, tylko szablonem).

Rigify rozwiazuje **druga** czesc i oddaje pierwsza czlowiekowi. Przy sylwetce
low poly dopasowanie kosci wilka do wlasnego potwora to zadanie na kilkanascie
minut, a nie problem badawczy.

## Kompromisy

- **To nie jest automatyczne rigowanie.** Nic nie wnioskuje szkieletu z siatki.
  Kosci ustawiasz sam.
- Wygenerowany rig ma **duzo kosci sterujacych**, ktore nie moga trafic do gry.
  Potrzebny jest krok czyszczenia przed eksportem.
- Metarigi pokrywaja typowe sylwetki. Dla szescionoga, osmionoga albo weza
  trzeba budowac wlasny metarig albo zlozyc go z czesci.
- Dodatek jest na licencji GPL, jak caly Blender. **Nie dotyczy to wyniku**:
  szkielet, ktory wygenerujesz, jest Twoj, bo GPL dotyczy kodu narzedzia,
  a nie plikow, ktore narzedzie produkuje.

## Warianty

**Eksport do Unity bez recznego czyszczenia.** Istnieja rozszerzenia, ktore
zamieniaja rig Rigify na czysty szkielet do gry i wypiekaja na niego akcje.
Jedno z nich deklaruje minimalna wersje Blendera 5.2. Sa na licencji GPL i sa
male, wiec **nadaja sie do przeczytania i przepisania**, jesli nie chcemy
cudzego kodu w potoku.

**Scalanie klipow animacji.** Osobny problem, ktory pojawia sie zaraz potem:
kazda animacja wychodzi w osobnym pliku, a Unity chce jednego. Istnieja
narzedzia scalajace klipy po nazwach kosci, czyli dzialajace **niezaleznie
od tego, czy szkielet jest humanoidalny**.

## Dlaczego to warto bylo zapisac

Bo szukalismy rozwiazania tego problemu przez tygodnie: odrzucone modele AI
z licencjami wykluczajacymi Unie Europejska, platne uslugi w chmurze po
0,15 do 0,25 dolara za szkielet, porzucone repozytoria badawcze. Tymczasem
**pieciu gotowych szkieletow zwierzat lezalo w programie, ktory mamy
zainstalowany**, i w calej bazie wiedzy nie bylo o tym ani jednej wzmianki.

Reguła ogolniejsza: **zanim zaczniesz szukac nowego narzedzia, sprawdz, co juz
jest w tym, ktore masz**. To samo pytanie zadaj przy Unity i przy Claude Code.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260809-1740-rigify-wolf-nie-jest-rigiem-do-gry|Metarig wilka w Rigify to rig filmowy, nie growy - do gry idzie basic_quadruped]] - wspolne: rigify, czworonogi, low-poly
- [[20260730-1950-proxy-clothing-tangential-smoothing|Ubrania proxy na low-poly ciele: wygładzanie styczne zamiast laplasjanu]] - wspolne: rigging, low-poly, blender
- [[20260807-1620-skinning-lerp-zapada-nadgarstek|20260807-1620-skinning-lerp-zapada-nadgarstek]] - wspolne: rigging, blender
- [[20260531-2000-blender-mesh-only-fbx-for-mixamo|Batch-extract clean mesh-only FBX from rigged .blend for Mixamo re-rig]] - wspolne: rigging, blender
- [[character-pipeline-tripo-mixamo-unity|Character pipeline: Tripo mesh → Mixamo rig → Unity (clean, working recipe)]] - wspolne: rigging, blender
- [[20260531-1500-mixamo-clean-mesh-extraction|Extract a clean mesh-only FBX from a rigged source for Mixamo re-rig]] - wspolne: rigging, blender
<!-- /POWIAZANE:auto -->
