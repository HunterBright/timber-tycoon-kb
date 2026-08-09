---
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: engine/lessons
tags: [blender, rig, skinning, bone-heat, generatory-3d, wagi]
date: 2026-08-05
status: draft
---

# Automatyczne wagi (bone heat) pekaja na siatkach z generatorow - licz z odcinkow kosci

## Problem
Model postaci z generatora 3D (Tencent, tryb etapowy) sklada sie z 20+ OSOBNYCH bryl
(guziki, detale butow, oczy). "Automatic Weights" Blendera (rozplyw ciepla) konczy sie
"Bone Heat Weighting: failed to find solution" i - co gorsze - czesc wierzcholkow zostaje
BEZ wag, a uzupelniacz najblizszym sasiadem potrafi zalac cala postac jedna para kosci.
Zmierzone: dlon, stopa i biodra na [Chest 0.52, Neck 0.48] - postac to rzezba przyklejona
do klatki piersiowej. Bramka rozdarc tego NIE widzi (nic sie nie rwie, gdy nic sie nie
rusza wzgledem siebie) - widzi to dopiero bramka dominacji obszarow.

## Co NIE dziala
Sklejenie kopii voxel remeshem i liczenie ciepla na niej - w naszym przypadku solver
padl nawet na szczelnej bryle (przyczyna nieustalona; nie inwestowac w dalsza diagnoze).

## Co dziala (plan B, deterministyczny)
Wagi z ODCINKOW kosci: dla kazdego wierzcholka 3 najblizsze kosci-odcinki
(odleglosc punkt-odcinek), waga 1/d^4, odciecie ponizej 2%. Ostry wykladnik daje niemal
twarde przypisanie z waskim przejsciem w stawach; standardowe wygladzanie i petla
domykajaca rozdarcia szlifuja reszte. Bramki dominacji przeszly z zapasem
(dlon 88% na kosci dloni). Zero zaleznosci od czarnej skrzynki solvera.

## Lekcja przy okazji: poprzeczka bramek
Progi bramek skalibrowane na INNEJ postaci (kitel dlugi vs krotki) bywaja nieosiagalne.
Zanim uznasz wynik za porazke, zmierz POPRZEDNIA WERSJE PRZYJETA przez rezysera tymi
samymi bramkami - u nas zaakceptowana wersja miala 225 rozdarc, a "oblana" nowa 80.
Punktem odniesienia jest ostatni zaakceptowany produkt, nie liczba w skrypcie.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260807-1620-skinning-lerp-zapada-nadgarstek|20260807-1620-skinning-lerp-zapada-nadgarstek]] - wspolne: skinning, blender
- [[20260801-0826-trellis-2-generator-3d-bez-blokady-ue|TRELLIS.2 jako generator 3D bez blokady licencyjnej w UE]] - wspolne: generatory-3d, blender
- [[20260725-2320-fartuch-skinning-srednia-dwoch-ud-daje-zero|Fartuch ważony po połowie na oba uda NIE RUSZA SIĘ przy chodzie]] - wspolne: skinning, blender
- [[20260531-1530-unity-humanoid-autorig-mirrored-foot|Crooked foot under Unity Humanoid = auto-rig copied the foot bind pose instead of mirroring it]] - wspolne: rig, blender
- [[20260725-1015-ai-autorig-proportions-crush-humanoid|Szkielet z auto-rigu AI ma inne proporcje niz siatka: postac w grze skladasie w harmonijke]] - wspolne: rig, blender
- [[20260730-1217-bone-side-names-vs-axis-sign|Nie zgaduj strony ciała ze znaku osi ani z nazwy kości (.L/.R)]] - wspolne: rig, blender
<!-- /POWIAZANE:auto -->
