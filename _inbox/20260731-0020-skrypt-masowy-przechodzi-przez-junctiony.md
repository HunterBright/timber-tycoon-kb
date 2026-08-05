---
title: Skrypt masowy puszczony na vaulcie z junctionami siega poza zamierzony zakres
type: anti-pattern
status: verified
confidence: high
verified: '2026-08-05'
date: '2026-07-31'
project: Kerf - Sawmill Tycoon
tags:
- workflow
- skrypty
- windows
- junction
- zakres-zmian
source: 'https://github.com/python/cpython/blob/main/Python/fileutils.c (wpadka podczas migracji bazy wiedzy do Obsidiana 2026-07-31)'
severity: medium
---

# Skrypt masowy puszczony na vaulcie z junctionami siega poza zamierzony zakres

## Pulapka

Budujesz vault (albo dowolny katalog zbiorczy), w ktorym inne lokalizacje sa podpiete
skrotami systemowymi Windows (junction), bo chcesz jedna kopie pliku widoczna w dwoch miejscach.
Potem puszczasz na tym vaulcie skrypt masowy: zamiane znakow, normalizacje, przenoszenie.

Wydaje ci sie, ze operujesz na swoim katalogu.

## Dlaczego zawodzi

**`os.walk` w Pythonie przechodzi przez junctiony i wchodzi w nie jak w zwykle katalogi.**
Skrypt dociera wiec do wszystkiego, co jest podpiete: do repozytorium gry, do pamieci narzedzia,
do dokumentacji, ktorej nie mial dotykac.

Mylace jest to, ze **`find` z Gita Bash zachowuje sie odwrotnie** - bez `-L` NIE wchodzi
w junctiony i pokazuje zero plikow. Latwo wiec sprawdzic zasieg jednym narzedziem,
zobaczyc "pusto" i uznac, ze drugie narzedzie zachowa sie tak samo.

## Objawy

Po uruchomieniu skryptu `git status` w zupelnie innym repozytorium pokazuje dziesiatki
zmienionych plikow, ktorych nie tykales.

## Co robic zamiast

1. **Wyklucz katalogi ze skrotami jawnie**, po nazwie, w liscie pomijanych:
   `SKIP_DIRS = {'.git', '.obsidian', '40-Project-Kerf'}`.
2. **Zanim uruchomisz cokolwiek masowego, zrob snapshot `git status`** wszystkich repozytoriow,
   ktore moga byc w zasiegu. Bez tego nie odroznisz swojej zmiany od cudzej i nie cofniesz
   selektywnie.
3. **Cofaj po pliku, nie po katalogu.** `git checkout -- Docs/` wymiotloby takze zmiany,
   ktore byly tam przed twoim skryptem.

## Dowod

Skrypt zamieniajacy dlugie myslniki, uruchomiony na `D:\GameDevOS`, ruszyl 43 pliki
w `Docs` i `_Handoff` repozytorium gry. Wykryte porownaniem `git status` ze snapshotem
sprzed pracy, cofniete lista plikow (nie katalogiem), skrypt poprawiony.

## Powiazane

- [[gate-must-have-provable-failure-mode]]
