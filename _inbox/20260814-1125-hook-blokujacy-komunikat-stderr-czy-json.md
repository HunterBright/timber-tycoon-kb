---
title: Hook blokujacy - exit 2 blokuje, ale komunikat idzie przez stderr, nie przez JSON na stdout
type: lesson
status: draft
confidence: high
verified: '2026-08-14 (test na zywym hooku, D:\Unity\AnotherQuest)'
date: 2026-08-14
project: Another Quest
tags:
- claude-code
- hooks
- powershell
- windows
applies_to: []
source: '.claude/hooks/block-scene-text-edit.ps1'
severity: medium
suggested-category: tooling/lessons
time_lost: ''
---

# Hook blokujacy - exit 2 blokuje, ale komunikat idzie przez stderr, nie przez JSON na stdout

## Problem

Hook PreToolUse przeniesiony z jednego projektu do drugiego wypisywal poprawny JSON
z `permissionDecision: "deny"` i konczyl sie kodem 2. Blokada dzialala. Pytanie, ktore sie nie
zadalo przez pol roku: **czy Hunter (i model) widza POWOD blokady**, czy tylko sam fakt odbicia.

To nie jest kosmetyka. Jesli test trybu porazki brzmi "zapis do .unity ma sie odbic z komunikatem
hooka", a komunikat nie dociera, to test przechodzi na oko i nie dowodzi niczego o atrybucji.

## Root cause

Claude Code ma **dwie rozlaczne drogi** odpowiedzi hooka:

1. **JSON na stdout + exit 0** - nowa droga decyzyjna; pole `permissionDecisionReason` jest
   czytane i pokazywane.
2. **exit 2** - stara droga; wywolanie jest zablokowane, a do modelu wraca **stderr**.
   Stdout przy kodzie 2 nie musi byc parsowany jako decyzja.

Hook pisal JSON na stdout i konczyl kodem 2 - czyli mieszal drogi. Blokada byla pewna (kod 2),
ale powod jechal kanalem, ktorego przy tym kodzie nikt nie czyta.

## Solution

Nie wybieraj drogi - obsluz obie, bo nic to nie kosztuje:

```powershell
$powod = "BLOKADA (hook <nazwa>): <co i dlaczego> - legalna droga: <co zrobic zamiast>"
@{ hookSpecificOutput = @{ hookEventName = "PreToolUse"
   permissionDecision = "deny"; permissionDecisionReason = $powod }} | ConvertTo-Json -Compress
[Console]::Error.WriteLine($powod)
exit 2
```

Test na sucho, bez czekania na prawdziwa sytuacje:

```powershell
'{"tool_input":{"file_path":"Assets/Foo.prefab"}}' | & cmd /c ".claude\hooks\<hook>.cmd"
```

Sprawdzasz trzy rzeczy: JSON na stdout, tekst na stderr, kod wyjscia 2. I przypadek negatywny
(plik, ktory ma przejsc) - cisza i kod 0.

Druga polowa lekcji: **komunikat hooka ma nazywac hook**. "Scene files are binary" nie mowi,
co zablokowalo. "BLOKADA (hook block-scene-text-edit)" pozwala odroznic blokade hooka od
blokady `permissions.deny`, gdy obie pokrywaja ten sam plik - a hook odpala pierwszy.

## What didn't work

Zalozenie "dziala od pol roku w drugim projekcie, wiec jest dobre". Dzialala blokada; nikt nie
sprawdzil drugiej polowy kontraktu.

## Transferability

Dotyczy kazdego hooka blokujacego w Claude Code, na dowolnym systemie i w dowolnym projekcie.
Ogolniejsza zasada: przenoszac zabezpieczenie miedzy projektami, przetestuj obie polowy jego
kontraktu - ze zatrzymuje ORAZ ze mowi dlaczego.

## Related
- Bramka bez udowodnionego trybu porazki niczego nie pilnuje - obie polowy: ma oblac i nazwac winowajce
