---
title: In-place load osieroca obiekty-satelity (dziura po wczytaniu stała na dorosłym drzewie)
type: anti-pattern
status: draft
confidence: low
verified: ''
date: '2026-07-16'
project: Kerf - Sawmill Tycoon
tags:
- unity
- save-load
- in-place-load
- ownership
- orphans
- reconciliation
applies_to: []
source: ''
suggested-category: engine/anti-patterns
---

# In-place load osieroca obiekty-satelity (dziura po wczytaniu stała na dorosłym drzewie)

## Anty-wzorzec
System zapisu, w którym wczytanie NA ŻYWĄ SCENĘ (in-place, bez przeładowania sceny) tylko
DOKŁADA stan z pliku ("spawnuj jeśli flaga true"), a nigdy nie UZGADNIA go z żywą sceną.
Obiekty-satelity (spawnowane przez właściciela jako ROOT bez rodzica, nieśledzone przez
rejestr trwałości) przeżywają wczytanie jako sieroty, gdy:
- (a) load jest asymetryczny: flaga=false w zapisie NIE niszczy żywej instancji,
- (b) właściciel ginie w passie czyszczenia rejestru, ale satelita nie jest na żadnej liście,
- (c) obiekt przetwarzany PODWÓJNIE (żywa instancja w main-pass + destroy/respawn z rejestru),
- (d) po przejściu cyklu życia (dziura -> drzewo) nikt nie odpowiada za usunięcie starego stanu.
Objaw klasyczny: bug występuje TYLKO przy wczytaniu w trakcie gry, a restart + load jest czysty
(fresh boot nie ma żywej sceny do uzgodnienia).

## Wzorzec naprawy (4 warstwy, wszystkie naraz)
1. SYMETRIA: load z flagą=false niszczy żywą instancję i zeruje stan (nie tylko spawn przy true).
2. WŁASNOŚĆ: `OnDestroy()` właściciela niszczy satelitę (umierający właściciel zabiera swoje).
3. SWEEP: system ładujący się NAJWCZEŚNIEJ (rejestracja w Awake vs Start = kolejność w save
   registry) niszczy WSZYSTKIE żywe satelity na początku swojego LoadSaveData - każdą legalną
   odtworzy jej właściciel później w tym samym wczytaniu. Sweep PRZED early-returnem na pustym
   zapisie (pusty save też musi czyścić scenę).
4. GUARD stale-save: spawn satelity odmawia, gdy w miejscu doszło do konfliktu cyklu życia
   (np. dziura pod stojącym drzewem) - stare zapisy sprzed naprawy nie mogą odtworzyć bugu.
Plus: jeśli satelita ma własny stan przejściowy (minigra z odczepioną kamerą, kursor), przed
zniszczeniem w sweepie wołaj synchroniczne `ForceCancelForLoad()` (przywróć kamerę/kursor).

**Why:** in-place load to inna klasa problemów niż fresh boot - każdy obiekt spawnowany w
runtime musi mieć odpowiedź na pytanie "kto mnie sprząta przy wczytaniu".
**How to apply:** przy projektowaniu ISaveable dla obiektu spawnującego satelity od razu
napisz test in-place double-load i wymuś 4 warstwy; testuj ZAWSZE scenariusz "wczytaj w
trakcie gry", nie tylko "uruchom i wczytaj".
