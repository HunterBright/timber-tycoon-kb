---
type: pattern
project: Kerf - Sawmill Tycoon
suggested-category: engine/patterns
tags: [unity, gpu-instancing, baked-data, binary-blob, gate, scatter]
date: 2026-08-05
status: draft
---

# Chirurgia binarnych blobów instancji zamiast re-rozsiewu

## Problem

Flora/dekor rozsiane proceduralnie do binarnych blobów (GPU instancing, zero
GameObjectów). Trzeba usunąć/przenieść KONKRETNE instancje (głaz przy rampie),
ale narzędzie rozsiewu jest monolityczne i seedowane strumieniowo: odrzucenia
konsumują RNG, więc po każdej zmianie sceny re-run przelosowuje CAŁĄ mapę
i cofa wcześniejsze ręczne poprawki.

## Wzorzec

Osobne narzędzie "chirurgiczne" na blobie, nigdy re-scatter:

1. **Parser/writer lustrzany do RUNTIME'OWEGO readera** (nie do writera scattera,
   który przelicza bounds). Bramka wejściowa: `Write(Parse(bytes)) == bytes`
   co do bajta - dopiero wtedy jakakolwiek matematyka na blobie jest wiarygodna.
2. **Usuwanie jest tanie**: bounds komórek to nadzbiór używany tylko do cullingu,
   więc po usunięciu instancji zostają NIETKNIĘTE (puste komórki też zostają) -
   delta bajtowa jest wtedy dokładnie `-N × rozmiar_instancji` i działa jako bramka.
   Dodawanie wymaga przeliczenia AABB dotkniętej komórki wzorem writera.
3. **Dopasowanie sterowane danymi** (JSON: etykieta + XZ + tolerancja) z bramkami:
   dokładnie 1 instancja w tolerancji, druga najbliższa > 2×tol, zero podwójnych
   roszczeń, liczba grup blobu == liczba grup renderera (kolejność POZYCYJNA!).
   Jedno naruszenie = odmowa całości (zero częściowych zapisów).
4. **Self-test z dowodami odmowy**: celowo zły wpis (x=9999) i celowo
   wieloznaczny (tol=500) muszą dać ODMOWĘ; zepsuty nagłówek musi rzucić.
5. Po zapisie: ImportAsset + Rebuild renderera + sonda liczników (instancje,
   collidery-towarzysze) przed/po.

## Dlaczego to działa

Bloby są małe (setki KB), w zwykłym gicie (nie LFS) - diff po rozmiarze,
rollback jednym checkoutem. "Przed" do zdjęć porównawczych odzyskuje się
podmieniając plik blobu na backup + Rebuild (renderer czyta TextAsset,
scena nietknięta).

## Anty-wzorzec

Re-run seedowanego scattera "bo seed ten sam" - strumień RNG konsumowany przez
ODRZUCENIA zależy od całej geometrii sceny; identyczny seed ≠ identyczny układ.

## Kontekst źródłowy

Kerf 2026-08-05: usunięcie 8 kamieni z DecorInstanced.bytes (2550→2542,
-320 B co do bajta), narzędzie `Assets/Editor/FloraSurgery.cs`; precedensy
ręczne dc83be2 i 5bb09d5.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260710-2140-flora-scatter-exclusion-zones|Proceduralny rozsiew dekoracji musi wykluczac pozycje obiektow interaktywnych]] - wspolne: gpu-instancing, scatter
<!-- /POWIAZANE:auto -->
