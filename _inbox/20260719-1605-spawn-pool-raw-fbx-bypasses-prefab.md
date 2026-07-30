---
title: 'Anty-wzorzec: pula spawnera wskazuje surowy FBX zamiast prefabu-wrappera'
type: anti-pattern
status: draft
confidence: low
verified: ''
date: '2026-07-19'
project: Kerf - Sawmill Tycoon
tags:
- unity
- prefab
- spawner
- scriptableobject
- fbx
- override
applies_to: []
source: ''
suggested-category: engine/anti-patterns
---

# Anty-wzorzec: pula spawnera wskazuje surowy FBX zamiast prefabu-wrappera

## Co się stało
Pula aut NPC (ScriptableObject z listą prefabów) wskazywała bezpośrednio asset modelu
NPCPickup01.fbx (typ Model), a nie prefab NPCPickup01.prefab. Wszystkie poprawki zrobione
w prefabie - wyłączone cieniowanie (decyzja z play-testu!), podmiany materiałów, komponent
debugowy - NIGDY nie działały w grze, bo gra spawnowała goły FBX. Poprawka "cienie aut
NPC off" przez wiele tygodni wyglądała na wdrożoną, a nie działała.

## Dlaczego to podstępne
- W Inspectorze pole "GameObject" przyjmuje FBX i prefab tak samo - nic nie krzyczy.
- Efekty rozjazdu są ciche i odroczone (cienie, materiały, skrypty pomocnicze).
- Dodatkowo kod może przypadkiem ZALEŻEĆ od surowego FBX (u nas: malowanie lakieru
  szuka slotu po nazwie "Mat_Body", która na FBX istnieje, a w prefabie została
  podmieniona na inną) - wtedy naiwne "przepnij na prefab" psuje coś innego.

## Jak się bronić
1. W audytach/sondach sprawdzaj `PrefabUtility.GetPrefabAssetType` każdego wpisu puli
   spawnera: `Model` = czerwona flaga.
2. Poprawka wizualna na prefabie-wrapperze MUSI być zweryfikowana na obiekcie
   zespawnowanym w grze/buildzie, nie w podglądzie prefabu (por. "Edytor kłamie").
3. Przepinając pulę z FBX na prefab, przejrzyj kod spawnera pod kątem zależności od
   nazw materiałów/hierarchii surowego modelu.
