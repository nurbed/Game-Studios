# Unity 6 — Current Best Practices

**Last verified:** 2026-03-25 (Unity 6.4 / 6000.4)

Modern Unity 6.4 patterns. Prefer these over anything the model suggests
from prior Unity 2022/2023 knowledge.

---

## Render Graph (URP) — MANDATORY from 6.4

URP Compatibility Mode is gone. All custom render passes must use the Render Graph API.
The LLM will suggest `Execute(ScriptableRenderContext, ref RenderingData)` — do not use it.

```csharp
// ❌ OLD — compile error in 6.4
public override void Execute(ScriptableRenderContext context, ref RenderingData data) { }

// ✅ NEW — Render Graph
public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData)
{
    using (var builder = renderGraph.AddRasterRenderPass<PassData>("MyPass", out var passData))
    {
        var resourceData = frameData.Get<UniversalResourceData>();
        builder.UseTexture(resourceData.activeColorTexture);
        builder.SetRenderFunc((PassData data, RasterGraphContext ctx) =>
        {
            // ctx.cmd for draw calls
        });
    }
}
```

**Key rules:**
- Declare ALL resource usage via `builder.UseTexture()` / `builder.SetRenderAttachment()` before the render func.
- Do NOT hold `RTHandle` references across frames inside Render Graph passes.
- Use **Render Graph Viewer** (Window > Analysis > Render Graph Viewer) to debug pass execution.

---

## GPU Resident Drawer

Automatically batches draw calls via `BatchRendererGroup`. Can reduce thousands of
draw calls to tens for large scenes — directly relevant to CloudsCity's city tiers.

**Enable**: Project Settings > Graphics > GPU Resident Drawer > Instanced Drawing

**Compatibility requirements:**
- Do NOT use `MaterialPropertyBlock` on batched objects — breaks batching
- Avoid `OnRenderObject` callbacks on batched objects
- Use DOTS-instancing shaders or standard URP Lit (already compatible)
- Static environment geometry benefits most; skinned/dynamic meshes benefit less

---

## Adaptive Probe Volumes (APV)

Replaces manual Light Probe Groups. Preferred for all new projects.

**Enable**: Lighting Settings > Light Probe System > Probe Volumes

Features:
- Auto-places probes based on scene geometry
- Disk streaming for large scenes
- Sky occlusion baking
- Lighting scenario blending (blend between baked lighting states at runtime)
- Available in both URP and HDRP

---

## DirectStorage (6.4, Windows)

Reduces CPU overhead for loading large assets (textures, meshes, DOTS entity data).
Useful for streaming multi-layer environments.

**Enable**: Project Settings > Player > Enable DirectStorage

---

## Object Finding — Use the New API

```csharp
// ❌ Deprecated — do not use
FindObjectsOfType<Enemy>();
FindObjectOfType<GameManager>();

// ✅ Use these
FindObjectsByType<Enemy>(FindObjectsSortMode.None);         // unsorted, fastest
FindObjectsByType<Enemy>(FindObjectsSortMode.ByInstanceID); // deterministic order
FindFirstObjectByType<GameManager>();   // fastest single-object lookup
FindAnyObjectByType<GameManager>();     // any instance, non-deterministic
```

---

## DOTS/ECS (Core in 6.4 — no separate install needed)

```csharp
// ✅ Modern ISystem (unmanaged, Burst-compatible)
public partial struct MovementSystem : ISystem {
    public void OnUpdate(ref SystemState state) {
        foreach (var (transform, speed) in
            SystemAPI.Query<RefRW<LocalTransform>, RefRO<MoveSpeed>>()) {
            transform.ValueRW.Position += speed.ValueRO.Value * SystemAPI.Time.DeltaTime;
        }
    }
}

// ✅ IJobEntity for parallel jobs
[BurstCompile]
public partial struct DamageJob : IJobEntity {
    public float DeltaTime;
    void Execute(ref Health health, in DamageOverTime dot) {
        health.Value -= dot.DamagePerSecond * DeltaTime;
    }
}
```

Use `EntityId` (not `InstanceID`) for ECS object identity in 6.4.

---

## Input System

```csharp
// ❌ Deprecated Legacy Input — do not use
if (Input.GetKeyDown(KeyCode.Space)) { }

// ✅ Input System package (com.unity.inputsystem)
using UnityEngine.InputSystem;

public class PlayerInput : MonoBehaviour {
    private PlayerControls _controls; // generated C# class from Input Actions asset

    void Awake() {
        _controls = new PlayerControls();
        _controls.Gameplay.Jump.performed += ctx => Jump();
    }

    void OnEnable() => _controls.Enable();
    void OnDisable() => _controls.Disable();
}
```

Create Input Actions asset in editor, right-click > Generate C# class.

---

## UI Toolkit (Runtime UI)

```csharp
// ✅ UI Toolkit — recommended for new projects
using UnityEngine.UIElements;

public class MainMenu : MonoBehaviour {
    void OnEnable() {
        var root = GetComponent<UIDocument>().rootVisualElement;
        root.Q<Button>("play-button").clicked += StartGame;
        root.Q<Label>("score").text = $"High Score: {PlayerPrefs.GetInt("HighScore")}";
    }
}

// ✅ Custom control (6.2+ attribute syntax)
[UxmlElement]
public partial class ResourceBar : VisualElement {
    [UxmlAttribute]
    public float MaxValue { get; set; }
}
```

UXML (structure) + USS (styling) = HTML/CSS-like workflow.
UGUI still works but UI Toolkit is preferred for new UI in Unity 6.

---

## Asset Management

```csharp
// ❌ Deprecated
var prefab = Resources.Load<GameObject>("City/BuildingA");

// ✅ Addressables (async, memory-efficient)
using UnityEngine.AddressableAssets;

public async Task<GameObject> SpawnBuildingAsync(string key) {
    try {
        var handle = Addressables.InstantiateAsync(key);
        return await handle.Task;
    } catch (Exception e) {
        Debug.LogError($"Failed to spawn building: {e}");
        return null;
    }
}
// Release when done: Addressables.ReleaseInstance(go);
```

---

## Lighting — Manual Bake Required

```csharp
// Trigger bake from code
await Lightmapping.BakeAsync();

// Or in editor: Window > Rendering > Lighting > Generate Lighting
// Auto Generate is OFF in Unity 6 — must bake manually after scene changes
```

---

## Performance

```csharp
// ✅ Burst-compiled parallel job
[BurstCompile]
struct UpdateJob : IJobParallelFor {
    public NativeArray<float3> Positions;
    public NativeArray<float3> Velocities;
    public float DeltaTime;

    public void Execute(int i) {
        Positions[i] += Velocities[i] * DeltaTime;
    }
}

// ✅ NativeArray — no GC, Burst-compatible
using var positions = new NativeArray<float3>(count, Allocator.TempJob);
```

---

## Animation (6.4)

State machines can now initialize in non-default states — no more one-frame
transition delay on Animator start. No API change required; applies automatically.

---

## Scripting — C# 9+ Features (Unity 6)

```csharp
// ✅ Records for data
public record BuildingData(string Id, int Level, Vector3 Position);

// ✅ Pattern matching
var result = building switch {
    Tower t when t.Level > 5 => t.Upgrade(),
    Residential r => r.AddFloor(),
    _ => null
};

// ✅ Init-only properties
public class CityConfig {
    public int MaxTiers { get; init; }
    public float TierHeight { get; init; }
}
```

---

## Testing (NUnit / Unity Test Framework)

```csharp
[UnityTest]
public IEnumerator Building_WhenPlaced_IncreasesPopulation() {
    var city = new GameObject().AddComponent<CityManager>();
    var before = city.Population;

    city.PlaceBuilding(BuildingType.Residential, Vector3.zero);
    yield return null; // one frame

    Assert.Greater(city.Population, before);
}
```

Run via Window > General > Test Runner.

---

## Summary: Unity 6.4 Stack

| Feature | Use This | Avoid |
|---------|----------|-------|
| **Render passes** | `RecordRenderGraph()` | `Execute()` (removed) |
| **Instancing** | GPU Resident Drawer | Manual `MaterialPropertyBlock` batching |
| **Probes** | Adaptive Probe Volumes | Manual Light Probe Groups |
| **Input** | Input System package | `Input` class |
| **UI** | UI Toolkit | UGUI (still works, not preferred) |
| **ECS** | `ISystem` + `IJobEntity` | `ComponentSystem` (removed) |
| **Assets** | Addressables | `Resources.Load` |
| **Object find** | `FindObjectsByType` | `FindObjectsOfType` |

---

**Sources:**
- https://docs.unity3d.com/6000.4/Documentation/Manual/WhatsNewUnity64.html
- https://docs.unity3d.com/6000.4/Documentation/Manual/BestPracticeGuides.html
- https://docs.unity3d.com/Packages/com.unity.entities@1.3/manual/index.html
- https://docs.unity3d.com/Packages/com.unity.inputsystem@1.11/manual/index.html
