---
title: Licencja repozytorium GitHub to nie jest licencja wag na Hugging Face
type: lesson
status: verified
confidence: high
verified: '2026-08-05'
date: 2026-08-02
project: GameDevOS
tags:
- licencje
- hugging-face
- github
- ai-3d
- due-diligence
applies_to: []
source: 'https://huggingface.co/api/models/nvidia/Kimodo-SOMA-RP-v1.1 , https://api.github.com/repos/czpcf/TopoCap/license , https://huggingface.co/api/models/tencent/Hunyuan3D-2.1'
severity: high
suggested-category: process/lessons
time_lost: ''
---

# Licencja repozytorium GitHub to nie jest licencja wag na Hugging Face

## Problem
Przy sprawdzaniu, czy dany model AI wolno uzyc komercyjnie, latwo przeczytac plik LICENSE
w repozytorium GitHub, zobaczyc "Apache-2.0" albo "MIT" i uznac sprawe za zamknieta.
To jest za malo. Trzy realne przypadki z jednego dnia zwiadu:

1. **nv-tlabs/Kimodo** - kod na GitHubie ma czysty Apache-2.0. Ale wagi na Hugging Face
   (`nvidia/Kimodo-SOMA-RP-v1.1`, `nvidia/Kimodo-SOMA-SEED-v1`) maja
   `license: other`, `license_name: nvidia-open-model-license`. Inny dokument, inne warunki.
2. **czpcf/TopoCap** - README deklaruje "This project is licensed under the MIT License"
   i linkuje do pliku `LICENSE`, ktorego w repozytorium NIE MA (HTTP 404).
   Deklaracja bez pliku to prawnie slaba podstawa; domyslnie kod bez licencji
   jest "wszystkie prawa zastrzezone".
3. **tencent/Hunyuan3D-2.1** - plik LICENSE na GitHubie zawiera wykluczenie UE,
   ale osobno karta modelu na Hugging Face ma flage `extra_gated_eu_disallowed: true`,
   ktora fizycznie blokuje pobieranie z UE. Dwa niezalezne mechanizmy, oba trzeba sprawdzic.

## Root cause
Kod i wagi to dwa rozne przedmioty prawne, czesto wydawane przez rozne podmioty
(uczelnia wydaje kod, firma wydaje wagi) i pod roznymi dokumentami.
Do tego Hugging Face ma wlasna warstwe metadanych (`license`, `license_name`,
`gated`, `extra_gated_eu_disallowed`), ktora nie jest widoczna w interfejsie
i nie odpowiada zawartosci pliku LICENSE w repo.

## Solution
Checklista minimum przy kazdym modelu AI, ktory ma dotknac projektu komercyjnego:

1. `curl https://raw.githubusercontent.com/<org>/<repo>/main/LICENSE` - czytaj plik,
   nie streszczenie. Sprawdz, czy plik w ogole istnieje (kod odpowiedzi 200 vs 404).
2. `curl https://api.github.com/repos/<org>/<repo>/license` - pole `spdx_id`.
   Wartosc `NOASSERTION` znaczy "GitHub nie rozpoznal licencji", czyli czerwona flaga.
3. `curl https://huggingface.co/api/models/<org>/<model>` - sprawdz w JSON:
   `cardData.license`, `cardData.license_name`, `cardData.license_link`, `gated`
   oraz wszystkie klucze zawierajace `gated` (szczegolnie `extra_gated_eu_disallowed`).
4. Jesli `license_name` wskazuje wlasny dokument firmy, pobierz i przeczytaj ten dokument.
   Szukaj slow: `Territory`, `European Union`, `excluding`, `monthly active users`, `revenue`.
5. `curl https://api.github.com/repos/<org>/<repo>/commits?path=LICENSE` - historia zmian
   pliku licencji. Pokazuje, czy warunki byly po cichu zmieniane.

## What didn't work
Poleganie na wyszukiwarce i streszczeniach. Wyszukiwarka zwraca artykuly typu
"Model X jest open source", ktore mieszaja licencje kodu z licencja wag
i nie widza flag regionalnych Hugging Face.

## Transferability
Dotyczy kazdego projektu, ktory wciaga model AI do pipeline'u produkcyjnego:
generacja 3D, rigowanie, tekstury, audio, LLM. Niezalezne od silnika i gatunku gry.
Licencja jest jedyna rzecza, ktora potrafi uniewaznic cala prace wstecz,
wiec koszt tej checklisty (5 zapytan curl) jest nieporownywalnie nizszy niz ryzyko.

## Related
- _inbox/20260801-1245-regulamin-uslugi-a-licencja-wag-to-dwa-rozne-swiaty.md

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260801-0825-licencja-marki-nie-jest-licencja-produktu|Licencja marki nie jest licencja produktu]] - wspolne: due-diligence, licencje
<!-- /POWIAZANE:auto -->
