# Unity 6 — Deprecated & Removed APIs

**Last verified:** 2026-03-25

Use this table before writing any Unity API call. **REMOVED** = compile error.
**DEPRECATED** = warning, will be removed in a future version.

---

## Object Finding

| Status | Old API | Replacement |
|--------|---------|-------------|
| DEPRECATED | `Object.FindObjectsOfType<T>()` | `Object.FindObjectsByType<T>(FindObjectsSortMode.None)` |
| DEPRECATED | `Object.FindObjectOfType<T>()` | `Object.FindFirstObjectByType<T>()` or `Object.FindAnyObjectByType<T>()` |

---

## Input

| Status | Old API | Replacement |
|--------|---------|-------------|
| DEPRECATED | `Input.GetKey()` | `Keyboard.current[Key.X].isPressed` |
| DEPRECATED | `Input.GetKeyDown()` | `Keyboard.current[Key.X].wasPressedThisFrame` |
| DEPRECATED | `Input.GetMouseButton()` | `Mouse.current.leftButton.isPressed` |
| DEPRECATED | `Input.GetAxis()` | `InputAction` callbacks |
| DEPRECATED | `Input.mousePosition` | `Mouse.current.position.ReadValue()` |

Install package: `com.unity.inputsystem`

---

## Physics

| Status | Old API | Replacement |
|--------|---------|-------------|
| DEPRECATED (6.1) | `Rigidbody.SetDensity(float)` | `Rigidbody.mass = value` directly |
| DEPRECATED | `Physics.RaycastAll()` | `Physics.RaycastNonAlloc()` — avoids GC |

---

## Rendering / Graphics

| Status | Old API | Replacement |
|--------|---------|-------------|
| REMOVED | `GraphicsFormat.DepthAuto` | Use an explicit depth format |
| REMOVED | `GraphicsFormat.ShadowAuto` | Use an explicit shadow format |
| REMOVED | `GraphicsFormat.VideoAuto` | Use an explicit video format |
| RENAMED (6.1) | `_FORWARD_PLUS` shader keyword | `_CLUSTER_LIGHT_LOOP` |
| REMOVED (6.4) | PVRTC texture compression | ASTC or ETC2 |
| DEPRECATED | `CommandBuffer.DrawMesh()` in SRP | Render Graph API |
| DEPRECATED | `OnPreRender()` / `OnPostRender()` | `RenderPipelineManager` callbacks |
| DEPRECATED | `Camera.SetReplacementShader()` | Custom render pass |

---

## URP — Scriptable Renderer Features

| Status | Old API | Replacement |
|--------|---------|-------------|
| REMOVED (6.4) | `SetupRenderPasses()` override | `AddRenderPasses()` with Render Graph |
| REMOVED (6.4) | `ScriptableRenderPass.Execute()` | `RecordRenderGraph()` |
| REMOVED (6.4) | `URP_COMPATIBILITY_MODE` scripting define | N/A — migrate fully to Render Graph |
| CHANGED (6.2) | `AfterRendering` injection point timing | Use `AfterRenderingPostProcessing` |

---

## Volume Framework

| Status | Old API | Replacement |
|--------|---------|-------------|
| REMOVED (6.4) | `IVolume` interface | `VolumeGizmoDrawer` class |
| DEPRECATED (6.2) | `CustomEditorForRenderPipelineAttribute` | `[CustomEditor]` + `[SupportedOnRenderPipeline]` |
| DEPRECATED (6.2) | `VolumeComponentMenuForRenderPipelineAttribute` | `[VolumeComponentMenu]` + `[SupportedOnRenderPipeline]` |
| DEPRECATED (6.2) | `RenderPipelineEditorUtility.FetchFirstCompatibleTypeUsingScriptableRenderPipelineExtension` | `GetDerivedTypesSupportedOnCurrentPipeline` |

---

## UI Toolkit

| Status | Old API | Replacement |
|--------|---------|-------------|
| DEPRECATED (6.2) | `VisualElement.transform` (set) | `style.translate`, `style.rotate`, `style.scale` |
| DEPRECATED (6.2) | `VisualElement.transform` (get) | `resolvedStyle.translate`, `resolvedStyle.rotate`, `resolvedStyle.scale` |
| DEPRECATED (6.2) | `ExecuteDefaultAction` | `HandleEventTrickleDown` |
| DEPRECATED (6.2) | `ExecuteDefaultActionAtTarget` | `HandleEventBubbleUp` |
| DEPRECATED (6.2) | `PreventDefault()` | `StopPropagation()` |
| DEPRECATED (6.2) | `UxmlFactory<T, D>` | `[UxmlElement]` attribute on the class |
| DEPRECATED (6.2) | `UxmlTraits` | `[UxmlAttribute]` attributes on fields |

### UI Toolkit — New Custom Control Pattern

```csharp
// ❌ OLD (deprecated)
public new class UxmlFactory : UxmlFactory<MyControl, UxmlTraits> { }

// ✅ NEW
[UxmlElement]
public partial class MyControl : VisualElement
{
    [UxmlAttribute]
    public string Label { get; set; }
}
```

---

## Lighting

| Status | Old API | Replacement |
|--------|---------|-------------|
| REMOVED | Enlighten Baked GI | Progressive Lightmapper |
| REMOVED | Auto Generate lighting (editor toggle) | `Lightmapping.Bake()` / manual bake |
| RENAMED (6.4) | `LightShadowCasterMode.NonLightmappedOnly` | `LightShadowCasterMode.ShadowMask` |
| RENAMED (6.4) | `LightShadowCasterMode.Everything` | `LightShadowCasterMode.DistanceShadowMask` |

---

## DOTS / ECS

| Status | Old API | Replacement |
|--------|---------|-------------|
| REMOVED | `ComponentSystem` | `ISystem` (unmanaged, Burst-compatible) |
| REMOVED | `JobComponentSystem` | `ISystem` with `IJobEntity` |
| REMOVED | `GameObjectEntity` | Pure ECS workflow |
| REMOVED | `ComponentDataFromEntity<T>` | `ComponentLookup<T>` |
| DEPRECATED (6.4) | `InstanceID` | `EntityId` |

---

## UI (Legacy UGUI)

| Status | Old API | Replacement |
|--------|---------|-------------|
| DEPRECATED | `Canvas`-based UGUI | `UIDocument` (UI Toolkit) — recommended for new projects |
| DEPRECATED | `Text` component | `TextMeshPro` or UI Toolkit `Label` |

UGUI still works; UI Toolkit is the recommended path for new projects.

---

## Asset Loading

| Status | Old API | Replacement |
|--------|---------|-------------|
| DEPRECATED | `Resources.Load()` | Addressables |
| DEPRECATED | Synchronous asset loading | `Addressables.LoadAssetAsync()` |

---

## Scripting

| Status | Old API | Replacement |
|--------|---------|-------------|
| DEPRECATED | `WWW` class | `UnityWebRequest` |
| DEPRECATED | `Application.LoadLevel()` | `SceneManager.LoadScene()` |

---

## Package Manager / Environment

| Status | Old API | Replacement |
|--------|---------|-------------|
| REMOVED | `UPM_CACHE_PATH` env var | `UPM_CACHE_ROOT` |

---

## Android (UnityPlayer)

| Status | Old API | Replacement |
|--------|---------|-------------|
| CHANGED (6.0) | `UnityPlayer` extends `FrameLayout` | `UnityPlayer.getFrameLayout()` |
| REPLACED (6.0) | `UnityPlayer` class | `UnityPlayerForActivityOrService` or `UnityPlayerForGameActivity` |

---

## Quick Migration Patterns

### Input
```csharp
// ❌ Deprecated
if (Input.GetKeyDown(KeyCode.Space)) { Jump(); }

// ✅ New Input System
if (Keyboard.current.spaceKey.wasPressedThisFrame) { Jump(); }
```

### Asset Loading
```csharp
// ❌ Deprecated
var prefab = Resources.Load<GameObject>("Enemies/Goblin");

// ✅ Addressables
var handle = Addressables.LoadAssetAsync<GameObject>("Enemies/Goblin");
await handle.Task;
var prefab = handle.Result;
```

### UI Toolkit
```csharp
// ❌ Deprecated (UGUI Text)
GetComponent<Text>().text = "Score: 100";

// ✅ UI Toolkit
rootVisualElement.Q<Label>("score-label").text = "Score: 100";
```
