---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [refactoring, deletion, checklist, technical-debt]
date: 2026-05-17
status: draft
---

# Before-Delete Legacy Class Checklist

## When to use
Before deleting any class, script file, component, or SO from a Unity project. The cost of checking is 5 minutes; the cost of skipping is potentially hours of recovery.

## Steps
1. Grep all .cs files:
   ```
   grep -r "ClassName" Assets/Project/Scripts/
   ```
2. Search scene and prefab YAML:
   ```
   grep -r "ClassName" Assets/ --include="*.unity" --include="*.prefab"
   ```
3. Search ScriptableObject asset files:
   ```
   grep -r "ClassName" Assets/ --include="*.asset"
   ```
4. Check AssemblyDefinition references (`.asmdef` files)
5. Only if all clear → delete

Recovery if you deleted prematurely: scene shows `m_Script: {fileID: 11500000, guid: 00000…}` — open scene → for each missing script → Inspector → ⋮ menu → Remove Component.

## Why this works
Unity stores MonoBehaviour references in scene/prefab YAML as GUID+fileID pairs. Deleting the script file leaves the GUID reference intact — the scene loads with "missing script" errors and broken behavior.

## Trade-offs
5-minute discipline per deletion. Always worth it.

## Variants
Same checklist applies to: renaming a class (update all references before renaming), moving a namespace (all `using` statements), removing a method (all callers).

See also: [[storage-migration-primary-plus-legacy-fallback]], [[legacy-code-conflict-after-refactor]]
