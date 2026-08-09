---
title: Edytora nie da sie oszukac, zeby udawal build
type: lesson
status: verified
confidence: high
verified: '2026-07-30'
date: '2026-07-30'
project: Kerf - Sawmill Tycoon
tags:
- unity
- build
- qa
- sonda
- weryfikacja
- metodologia
applies_to:
- unity-projects
source: konsolidacja 7 odnosnikow do nieistniejacego wpisu podczas migracji do Obsidiana
severity: high
---

# Edytora nie da sie oszukac, zeby udawal build

Powstal z tego samego powodu co [[gate-must-have-provable-failure-mode]]:
siedem odnosnikow pod czterema nazwami (`editor-cannot-simulate-build`,
`build-only-bugs-editor-lies`, `measure-in-build-not-in-simulation`,
`weryfikacja-w-buildzie-edytor-klamie`) i zadnej notatki.

## Objaw

Kod dziala w Edytorze i pada w buildzie - albo, co gorsze, cicho umiera:
nic nie wybucha, ale funkcja po prostu nie dziala. Gra przez rok nie byla budowana
i uzbierala cala klase takich bledow. Trzy blokery wyszly w ciagu **godziny**
od pierwszego builda, a czwarty dopiero wtedy, gdy Hunter w tym buildzie zagral.

## Przyczyna

Edytor ma zasoby, ktorych build nie ma, i toleruje rzeczy, ktorych build nie toleruje:

- **Siatki z wylaczonym Read/Write** sa w Edytorze czytelne z pamieci importera,
  w buildzie nie. `AddComponent<MeshCollider>` w trakcie gry albo `mesh.vertices`
  daja tam pustke lub crash natywny.
- **`Shader.Find` i `GameObject.CreatePrimitive`** znajduja shader w Edytorze,
  bo tam jest caly katalog shaderow. W buildzie shader musi byc dociagniety
  przez uzycie w scenie albo liste "Always Included" - inaczej magenta.
- **Klasa MonoBehaviour w pliku o innej nazwie** kompiluje sie w Edytorze,
  a w buildzie scena sie nie wczyta.
- **`UnityEditor.*` w kodzie runtime** - Edytor to lyka, player sie nie skompiluje.
- **`Application.isEditor` w tescie** - test bramkuje sam siebie i nigdy nie odpali
  sie tam, gdzie mial cos udowodnic.
- **Flaga `isReadable`** nie dowodzi niczego. Trzeba sprawdzic **zachowanie**:
  czy promien faktycznie trafia w collider.

## Rozwiazanie

Zbuduj. Nie szukaj obejscia.

```powershell
# Unity MUSI byc zamkniete
& "D:\Unity\6000.5.1f1\Editor\Unity.exe" -batchmode -quit -projectPath "D:\Unity\Timber_Tycoon" `
   -executeMethod BuildTimberTycoon.BuildWindows -logFile "_Handoff\build_game.log"

.\Builds\Win64\Kerf.exe -meshaudit -logFile "_Handoff\player.log"
echo $LASTEXITCODE          # 0 = OK, 1 = jest FAIL
type .\Builds\Win64\MeshAudit_Result.txt
```

Zasada podzialu: zmiany z rodziny "Edytor klamie" (materialy tworzone w kodzie,
collidery i odczyt siatek w trakcie gry, komponenty dokladane skryptami,
API edytora w runtime, kazda zmiana sceny) buduj **natychmiast**, pojedynczo.
Reszte - wartosci w ScriptableObjectach, teksty, balans, UI - zbiorczo,
przy bramce przed oddaniem pracy.

## Czego nie robic

- Nie pisz "dziala" na podstawie Edytora. To twierdzenie o Edytorze, nie o grze.
- Nie ufaj staremu raportowi sondy. Przerwana sonda **zostawia poprzedni plik wyniku** -
  sprawdz date modyfikacji, zanim uznasz go za dowod.
- Nie zakladaj, ze `-batchmode` zakonczyl build, bo polecenie wrocilo.
  PowerShell nie czeka na Unity domyslnie.

## Dowod z tego projektu

- `CLAUDE.md`, sekcja "Weryfikacja w buildzie (od 2026-07-13)" - opis czterech blokerow
  i trzech bramek.
- `_Handoff/MESH_READABLE_AUDIT.md` - pelny raport i tabela przypadkow.
- Sonda `MeshReadabilityProbe` sprawdza w prawdziwym buildzie: collidery tworzone
  w runtime, integralnosc sceny, magente, obecnosc shaderow szukanych przez kod.

## Powiazane

- [[gate-must-have-provable-failure-mode]]
- [[20260713-1900-build-early-never-built-project-hides-editor-only-bugs]]
- [[20260713-1830-runtime-meshcollider-needs-readwrite-and-editor-cannot-prove-it]]
- [[20260713-1845-monobehaviour-class-must-match-filename]]
- [[20260713-2130-shader-find-null-and-createprimitive-magenta-in-build]]
- [[20260714-2320-if-unity-editor-fixes-the-build-and-kills-the-game]]
- [[20260714-2245-unity-batchmode-returns-before-build-finishes]]
- [[20260713-1030-verify-in-target-engine-not-source-tool]]
- [[20260718-0805-headless-visual-proof-batchmode]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[gate-must-have-provable-failure-mode|Bramka bez udowodnionego trybu porazki niczego nie pilnuje]] - wspolne: metodologia, qa, sonda
- [[20260731-0025-sedzia-na-artefaktach-lapie-bledy-wlasnego-narzedzia|Sedzia oceniajacy artefakty lapie bledy w narzedziu, ktore te artefakty produkuje]] - wspolne: metodologia, qa
- [[20260731-2200-slepa-dzwignia-debugger-bramek|20260731-2200-slepa-dzwignia-debugger-bramek]] - wspolne: qa, sonda
<!-- /POWIAZANE:auto -->
