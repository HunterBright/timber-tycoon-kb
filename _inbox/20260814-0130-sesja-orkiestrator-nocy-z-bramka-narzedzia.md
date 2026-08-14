---
type: pattern
project: Another Quest
suggested-category: process/patterns
tags: [praca-nocna, orkiestracja, subagenci, mcp, tryb-porazki, kontekst]
date: 2026-08-14
status: draft
---

# Sesja-orkiestrator nocy: jeden job = jeden subagent + bramka narzędzia na pierwszym kroku

## Problem

Praca nocna bezobsługowa (dyrektor śpi, nikt nie startuje sesji ręcznie) w KERF miała jedną sesję na noc.
Jedna sesja na 5–6 zadań degraduje kontekst i wiąże los wszystkich zadań ze sobą: awaria pierwszego
zadania zabija noc, a zadanie ostatnie pracuje na wyczerpanym kontekście.

## Wzorzec

**Jedna sesja-orkiestrator, każdy job jako subagent ze świeżym kontekstem, uruchamiane w wiążącej kolejności.**
Trzy elementy, które sprawiają, że to działa:

1. **Zakres jobu w JEDNYM miejscu (karta jobu).** Lista startowa niesie WYŁĄCZNIE kolejność, nigdy zakresy —
   inaczej lista i karta rozjeżdżają się przy każdej rewizji planu, a wykonawca czyta tę, która jest bliżej.
   Prompt subagenta wskazuje kartę po numerach linii i mówi: "autorytet ma karta, ten prompt to mapa".
2. **Stop dotyczy WYŁĄCZNIE następników zależnych.** Job niezależny o najdłuższym ogonie zegarowym startuje
   PIERWSZY i biegnie równolegle; jego porażka nie rusza reszty nocy.
3. **Bramka narzędzia jako pierwszy krok jobu zależnego od kanału zewnętrznego (MCP, sieć, licencja).**
   Job sprawdza dostępność narzędzia ZANIM cokolwiek utworzy, i ma nazwany tryb porażki:
   STOP + wpis do raportu porannego + jawny zakaz improwizowania innego kanału.

## Dlaczego bramka jest tym elementem, który się zwraca

W noc D1 AQ kanał generatora 3D (unity-mcp) nie wystawił narzędzi mimo poprawnych uprawnień i otwartego
edytora. Bez bramki job spaliłby noc na próbach obejścia — a obejścia BYŁY dostępne (Hunyuan przez Blender MCP,
coplay-mcp). Każde z nich dałoby asset spoza zatwierdzonego pipeline'u, czyli spike zweryfikowałby ścieżkę,
której projekt nie zamierza używać. Bramka zamieniła "spalona noc + fałszywy dowód" w "5-punktowa lista
naprawcza na rano + retry od zera".

**Zakaz improwizacji kanału musi być w prompcie jawny i wyliczać kanały zakazane po nazwie.** Subagent, który
widzi na liście narzędzie robiące "to samo", użyje go w dobrej wierze.

## Warunki brzegowe

- **Joby równoległe w jednym drzewie git**: każdy commituje WYŁĄCZNIE swoim pathspec, nigdy `git add -A`;
  retry na `.git/index.lock` (5 s × 5), potem praca zostaje na dysku.
- **Raport poranny per job, nie per noc.** Joby równoległe piszą fragmenty osobno (kolizja przy jednym pliku),
  joby sekwencyjne dopisują sekcje; orkiestrator scala na końcu.
- **Zmiany konfiguracji wymagające restartu sesji są poza zasięgiem nocy** — restart zabija orkiestratora.
  Trafiają na listę poranną, nawet gdy diagnoza jest pewna.
- Orkiestrator nie powinien czytać transkryptów subagentów (przepełnienie kontekstu) — tylko ich raporty
  zwrotne. To znaczy, że **raport zwrotny jest kontraktem** i trzeba go zamówić w prompcie strukturalnie.

## Zmierzone (noc D1 AQ, 2026-08-13/14)

6 jobów: 4 zamknięte w całości, 1 STOP na własnej bramce, 1 pominięty z braku wejścia. Zero kolizji commitów,
zero utraconej pracy, 13 commitów wypchniętych. Trzy joby złapały i zgłosiły własne błędy metodyczne — świeży
kontekst per job wydaje się to ułatwiać (job nie broni decyzji, której sam wcześniej nie podjął).

Powiązane: [[context-degradation]], [[gate-must-have-provable-failure-mode]], [[generator-nie-jest-evaluatorem]]
