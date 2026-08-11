---
type: anti-pattern
project: Kerf - Sawmill Tycoon (Discord MGDB Studio)
suggested-category: tooling/anti-patterns
tags: [powershell, npx, npm, windows, mcp, scoped-package, instrukcje-dla-uzytkownika]
date: 2026-08-10
status: draft
---

# Komenda `npx @scope/pakiet` skopiowana z dokumentacji nie działa w PowerShellu

## Objaw

```
PS> npx @quadslab.io/discord-mcp init
The splatting operator '@' cannot be used to reference variables in an expression.
'@quadslab' can be used only as an argument to a command.
    + FullyQualifiedErrorId : SplattingNotPermitted
```

Wygląda jak problem z paczką albo z npm. Nie jest. Paczka jest w porządku.

## Przyczyna

W PowerShellu `@` na początku słowa to **operator splattingu** (rozwijanie tablicy albo
tablicy asocjacyjnej na argumenty). Parser łapie go zanim komenda w ogóle ruszy, więc
błąd jest składniowy, nie wykonawczy. Dokumentacja praktycznie każdej paczki npm podaje
przykłady w składni POSIX, gdzie `@` nie znaczy nic szczególnego.

## Poprawnie

```powershell
npx --yes "@scope/pakiet" argument
```

Cudzysłów wystarczy. Alternatywnie token zatrzymania parsowania: `npx --% @scope/pakiet init`.

## Dlaczego to boli bardziej, niż wygląda

Sam błąd naprawia się w sekundę. Koszt siedzi gdzie indziej: **to jest komenda, którą
wkleiłem do instrukcji dla nieprogramisty.** Osoba, która ją dostała, nie ma jak odróżnić
"zła składnia powłoki" od "narzędzie nie działa" i przy odrobinie pecha rezygnuje z całego
kroku albo zaczyna szukać nie tam.

## Reguła

Zanim wkleisz do instrukcji dla użytkownika jakąkolwiek komendę przepisaną z dokumentacji
projektu, **uruchom ją u siebie w tej samej powłoce, w której użytkownik ma ją uruchomić.**
Nie chodzi o sprawdzenie efektu, tylko o sprawdzenie, czy powłoka ją w ogóle przyjmie.
Wersja bezpieczna dla samego sprawdzenia: podmień właściwą komendę na `--help` albo `version`.

Dotyczy w szczególności: `@scope/...` (npx, npm), `$zmienna` w cudzysłowie podwójnym,
`&&` i `||` (nie istnieją w Windows PowerShell 5.1), `2>/dev/null`, apostrofy w argumentach.

Powiązane: [[feedback-powershell-outfile-katalog]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260719-1210-unity-build-freshness-check-dll-not-exe|Świeżość builda Unity sprawdzaj po DLL z kodem gry, nie po .exe]] - wspolne: windows, powershell
<!-- /POWIAZANE:auto -->
