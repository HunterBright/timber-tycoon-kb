---
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: engine/lessons
tags: [kimodo, animacja, text-to-motion, prompt, comfyui, npc]
date: 2026-08-09
status: draft
---

# W generatorze ruchu z tekstu KROPKA dzieli prompt na osobne ruchy

## Objaw

Generowanie animacji "idzie niosac skrzynie" (NVIDIA Kimodo przez ComfyUI) dawalo klipy,
w ktorych postac ma **wyprostowane rece** (kat lokcia 155-170 stopni) mimo wyraznego opisu
"trzyma pudlo obiema rekami". Dodatkowo klipy mialy **dwa razy wiecej klatek**, niz wynikalo
z zamowionego czasu (360 zamiast 180 przy 6 s).

## Przyczyna

Wezel tekstowy dzieli prompt **po kropkach na osobne ODCINKI RUCHU** (tooltip wprost:
"Use periods to separate multiple motion segments"), a sampler liczy liczbe klatek
**dla kazdego odcinka osobno**: `num_frames = [int(duration * fps)] * len(zdania)`.

Opis dwuzdaniowy:

> "A person walks forward while carrying a box. The box is held with both hands in front
> of the stomach, elbows bent and the torso upright."

nie zostal odczytany jako jeden ruch z doprecyzowana postawa, tylko jako **dwa sklejone
ruchy**: chod, a po nim osobny ruch wygenerowany z opisu postawy. Drugie zdanie, ktore
mialo byc doprecyzowaniem, stalo sie wlasnym zadaniem dla modelu.

## Dowod

Ten sam model, ta sama liczba krokow, rozne prompty:

| prompt | zdan | klatek przy 6 s | kat lokcia |
|---|---|---|---|
| "A person is walking while carrying an object with two hands in front of them." | 1 | 180 | 95 st. |
| "A person walks forward while carrying a box. The box is held with both hands..." | 2 | 360 | 143-170 st. |

Powtorzone na 10 promptach w dwoch partiach. Te same ziarna daja identyczne liczby
co do trzeciego miejsca po przecinku, wiec roznica pochodzi z promptu, nie z losowania.

## Regula

**Caly opis jednego ruchu musi zmiescic sie w JEDNYM zdaniu.** Postawe wpisuje sie
w to samo zdanie jako okolicznik, nie jako zdanie nastepne:

- ZLE: `"A person walks forward carrying a crate. The elbows are bent."`
- DOBRZE: `"A person walks forward carrying a crate with both hands at waist height."`

Wiele zdan uzywaj **swiadomie i tylko** wtedy, gdy naprawde chcesz sekwencji
(np. "podnosi skrzynie" -> "idzie" -> "odstawia").

## Bramka

W skrypcie sterujacym warto postawic twarda kontrole, ktora nie przepusci
przypadkowego zdania skladowego:

```python
if prompt.rstrip().rstrip(".").count(".") > 0:
    raise SystemExit("PROMPT WIELOZDANIOWY - rozbije ruch na odcinki: " + klucz)
```

## Skutek uboczny wart zapamietania

Liczba klatek jest **jedynym widocznym sygnalem**, ze prompt sie rozjechal - klip nadal
sie eksportuje, importuje i gra. Zanim uznasz partie za udana, porownaj liczbe klatek
z `czas * takt_modelu`. Rozjazd o calkowita wielokrotnosc = tyle odcinkow, ile zdan.

Powiazane: [[kimodo-takt-30-a-nosnik-60]] (wymuszanie trybu czasu przy eksporcie FBX).

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260801-1330-liczbe-postaci-mow-twierdzaco-negatyw-nie-wystarcza|Liczbe postaci w kadrze trzeba powiedziec TWIERDZACO - negatyw jej nie pilnuje]] - wspolne: prompt, npc
- [[20260802-1620-humanoid-retarget-poza-wzorcowa|20260802-1620-humanoid-retarget-poza-wzorcowa]] - wspolne: animacja, npc
<!-- /POWIAZANE:auto -->
