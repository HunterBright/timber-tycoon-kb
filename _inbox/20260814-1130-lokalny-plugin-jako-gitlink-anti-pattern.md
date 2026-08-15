---
title: Lokalny plugin skilli wpiety do repo jako gitlink - pusty katalog po klonie
type: anti-pattern
status: draft
confidence: high
verified: '2026-08-14 (git ls-files -s w obu repo)'
date: 2026-08-14
project: Another Quest
tags:
- git
- claude-code
- plugins
- skills
applies_to: []
source: 'D:\Unity\Timber_Tycoon (gitlink 160000) vs D:\Unity\AnotherQuest (99 plikow)'
severity: medium
suggested-category: tooling/anti-patterns
time_lost: ''
---

# Lokalny plugin skilli wpiety do repo jako gitlink - pusty katalog po klonie

## Co probowalismy

Plugin z 35 skillami (sklonowany z GitHuba) lezal w korzeniu repo gry i byl wpiety przez
`.claude-plugin/marketplace.json` jako `"source": "./unity-claude-skills"`. Katalog **zachowal
wlasny `.git`**, wiec `git add` zapisal go jako **gitlink** (tryb `160000`, jeden wpis w indeksie)
- czyli submodule bez `.gitmodules`.

## Dlaczego nie dziala

- `git clone` repo gry daje **pusty katalog** plugina. Zaden `git submodule update` nie pomoze,
  bo bez `.gitmodules` git nie wie, skad go wziac.
- Wersja skilli nie jest wersjonowana razem z gra. Repo mowi "commit b954dcc", ale tresc zyje
  poza repo i moze zniknac razem z upstreamem.
- **Cichy tryb porazki**: dopoki pracujesz na tej samej maszynie, wszystko dziala. Awaria ujawnia
  sie dopiero na drugiej maszynie albo po archiwizacji projektu.
- Przy kopiowaniu konfiguracji do nowego projektu `cp -r` **przenosi rowniez `.git`** i powiela blad.

## Co robic zamiast

Przy wpinaniu plugina, ktory ma byc "przypiety w tym repo":

1. Skopiuj katalog, potem **usun zagniezdzony `.git`** (`rm -rf <plugin>/.git`).
2. Sprawdz, ze wchodzi jako zwykle pliki: `git ls-files -s <plugin> | head` ma pokazac `100644`,
   nie `160000`, a liczba wpisow ma odpowiadac liczbie plikow na dysku.
3. Zapisz **hash upstreamu i licencje w opisie commita** - proweniencja zostaje, zaleznosc znika.
4. Jesli plugin ma byc wspoldzielony miedzy projektami, id marketplace **nie moze byc dziedziczone**:
   id jest globalnie zwiazane ze sciezka w `~/.claude/plugins/known_marketplaces.json`. Skopiowany
   verbatim id sprawia, ze nowy projekt czyta skille **z folderu starego projektu** - i archiwizacja
   starego po cichu zabiera je nowemu.

## Sygnaly ostrzegawcze

- `git status` pokazuje katalog jako jedna pozycje zamiast listy plikow.
- `du` mowi o megabajtach, a `git ls-files | wc -l` mowi "1".
- Ten sam identyfikator marketplace w dwoch repo.

## Transferability

Dotyczy kazdego repo, do ktorego wpina sie sklonowana z zewnatrz paczke "na sztywno": pluginy
Claude Code, katalogi promptow, biblioteki narzedziowe, presety. Niezalezne od silnika.

## Related
- Reguła: kopiujac konfiguracje miedzy projektami, sprawdz, co w niej jest zwiazane ze sciezka zrodla
