---
title: permissions.deny - regula Edit() pokrywa Write i MultiEdit, wyliczanie czasownikow szkodzi
type: lesson
status: draft
confidence: high
verified: '2026-08-14 (dokumentacja code.claude.com/docs/en/permissions)'
date: 2026-08-14
project: Another Quest
tags:
- claude-code
- permissions
- konfiguracja
- ochrona-plikow
applies_to: []
source: 'D:\Unity\AnotherQuest\.claude\settings.json (commit 11ecffe)'
severity: medium
suggested-category: tooling/lessons
time_lost: ''
---

# permissions.deny - regula Edit() pokrywa Write i MultiEdit, wyliczanie czasownikow szkodzi

## Problem

Przy budowie ochrony plikow kanonu zalozeniem planu bylo: "deny pokrywa JEDEN czasownik na wpis,
nie licz na wildcard" - czyli kazda chroniona sciezka razy trzy wpisy (`Edit(...)`, `Write(...)`,
`MultiEdit(...)`). Zalozenie wyglada bezpiecznie ("lepiej wiecej niz mniej"), a jest bledne
i ma koszt.

## Root cause

W Claude Code reguly sciezkowe sa **grupowane po rodzinie narzedzi**, nie po nazwie narzedzia:

- `Edit(sciezka)` pokrywa **Edit, Write i MultiEdit** naraz.
- `Read(sciezka)` pokrywa Read, a w deny blokuje takze Edit i Write na tej samej sciezce.
- Wpisy `Write(...)` / `MultiEdit(...)` / `Glob(...)` ze sciezka nie sa "dodatkowa warstwa" -
  Claude Code **ostrzega przy starcie**, ze taka regula jest napisana zle, i kaze uzyc Edit/Read.

Czyli mnozenie czasownikow nie zwieksza ochrony, a dokłada ostrzezenie do kazdego startu sesji.
Ostrzezenie, ktore widzi sie codziennie i ktore "zawsze tam jest", przestaje byc czytane -
i przykryje prawdziwe ostrzezenie, gdy takie przyjdzie.

## Solution

Jeden wpis `Edit(sciezka)` na chroniona sciezke. Do tego dwa zakotwiczenia, bo semantyka sciezki
zalezy od miejsca, w ktorym regula mieszka:

- `Edit(docs/canon/*_LOCKED.md)` - wzgledem katalogu biezacego,
- `Edit(/docs/canon/*_LOCKED.md)` - wzgledem katalogu pliku ustawien (dla `.claude/settings.json`
  to korzen repo),
- `//` = korzen systemu plikow, `~` = katalog domowy.

W `settings.local.json` wzorce kotwicza sie na PIERWOTNYM cwd, nie na korzeniu repo - to celowe,
dla worktree.

Globy sa w stylu gitignore: `*` nie przechodzi przez `/`, `**` przechodzi.
Dla MCP dzialaja obie formy: `mcp__serwer` (usuwa serwer z kontekstu) i `mcp__serwer__*`
(wszystkie narzedzia serwera). W **allow** goły glob `mcp__*` jest odrzucany, w deny dziala.

`permissions.deny` **nie jest omijane przez `--permission-mode bypassPermissions`** - bypass
pomija tylko PYTANIA o zgode. To jest wlasnie powod, dla ktorego deny jest lepszym nosnikiem
ochrony niz hook z wlasna logika.

## What didn't work

Wyliczanie czasownikow "na wszelki wypadek" - kosztuje ostrzezenie na kazdym starcie i niczego
nie dokłada. Osobno: poleganie na jednym zakotwiczeniu sciezki - awaria formy w deny jest
**cicha** (nic sie nie dzieje, zapis przechodzi), w allow jest widoczna (pyta o zgode).
Dlatego reguly deny wymagaja testu trybu porazki, a nie samego przeczytania.

## Transferability

Dotyczy kazdego projektu z Claude Code, w ktorym cokolwiek ma byc chronione przed zapisem:
pliki kanonu, stan maszynowy, sekrety, sceny silnika. Niezalezne od silnika i gatunku gry.

## Related
- Bramka bez udowodnionego trybu porazki niczego nie pilnuje (reguła w active-rules)
