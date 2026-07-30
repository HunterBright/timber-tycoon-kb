---
type: anti-pattern
project: Timber Tycoon
suggested-category: engine/anti-patterns
tags: [unity, git, git-lfs, scene, serialization, repo-hygiene, gitattributes]
date: 2026-07-13
status: draft
---

# Anti-pattern: binarna scena Unity w Git LFS = nieodwracalny, rosnacy bez konca bloat

## Co sie robi (i dlaczego wyglada rozsadnie)

Standardowy `.gitattributes` dla Unity (kopiowany z setek poradnikow) wrzuca do LFS wszystko,
co "wyglada binarnie":

```
*.unity  filter=lfs diff=lfs merge=lfs -text
*.asset  filter=lfs diff=lfs merge=lfs -text
*.prefab filter=lfs diff=lfs merge=lfs -text
*.mat    filter=lfs diff=lfs merge=lfs -text
*.png    filter=lfs diff=lfs merge=lfs -text
*.blend  filter=lfs diff=lfs merge=lfs -text
```

Jednoczesnie projekt zostaje na domyslnej dla wielu setupow **binarnej serializacji sceny**
(Project Settings -> Editor -> Asset Serialization -> Mixed/Force Binary).

Wyglada niewinnie. Nie jest.

## Dlaczego to nie dziala

**Git LFS nie robi delt.** Zwykly Git pakuje wersje pliku tekstowego jako roznice
("zmienily sie 3 linijki"). LFS przechowuje **kazda wersje jako osobny, kompletny obiekt**.

Scena Unity to plik, ktory zmienia sie przy KAZDEJ pracy w edytorze. Binarna scena wazy
kilka-kilkanascie MB. Efekt:

> **Przesunieta jedna lampa = caly plik sceny (8,5 MB) lezy w repo NA ZAWSZE.**

Zmierzone na realnym projekcie (Timber Tycoon, ~1 rok pracy):

| | |
|---|---|
| `Assets/Demo_Scene.unity` | **119 wersji = 530 MB** (35% calego LFS) |
| Wszystkie pliki `.unity` (scena + backupy + "recovery") | **720 MB = 48% LFS** |
| Martwe stare wersje (nikt ich nigdy nie otworzy) | **907 MB = 60% calego LFS** |
| Potrzebne do dzialajacej gry | 596 MB |

Jeden pojedynczy push (43 commity) dolozyl 83 MB, **z czego 76 MB to 9 kopii sceny**.

## I teraz czesc, ktora naprawde boli

**Tego NIE DA SIE cofnac.**

- GitHub **nigdy nie usuwa obiektow LFS**. Raz wyslany obiekt liczy sie do limitu na zawsze,
  nawet gdy zaden commit juz go nie uzywa.
- Usuniecie plikow nowym commitem -> **nic nie zwalnia**.
- Skasowanie galezi -> **nic nie zwalnia**.
- Przepisanie historii (rebase / filter-branch / BFG) -> **rowniez nic nie zwalnia**. Obiekty
  zostaja na serwerze.
- Jedyne realne drogi: **skasowac i odtworzyc cale repozytorium** (traci issues, PR-y, gwiazdki)
  albo **napisac do GitHub Support** z prosba o purge.

Dodatkowo: nawet po skutecznym usunieciu, rozliczenie biezacego miesiaca nie jest przeliczane.

Czyli: **kazdy dzien zwloki to trwale zajete miejsce.** To nie jest dlug techniczny, ktory
sie splaca. To jest dlug, ktorego sie nie da splacic.

## Co robic zamiast tego (od pierwszego dnia projektu)

### 1. Force Text serialization
`Project Settings -> Editor -> Asset Serialization -> Force Text`

Scena staje sie plikiem YAML. Jest wtedy **wiekszy na dysku**, ale Git pakuje go deltami,
wiec commit sceny kosztuje **kilkadziesiat KB zamiast kilku MB**. To rowniez jedyny sposob,
zeby scena i prefaby w ogole dawaly sie sensownie mergowac i przegladac w diffie.

### 2. Zdjac z LFS to, co jest tekstem
Po Force Text **usunac** z `.gitattributes`:

```
*.unity  *.asset  *.prefab  *.mat  *.anim  *.controller
```

To sa male pliki tekstowe. W badanym repo mialy razem **ponad 1300 wersji** w historii
i siedzialy w LFS bez zadnego powodu.

W LFS zostawic to, co faktycznie jest binarne i duze: `*.fbx *.png *.tga *.psd *.wav *.mp3
*.blend *.otf *.ttf`.

### 3. Pilnowac, co wpada do LFS przy commicie
`.gitattributes` lapie po rozszerzeniu, wiec **byle zrzut ekranu wrzucony do repo idzie do LFS
na zawsze**. W badanym projekcie do commita szlo 12 MB pogladowych PNG-ow z folderu roboczego
i 6,7 MB logow buildu - w `.gitignore` nie bylo ZADNEJ reguly na `*.log`.

Przed commitem czegokolwiek binarnego:
```bash
git check-attr filter -- <plik>     # "filter: lfs" = idzie do LFS NA ZAWSZE
```

Foldery robocze (logi, zrzuty, podglady, backupy scen) -> **`.gitignore` od razu**, zanim
ktokolwiek je zacommituje.

## Sygnaly ostrzegawcze

- `.git/lfs` rosnie do gigabajtow, a projekt jest maly
- `git lfs ls-files --all | wc -l` zwraca tysiace obiektow
- jeden plik ma dziesiatki/setki wersji w LFS:
  ```bash
  git lfs ls-files --all --long | awk '{ $1=""; print }' | sort | uniq -c | sort -rn | head
  ```
- w historii sa `_Recovery/`, `_Backup_*/`, `Assets/Scenes/` (stara kopia sceny), wyjscia
  z Blendera - **kazde z nich to setki MB, ktorych nie da sie odzyskac**

## Limity GitHuba (stan: lipiec 2026)

- **Free / Pro: 10 GiB miejsca + 10 GiB transferu/miesiac** (podniesione ze starych 1 GB;
  pakiety danych za $5 zastapione rozliczeniem "placisz za nadwyzke")
- Team / Enterprise Cloud: 250 GiB
- Limit jest **per konto wlasciciela repo**, nie per repo
- **Push (upload) liczy sie TYLKO do miejsca, NIE do transferu.** Transfer zjadaja pobrania
  (clone/fetch/checkout).
- **Zuzycia LFS NIE DA SIE odczytac przez REST API.** Endpoint
  `/users/{u}/settings/billing/shared-storage` dotyczy wylacznie Actions i Packages.
  Jedyne miejsce: **https://github.com/settings/billing**
- Read-only sprawdzenie z linii polecen (0 wyslanych bajtow): POST na
  `<repo>.git/info/lfs/objects/batch` z `"operation":"upload"` i OID-em obiektu, ktory juz
  jest na serwerze. HTTP 403 "over data quota" = zablokowany.

## Zrodla

- https://docs.github.com/en/billing/concepts/product-billing/git-lfs
- https://github.com/git-lfs/git-lfs/blob/main/docs/api/batch.md
- Timber Tycoon, `_Handoff/REPO_STATE.md` (2026-07-13)

## Powiazane

- [[fbx-binary-overwrite-corrupts-bindposes]] - inny przypadek "binarka + git = klopot"
