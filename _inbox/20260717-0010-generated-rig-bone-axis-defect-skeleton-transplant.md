---
title: 'Rigi z generatorów AI (Hunyuan): osie kości rozjechane z frontem modelu = wykrzywiona stopa w retargecie; lek = przeszczep szkieletu w Blenderze'
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-17'
project: Kerf - Sawmill Tycoon
tags:
- unity
- blender
- rigging
- humanoid
- retarget
- hunyuan
- ai-generated-models
- mixamo
- generic
- skeleton-transplant
applies_to: []
source: ''
suggested-category: engine/lessons
---

# Rigi z generatorów AI (Hunyuan): osie kości rozjechane z frontem modelu = wykrzywiona stopa w retargecie; lek = przeszczep szkieletu w Blenderze

ZWALIDOWANE przez playtest (2026-07-17): stopy naprawione po przeszczepie; dłonie po korekcie
rollu z geometrii (sekcja niżej) też potwierdzone.

## Objaw i pomiar
NPC z modelu generowanego (Hunyuan) wygina LEWĄ stopę w kostce podczas chodu (Humanoid retarget
wspólnego klipu Mixamo). Miarodajna diagnoza: próbkowanie klipu przez PlayableGraph i średni kąt
kostki L/R po pełnym cyklu - klip na RODZIMYM rigu (X Bot): różnica 3.0 st.; na rigach Hunyuan:
34-36 st. KLUCZ METODYCZNY: wzorcem musi być znany-zdrowy rig - porównanie dwóch chorych modeli
między sobą pokazuje "równość" i maskuje wadę.

## Korzeń
Osie lokalne kości rigu Hunyuan mają rozjazd ~91 st. względem faktycznego frontu modelu (ramka
kości "myśli", że przód = bok ciała). Bind pose, pozycje stawów, siatka i wagi są CZYSTE - wada
siedzi wyłącznie w orientacjach/rollach kości. Unity buduje z nich osie mięśni Humanoid i
systematycznie wykręca stopę. Płytkie zabiegi NIE działają (sprawdzone): odmapowanie palców,
Enforce T-Pose (MakePoseValid nie rusza osi stopy), zmiana trybu cullingu.

## Lek: przeszczep szkieletu w Blenderze (gdy Mixamo odrzuca mesh)
1. DAWCA = działający szkielet mixamorig z projektu (u nas: model puli klientów po Mixamo).
2. Dopasowanie: head/tail kości dawcy -> pozycje stawów pacjenta; ROLL = konwencja dawcy
   przeniesiona przez `EditBone.align_roll(z_dawcy zrzutowane na plaszczyzne prostopadla do
   nowego kierunku kosci)` + KOREKTA FRONTU (obrot osi referencyjnych o zmierzony rozjazd
   frontow dawca-pacjent, u nas -91 st.) - bez tej korekty przeszczep przenosi wadę dalej.
3. Wagi: BEZ auto-weights - zmiana nazw grup wierzcholkow pacjenta na nazwy dawcy (mapowanie
   anatomiczne 1:1) i bind ARMATURE_NAME; delta wag 0.0 = deformacja identyczna jak przed.
4. Typ animacji w Unity: GENERIC (omija cały retarget). PUŁAPKA ŚCIEŻEK: klipy Generic wiążą
   się po pełnych ścieżkach transformów (np. `mixamorig:Hips/...`) - w finalnym prefabie korzeń
   szkieletu musi być BEZPOŚREDNIM dzieckiem obiektu z Animatorem (wrapper "Armature" z eksportu
   Blendera = klip nie animuje NIC = posąg). Najpewniej: przebudować hierarchię w bake'u prefabu
   (SetParent z worldPositionStays=true skleja transformacje wrapperów).
5. Prefaby przebudowywać W MIEJSCU (PrefabUtility.LoadPrefabContents -> SaveAsPrefabAsset) -
   zachowany GUID+fileID = zero zmian w assetach konfiguracyjnych.

## Iteracja 2: dłonie-płetwy = ANATOMIA SIATKI vs ramka kości (metryki kości ślepe!)
Po przeszczepie stopy zdrowe, ale dłonie machały "płetwami" (obrót ~90 st.). Rolle kości dłoni
były IDENTYCZNE z dawcą (delta 0.0) - wada siedziała w SIATCE: wnętrza dłoni wymodelowane ze
skrętem 90-97 st. wokół osi nadgarstka względem konwencji dawcy. KLUCZ: każda metryka porównująca
KOŚCI (rolle, osie, rotacje) jest na to ślepa - rozstrzyga pomiar GEOMETRII (płaszczyzna dopasowana
do wierzchołków dłoni pacjenta vs dawcy -> kąt skrętu -> dokładnie ta wartość jako roll kości Hand)
plus render wnętrza dłoni. Wniosek ogólny: przy modelach generowanych weryfikuj RENDEREM anatomii,
nie tylko liczbami z szkieletu.

## Weryfikacja (przed pokazaniem graczowi)
Kalibracja L/R kostki (cel: poziom wzorca) + asercja "kości się ruszają" (suma delt lokalnej
rotacji stopy po rozgrzewce; max==avg zdradza statyczną pozę = niezgodność ścieżek) + rendery
stóp z klipu + marsz po prawdziwym NavMesh ze zdjęciami. GOTCHA: obrys SkinnedMeshRenderer
zawyża wysokość postaci ~15% względem wymiarów siatki (bramki skali ustawiać z zapasem).

**Why:** modele z generatorów AI (Hunyuan/Tripo) regularnie mają niekanoniczne osie kości;
objaw wygląda na błąd animacji/importu i pochłania godziny w złym kierunku.
**How to apply:** przy "wykrzywionej stopie/kostce w chodzie" na modelu z generatora najpierw
kalibruj klip na znanym-zdrowym rigu; gdy wada potwierdzona - od razu przeszczep szkieletu
(2-3 h), zamiast serii płytkich zabiegów na awatarze.
