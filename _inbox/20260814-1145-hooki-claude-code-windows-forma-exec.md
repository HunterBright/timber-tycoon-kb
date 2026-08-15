---
title: Hooki Claude Code na Windows - forma powloki z Git Bashem cicho zabija hook, uzywaj formy EXEC (args)
type: anti-pattern
status: draft
confidence: high
verified: '2026-08-14'
date: 2026-08-14
project: AnotherQuest
tags:
- claude-code
- hooks
- windows
- git-bash
- msys
- fail-open
applies_to:
- claude-code-2.1.x
- windows
source: 'claude.exe 2.1.229, offset ~287882142 (funkcja spawnowania hooka)'
suggested-category: workflow/anti-patterns
---

# Hook typu `command` w formie powloki na Windows z Git Bashem

## The trap

W `settings.json` wpisujesz hook w formie powloki, po windowsowemu:

    { "type": "command",
      "command": "cmd /c \"%CLAUDE_PROJECT_DIR%/.claude/hooks/moj-hook.cmd\"" }

Wyglada poprawnie. Plik istnieje, sciezka sie zgadza, logika skryptu jest przetestowana osobno
i dziala. Hook jest **martwy** i nic o tym nie mowi.

## Why it fails

Claude Code na Windows wybiera powloke dla formy `command` (bez `args`) tak: **jesli Git Bash jest
zainstalowany, uzywa bash.exe**. PowerShell jest fallbackiem dopiero, gdy Git Basha nie ma.
Wywolanie wyglada wiec tak:

    C:\Program Files\Git\bin\bash.exe -c 'cmd /c "..."'

Warstwa MSYS w Git Bashu przepisuje argumenty wygladajace jak sciezki uniksowe: **`/c` staje sie
`C:/`**. `cmd` nie dostaje wiec swojego przelacznika ani polecenia - startuje **interaktywnie**,
zjada payload JSON ze stdin jako polecenie do wykonania, wypisuje swoj baner i konczy sie
**kodem 0**.

I tu zamyka sie pulapka: dla `PreToolUse` **kod 0 plus wyjscie nie-JSON znaczy "hook nie ma zdania"**,
czyli narzedzie **przechodzi**. Bezpiecznik, ktory mial blokowac, zamienil sie w bezszelestna
przepustke.

## Symptoms

- Hook jest w `settings.json`, skrypt istnieje, a mimo to nic nie blokuje i nic nie loguje.
- W wyjsciu hooka (jesli zajrzysz) widac baner `Microsoft Windows [Version ...]`
  i `'{"session_id":' is not recognized as an internal or external command`.
- Skrypt uruchomiony recznie z payloadem na stdin dziala **bezblednie** - bo recznie
  nie odpalasz go przez `bash.exe -c`.
- Kod wyjscia 0 tam, gdzie spodziewasz sie 2.

## Correct approach

Uzyj **formy EXEC**: pole `args` obok `command`. Gdy `args` jest obecne, Claude Code rozwiazuje
`command` jako plik wykonywalny i spawnuje go **bezposrednio, bez zadnej powloki** - zaden cmd,
zaden MSYS, zadne mielenie sciezek:

    { "type": "command",
      "command": "powershell",
      "args": ["-NoProfile", "-ExecutionPolicy", "Bypass", "-File",
               "${CLAUDE_PROJECT_DIR}/.claude/hooks/moj-hook.ps1"],
      "timeout": 15 }

`${CLAUDE_PROJECT_DIR}` **jest** podstawiany per element `args` (potwierdzone w kodzie binarki, nie
w dokumentacji - opis schematu wymienia tylko `${CLAUDE_PLUGIN_ROOT}`, ale implementacja podstawia
takze `CLAUDE_PROJECT_DIR`, `CLAUDE_PLUGIN_DATA` i `${user_config.*}`). Wartosc idzie jako zwykly
tekst, wiec spacje i znaki specjalne w sciezce nie maja przez co przejsc.

Warianty dzialajace, ale gorsze:
- `cmd //c "..."` - podwojny ukosnik chroni przed MSYS, ale nadal przechodzi przez dwie powloki;
- `"shell": "powershell"` - omija Git Basha, ale zostawia parser powloki w torze.

**Zasada ogolna:** jesli hook ma byc bezpiecznikiem, nie wpuszczaj powloki miedzy Claude Code
a skrypt. Kazda powloka po drodze to miejsce, w ktorym blad zamienia sie w ciche `exit 0`.

## Jak to sprawdzic, zanim uwierzysz

Odtworz **prawdziwa** sciezke wywolania (spawn z payloadem JSON na stdin), a nie recznie w terminalu.
I zawsze dolacz **kontrole negatywna** - plik, ktory ma przejsc. Bez niej test niczego nie dowodzi,
bo skrypt blokujacy wszystko przeszedlby probe pozytywna tak samo.
