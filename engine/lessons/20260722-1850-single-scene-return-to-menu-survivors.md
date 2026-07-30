---
title: 'Powrot do menu glownego w grze jednoscenowej: co przezywa przeladowanie sceny'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-22'
project: Kerf - Sawmill Tycoon (Timber Tycoon)
tags:
- unity
- scene-reload
- dontdestroyonload
- static-events
- main-menu
- single-scene
- build-only-bugs
applies_to:
- unity-6
- unity-2021+
- any-single-scene-game
source: ''
severity: high
time_lost: ~2h (rozpoznanie + budowa sprzatania i sondy)
promoted: '2026-07-30'
---

# Powrot do menu glownego w grze jednoscenowej: co przezywa przeladowanie sceny

## Problem
Gra ma JEDNA scene, a menu glowne jest nakladka otwierana raz przy starcie aplikacji
(hak `[RuntimeInitializeOnLoadMethod(AfterSceneLoad)]`). Trzeba bylo dodac "wyjscie do menu
glownego" z menu pauzy. Naiwne rozwiazanie - pokazac nakladke z powrotem - daje ciche klamstwo:
swiat za nakladka zostaje w stanie, w jakim gracz go zostawil, wiec przycisk "Nowa gra" startuje
w zuzytym swiecie, kasujac wylacznie plik zapisu.

Przeladowanie sceny naprawia to, ale odslania druga warstwe: **czesc stanu gry przezywa
przeladowanie i nikt tego nie widzi, dopoki nikt sceny nie przeladowuje**.

## Root cause
Przy `SceneManager.LoadScene` gina obiekty sceny. NIE gina:

1. **Obiekty z `DontDestroyOnLoad`.** Ich `Awake` juz nie zadziala, a stare instancje wygrywaja
   ze swiezymi (typowy singleton `if (Instance != null) { Destroy(this); return; }` kasuje
   NOWY obiekt, zostawiajac stary ze starym stanem).
2. **Zdarzenia STATYCZNE** (`public static event Action X`). Delegat jest polem KLASY, nie
   obiektu, wiec przezywa przeladowanie **razem z subskrybentami ze skasowanej sceny**. Kazde
   pozniejsze wywolanie leci w martwe obiekty, a nowa scena dopisuje sie obok starych.

Obie awarie ujawniaja sie **dopiero przy DRUGIM wejsciu do gry**. W Edytorze, gdzie sceny
zwykle nikt nie przeladowuje w trakcie testu, nie widac ich w ogole.

**Pulapka trzecia, znaleziona pomiarem:** `DontDestroyOnLoad` **cicho nie dziala na obiekcie,
ktory nie jest korzeniem hierarchii** - silnik loguje ostrzezenie i ignoruje zadanie. W tym
projekcie manager zadan od zawsze deklarowal trwalosc, ktorej nie mial. Nie wolno wiec ufac
liscie "co przezyje" spisanej z KODU - trzeba ja odczytac z LOGU dzialajacego builda.

## Solution
1. Wyjscie do menu = **przeladowanie sceny**, nie samo pokazanie nakladki. Tylko wtedy menu po
   powrocie znaczy to samo, co menu po starcie aplikacji.
2. Hak `RuntimeInitializeOnLoadMethod` biegnie RAZ na uruchomienie procesu i **nie odpali sie
   po przeladowaniu**. Menu otwiera sie z `SceneManager.sceneLoaded` (ten sam moment cyklu:
   po `Awake` calej sceny, przed `Start`).
3. Przed `LoadScene`: wyczyscic statyczne zdarzenia (metoda `ClearStaticSubscribers()` w kazdej
   klasie, ktora je deklaruje - statyczne zdarzenie mozna wyzerowac tylko wewnatrz swojego typu)
   i skasowac trwale obiekty niosace STAN GRY. Trwale obiekty **bezstanowe** (siatki
   bezpieczenstwa, muzyka, kursor) zostawic - one pytaja o serwisy na biezaco.
4. Obiekty tworzone leniwie (`EnsureInstance()`) mozna kasowac swobodnie - odtworza sie same.
   Obiektow tworzonych **wylacznie** przez `RuntimeInitializeOnLoadMethod` kasowac NIE WOLNO:
   nic ich juz nie odtworzy w tym procesie.
5. Czarna kurtyna na czas wczytywania (osobne plotno `DontDestroyOnLoad` o wysokim
   `sortingOrder`), zdejmowana klatke po otwarciu menu.

## Weryfikacja, ktora ma zeby
Test strukturalny ("czy jest przycisk") nie wystarczy. Dwa sprawdzenia, ktore realnie lapia te
rodzine bledow, oba wykonalne w automacie w BUILDZIE:

- **Przemiatanie refleksja**: znajdz wszystkie typy z publicznymi zdarzeniami statycznymi w
  assembly gry i zazadaj, zeby kazdy byl objety sprzataniem, a po sprzataniu **kazde pole
  delegata bylo null**. Nowa klasa ze statycznym zdarzeniem, o ktorej ktos zapomni, zapala
  lampke sama z siebie.
- **Prawdziwy restart**: zabrudzic swiat mierzalna wartoscia (np. dodac pieniadze), wykonac
  powrot, po przeladowaniu sprawdzic wartosc startowa ORAZ `ReferenceEquals` na managerach
  (po prawdziwym restarcie musza to byc INNE obiekty). `ReferenceEquals` omija "udawany null"
  Unity, wiec dziala takze na skasowanych obiektach.

Dodatkowo sprawdzic, czy aktywna scena w ogole jest w Build Settings (`buildIndex >= 0`) -
w Edytorze scena jest otwarta zawsze, w buildzie `LoadScene` nie ma czego wczytac i gracz
zostaje na czarnym ekranie.

## Pulapka nr 4, najdrozsza: "znajdz istniejace plotno UI"

Bootstrap UI mial klasyczna linijke: `MainCanvas = FindAnyObjectByType<Canvas>()` z komentarzem
"sprawdz, czy plotno juz istnieje". Zamysl byl taki: **plotno z SCENY**. Ale `FindAnyObjectByType`
przeszukuje wszystkie zaladowane sceny, **lacznie ze scena DontDestroyOnLoad**.

Po przeladowaniu sceny zyja tam plotna pomocnicze (podpowiedz przy kursorze, dymki nad postaciami,
czarna kurtyna przejscia). Bootstrap adoptowal jedno z nich, budowal na nim CALE UI gry - i UI
znikalo w chwili, gdy to tymczasowe plotno bylo sprzatane. Objaw dla gracza: menu **dziala**
(kamera krazy po ujeciach, logika odpowiada), ale ekran jest pusty.

Sygnal diagnostyczny jest w logu i widac go golym okiem: przy pierwszym starcie bootstrap pisze
"Canvas created", a po powrocie do menu "**Using existing Canvas**".

Fix jednolinijkowy i precyzyjny: adoptuj wylacznie plotno, ktorego `gameObject.scene` rowna sie
`SceneManager.GetActiveScene()`. Ogolniej: **kazde `FindAnyObjectByType` / `FindObjectsOfType`
staje sie dwuznaczne w chwili, gdy w grze pojawia sie obiekty trwale.** Zapytanie "znajdz obiekt
typu T" trzeba wtedy zawezic do sceny, w ktorej ma sens.

## What didn't work
- **Ufanie kodowi zamiast logowi**: lista "co przezywa przeladowanie" spisana z wywolan
  `DontDestroyOnLoad` byla o jeden obiekt za dluga - jeden z nich nie byl korzeniem hierarchii
  i silnik ignorowal znacznik od miesiecy.
- **Sprawdzanie samego przycisku**: pierwsza wersja kontroli patrzyla tylko, czy pozycja
  istnieje w menu pauzy. Przeszlaby nad kazdym z opisanych wyzej bledow.
- **Sprawdzanie FLAGI zamiast EKRANU**: kontrola "menu otwarte" (`IsOpen == true`) plus stan gry
  plus wartosc portfela **swiecila na zielono przy calkowicie pustym ekranie**. Flaga mowi, ze
  kod menu wykonal sie do konca - nie mowi, ze cokolwiek widac. Bledy warstwy prezentacji lapie
  wylacznie kontrola patrzaca na OBIEKTY NA EKRANIE: czy element istnieje, na jakim plotnie wisi,
  czy ma swoje dzieci. To ten sam blad rozumowania co "test wzajemnej zgodnosci nie jest testem
  zgodnosci z rzeczywistoscia".

## Transferability
Dotyczy kazdej gry Unity, ktora (a) trzyma rozgrywke w jednej scenie i pokazuje menu jako
nakladke, albo (b) wraca do menu przez przeladowanie sceny. Zdarzenia statyczne i
`DontDestroyOnLoad` to mechanizmy silnika, niezalezne od gatunku gry. Reguly o kolejnosci
(`RuntimeInitializeOnLoadMethod` raz na proces, `sceneLoaded` po `Awake`) sa uniwersalne.

## Related
- [[build-is-the-only-truth-editor-lies]] - ta sama rodzina "dziala w Edytorze, pada w buildzie"
- [[gate-must-have-provable-failure-mode]] - kazde nowe sprawdzenie potrzebuje udowodnionego trybu porazki
