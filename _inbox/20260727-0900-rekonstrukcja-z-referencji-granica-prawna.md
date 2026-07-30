---
type: decision
project: Kerf - Sawmill Tycoon
suggested-category: workflow/decisions
tags: [prawo-autorskie, referencje, assety, modele-3d, wspolpraca]
date: 2026-07-27
status: draft
---

# Referencja a kopia: gdzie przebiega granica przy odtwarzaniu cudzego modelu

## Kontekst

Reżyser znalazł komercyjny model bazowy postaci (3 dolary, licencja obejmująca użycie
komercyjne) i poprosił o odwzorowanie go na podstawie sześciu podglądów w widoku
krawędziowym, bez kupowania. Uzasadnił to przekonaniem, że "odwzorowanie w 99% nie
podlega prawu autorskiemu, bo nie jest kopią 1:1".

## Sprostowanie, które trzeba było zrobić

**Nie istnieje próg procentowy.** Liczy się istotne podobieństwo do chronionego
wyrazu. Im bliżej oryginału, tym MOCNIEJSZY dowód, że powstał utwór zależny -
świadome odwzorowanie w 99% jest przypadkiem najbardziej, a nie najmniej, objętym
ochroną.

Rozdzielenie, które działa w praktyce:

| wolno | nie wolno |
|---|---|
| układ pętli krawędzi (standard warsztatowy) | to konkretne rozłożenie wierzchołków |
| proporcje anatomiczne | ta konkretna siatka odtworzona z podglądu |
| poza bazowa, gęstość, sposób prowadzenia barku | plik przerobiony i podany jako własny |
| nauczenie się, JAK coś zbudowano | celowanie w wynik nieodróżnialny od oryginału |

Chronione jest **wyrażenie**, nie **idea ani metoda**. "Niska siatka człowieka
w A-pozie z pętlami wokół oczu" to idea. To konkretne 765 wierzchołków to wyrażenie.

## Argument techniczny, który rozstrzygnął sprawę bez sporu

Warto go mieć, bo działa lepiej niż wykład o prawie: **z renderów podglądowych i tak
nie da się odtworzyć dokładnych pozycji wierzchołków.** Rendery są w perspektywie,
o nieznanej ogniskowej i odległości kamery, bez skali odniesienia, w stratnym JPG,
a wierzchołki zasłonięte (tył głowy, wnętrze dłoni, podeszwa) w ogóle nie są widoczne.

Powstaje zbieżność, którą łatwo pokazać:

> Te 99%, o których mówisz, są **jednocześnie nieosiągalne z tych obrazków
> i jedynym wariantem, który byłby problemem prawnym.**

Realnie z podglądów da się odtworzyć 80-90% sylwetki - co jest dokładnie tym
poziomem, który mieści się w "inspiracji".

## Decyzja

Reżyser wybrał drogę uczenia się z podglądów bez kupowania. To jest legalne i to
jest wykonalne. Zapisano granicę w dokumencie przekazania, żeby kolejna sesja
(albo agent) nie przekroczyła jej przez przypadek, celując w "jak najwierniej".

## Wniosek przenośny

Gdy pojawia się prośba o odtworzenie cudzego assetu:
1. Powiedz raz, konkretnie, gdzie leży granica - bez moralizowania i bez powtarzania.
2. Sprawdź, czy istnieje droga czysta i tania (tu: 3 dolary z licencją komercyjną).
   Często jest tańsza niż godzina pracy nad rekonstrukcją.
3. **Podaj argument techniczny obok prawnego.** "Tego i tak nie da się odtworzyć
   z tych plików" przekonuje szybciej niż wywód o utworze zależnym.
4. Zapisz ustaloną granicę w dokumencie projektu, nie tylko w rozmowie - inaczej
   następna sesja jej nie zna.
