---
type: pattern
project: Timber Tycoon
suggested-category: workflow/patterns
tags: [git, unity, merge, fast-forward, worktree, workflow]
date: 2026-07-13
status: draft
---

# Pattern: przewiniecie galezi BEZ checkoutu (`git fetch . src:dst`)

## Problem

Chcesz zmergowac galaz roboczą do `master`, merge jest zwyklym przewinieciem (fast-forward),
ale **drzewo robocze nie jest czyste**. Klasyczna sekwencja:

```bash
git checkout master
git merge --ff-only feature/cos
```

...**zostanie odrzucona**:

```
error: Your local changes to the following files would be overwritten by checkout
```

Git broni Cie przed utrata zmian - i slusznie. Kusi, zeby to obejsc przez `git checkout -f`
albo `git reset --hard`. **To kasuje lokalne zmiany bezpowrotnie.**

## Dlaczego to boli szczegolnie w Unity

Unity **samo brudzi pliki** i nie da sie tego wylaczyc. Klasyczne ofiary:

- materialy z emisja / swiatlem (`Mat_Lamp_Bulb.mat`) - Unity przelicza je po KAZDYM Play Mode
- pliki `.meta` przy reimporcie
- `ProjectSettings/` przy zmianie ustawien edytora

Te pliki wisza jako `M` w `git status` **zawsze** i wiele projektow ma zasade "nigdy ich nie
commituj". Jesli taki plik **rozni sie miedzy galeziami**, checkout jest zablokowany i klasyczny
merge staje.

Drugi problem: `git checkout master` przy duzej roznicy galezi przestawia **tysiace plikow**
(u nas: 2617). Jesli Unity jest otwarte, edytor rzuca sie na masowy reimport w trakcie
przelaczania i potrafi narobic balaganu w `.meta`.

## Rozwiazanie

```bash
# stojac NA galezi roboczej, nie przelaczajac sie nigdzie:
git fetch . feature/cos:master
```

`git fetch` z repozytorium `.` (samym soba) i refspecem `zrodlo:cel` **przestawia wskaznik
galezi `master`, nie dotykajac drzewa roboczego**.

Co to daje:

- ✅ **zero plikow ruszonych na dysku** - brudne pliki Unity zostaja nietkniete
- ✅ **Unity moze zostac otwarte** - nic sie nie reimportuje
- ✅ **wymusza fast-forward** - bez wiodacego `+` w refspecu Git **odmowi**, gdyby merge nie byl
  przewinieciem. Czyli nie da sie tym przypadkiem nadpisac historii.
- ✅ nie trzeba `git stash` (ktory tez dziala, ale rusza pliki i trzeba pamietac o `pop`)

Potem normalnie:

```bash
git push origin master
```

## Kiedy uzywac

- merge do `master`/`main`, ktory jest **fast-forward** (`git merge-base --is-ancestor master feature/cos`)
- drzewo robocze jest brudne plikami, ktorych **nie chcesz ani commitowac, ani stracic**
- galezie roznia sie duza liczba plikow, a otwarty edytor/IDE zle znosi masowe przelaczanie
- chcesz zaktualizowac **inna** galaz niz ta, na ktorej stoisz (np. podciagnac `develop`
  bez opuszczania swojej)

## Ograniczenia

- **Nie zadziala na galaz aktualnie wypchnieta (checked out)** - Git odmowi przestawienia refa,
  na ktorym stoi HEAD. To wlasnie zabezpieczenie, o ktore chodzi.
- Dziala **tylko dla fast-forward**. Jesli potrzebny jest prawdziwy merge (z commitem scalajacym),
  trzeba sie przelaczyc lub uzyc `git worktree`.
- Wskaznik galezi sie przesuwa, ale Twoje **drzewo robocze nadal odpowiada starej galezi** -
  co jest tu celem, ale trzeba o tym pamietac, zeby sie nie zdziwic.

## Weryfikacja przed uzyciem

```bash
# czy to na pewno fast-forward?
git merge-base --is-ancestor master feature/cos && echo "FF OK"

# czy cokolwiek by zginelo? (musi byc 0)
git rev-list --count --all --not feature/cos
```

## Alternatywa

`git worktree add` - jesli naprawde potrzebujesz drugiego drzewa roboczego. Ciezsze
(kopia plikow na dysku), ale pozwala na pelny merge z konfliktami bez ruszania glownego
katalogu roboczego.
