---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, editor, custom-editor, generator, terrain, ui]
date: 2026-05-17
status: draft
---

# Custom Editor Pattern for Generators

## When to use
Any MonoBehaviour that generates content procedurally (terrain, roads, mesh buildings) that designers need to iterate on without writing code.

## Steps
Create `Assets/Project/Scripts/Editor/{GeneratorName}Editor.cs`:

```csharp
[CustomEditor(typeof(TerrainGenerator))]
public class TerrainGeneratorEditor : Editor {
    public override void OnInspectorGUI() {
        DrawDefaultInspector();
        
        GUIStyle btnStyle = new GUIStyle(GUI.skin.button);
        btnStyle.fontSize = 16;
        btnStyle.fontStyle = FontStyle.Bold;
        
        if (GUILayout.Button("🌲 GENERUJ TEREN", btnStyle, GUILayout.Height(40)))
            ((TerrainGenerator)target).Generate();
        
        // Preset row
        EditorGUILayout.BeginHorizontal();
        if (GUILayout.Button("Płaski")) { gen.maxHeight = 1; EditorUtility.SetDirty(target); }
        if (GUILayout.Button("Wzgórza")) { gen.maxHeight = 4; EditorUtility.SetDirty(target); }
        if (GUILayout.Button("Górski")) { gen.maxHeight = 8; EditorUtility.SetDirty(target); }
        EditorGUILayout.EndHorizontal();
        
        if (GUILayout.Button("💾 Eksportuj mesh"))
            ExportMesh((TerrainGenerator)target);
    }
}
```

Wrap in `#if UNITY_EDITOR` or keep in `Editor/` folder.

## Why this works
Designer iterates via 3 button clicks instead of typing parameter values manually. Preset buttons lock in known-good configurations. Export button makes the workflow self-contained.

## Trade-offs
One editor script per generator. Low maintenance cost; pays off immediately on the second use.

## Variants
Same pattern for: road generators, building generators, NPC spawn configurators, audio configurators.
