---
title: 'Anti-pattern: pickup zużywa się bezwarunkowo, ale nadaje zdolność za bramką fazy questa'
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-06-22'
project: Kerf - Sawmill Tycoon
tags:
- quest
- softlock
- pickup
- inventory
- state-machine
- unity
- gating
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Anti-pattern: pickup zużywa się bezwarunkowo, ale nadaje zdolność za bramką fazy questa

## Objaw
Gracz podnosi przedmiot (np. siekierę), przedmiot znika ze świata, ale narzędzie nie jest
użyteczne - nie da się go wybrać ani podnieść ponownie. Trwały softlock, niemożliwy do naprawy
bez edycji save'a.

## Co się stało (case study)
`AxePickup.OnInteract` zawsze wykonywał „zużycie" pickupa:
- `isCollected = true`
- `QuestWorldState.MarkConsumed(name)` (zapis: obiekt zniknął na stałe)
- `gameObject.SetActive(false)`

...ale samo ODBLOKOWANIE narzędzia szło tylko przez `QuestManager.CollectAxe()`, które miało
bramkę fazy: `if (CurrentQuest != QuestPhase.GetTools) return;`. `ToolManager` odblokowywał
siekierę dopiero w reakcji na event `OnAxeCollected`, a ten event odpalał się DOPIERO po bramce.

Efekt: gdy gracz podniósł siekierę poza fazą `GetTools` (poza kolejnością / przy drugim podejściu),
pickup się zużył i zniknął na stałe, ale narzędzie nigdy nie zostało nadane → koło narzędzi
pokazywało je jako zablokowane, a fizycznego pickupa już nie było. Softlock.

Bliźniaczy `ShovelPickup` był ODPORNY, bo odblokowywał narzędzie BEZPOŚREDNIO
(`ToolManager.UnlockTool(Shovel)`), niezależnie od fazy - i to była jedyna różnica.

## DRUGA odsłona (ważna korekta reguły)
Pierwsza naprawa załatwiła tylko unlock narzędzia. Okazało się, że bramkowanie SAMEGO KREDYTU questa
też softlockuje: gracz podniósł narzędzia przed fazą → ma je w EQ (unlock OK), ale `CollectAxe`
zwracał wcześnie (bramka fazy) → flaga `axeCollected` nigdy nie ustawiona, a pickup już zużyty →
quest nie do ukończenia. Ta sama klasa dotknęła też ścięcia świerka (`ChopTree` przed FirstCut) i
wykopania pniaka (`DigStump` przed DigStump) - wszędzie: akcja zużywa nieodtwarzalny zasób + kredyt
za bramką fazy + akcja osiągalna przed fazą.

## Reguła (transferowalna - SKORYGOWANA)
**Jeśli akcja zużywa NIEODTWARZALNY zasób (zniszczony pickup, ścięte drzewo, wykopany pniak) ORAZ
jest osiągalna PRZED swoją fazą questa, to NIE wolno bramkować fazą samego kredytu.** Trzeba albo:
- **(A) zapisywać kredyt NIEZALEŻNIE od fazy**, przechodzić dalej TYLKO we właściwej fazie (strażnik,
  by nie pominąć faz pośrednich), i AUTO-ZALICZAĆ przy WEJŚCIU w fazę oraz przy WCZYTANIU (odzysk z
  trwałego źródła prawdy, np. ToolManager - ratuje już-zablokowane zapisy); ALBO
- **(B) zablokować samą akcję** do właściwej fazy (wzorzec jak `CounterRepair.CanRepairCounter`).

Zdolność/posiadanie (unlock narzędzia, stan obiektu) ZAWSZE nadawaj bezwarunkowo. Bezpieczne do
bramkowania fazą są tylko akcje ODWRACALNE (załadunek/rozładunek) i te, których zasób powstaje dopiero
po wcześniejszej fazie (nie da się ich wykonać za wcześnie).

## Jak wykrywać podobne miejsca (audyt)
1. Znajdź wszystkie pickupy/konsumowalne (`IQuestWorldConsumable`, `OnInteract` + `SetActive(false)`/`Destroy`).
2. Dla każdego sprawdź: czy konsumpcja jest bezwarunkowa, a efekt (grant) za bramką
   `if (CurrentQuest != QuestPhase.X) return;` lub za eventem odpalanym po takiej bramce?
3. Jeśli tak → przenieś grant przed bramkę albo nadaj zdolność bezpośrednio w pickupie
   (wzorzec idempotentny: `UnlockTool` ma early-return jeśli już odblokowane).

## Fix zastosowany
1. Unlock: w `AxePickup.OnInteract` bezpośrednie `ToolManager.UnlockTool(Axe)` (symetria z `ShovelPickup`).
2. Kredyt (wariant A): w `QuestManager` usunięto bramki fazy z `CollectAxe`/`CollectShovel`/
   `ChopTutorialTree`/`DigStump` (kredyt zapisuje się zawsze), advance tylko we właściwej fazie
   (strażnik fazy w check), auto-zaliczenie przy wejściu w fazę (`AdvanceTo`) i przy wczytaniu
   (`DeferredPostLoad`, 2 klatki - bo ToolManager/managery ładują się w nieznanej kolejności).
   Odzysk istniejących zablokowanych zapisów: `SyncGetToolsFromToolManager()` (ToolManager = źródło prawdy).
Zweryfikowane harnessem Play-Mode 6/6 (early-pickup/chop/dig, odzysk, regresja kolejności i kaskady).
