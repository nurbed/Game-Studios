# Unity 6 — Breaking Changes

**Last verified:** 2026-03-25

Follow upgrade guides **in order** — do not skip versions. API Updater handles
some renames automatically on project open.

---

## Unity 6.0 (6000.0) — Most Disruptive

### Lighting
- **Enlighten Baked GI removed** — replaced by Progressive Lightmapper. No fallback.
- **Auto Generate lighting removed** — call `Lightmapping.Bake()` explicitly or use the Lighting window.
- **Environment lighting not baked automatically** — ambient probe and skybox Reflection Probe must be generated manually after scene changes.
- **Light Probe brightness corrected** — now 100% of lightmap brightness (was 94%). Scenes will appear slightly brighter; rebake to match.

### Rendering
- **`GraphicsFormat.DepthAuto`, `.ShadowAuto`, `.VideoAuto` now throw compile errors** (were warnings in 2022). Use explicit formats.
- **Metal shader buffer layout changed** — `min16float`/`half`/`real` consistently map to 32-bit MSL float. May affect custom Metal shaders.
- **Mipmap limits reversed for runtime textures** — opt-in via `MipmapLimitDescriptor` (was opt-out).

### Android
- **`UnityPlayer` class reorganized** — no longer extends `FrameLayout`. Use `getFrameLayout()`. Replace with `UnityPlayerForActivityOrService` or `UnityPlayerForGameActivity`.
- **Toolchain updated**: Gradle 8.4, AGP 8.3.0, SDK Build Tools 34.0.0, JDK 17.
- **Minimum API level raised to 24** (Android 7.0).

### iOS
- **Minimum deployment target raised to iOS 13.**

### DOTS/ECS
- **Entities API complete overhaul** (Entities 1.0+):

```csharp
// ❌ OLD (pre-Unity 6)
public class HealthComponent : ComponentData { public float Value; }
public class DamageSystem : ComponentSystem { }

// ✅ NEW (Unity 6+)
public struct HealthComponent : IComponentData { public float Value; }
public partial struct DamageSystem : ISystem {
    public void OnCreate(ref SystemState state) { }
    public void OnUpdate(ref SystemState state) { }
}
```

### Input
- **Legacy `Input` class deprecated** — install `com.unity.inputsystem` package:

```csharp
// ❌ OLD
if (Input.GetKeyDown(KeyCode.Space)) { }

// ✅ NEW
if (Keyboard.current.spaceKey.wasPressedThisFrame) { }
```

### Package Manager
- **`UPM_CACHE_PATH` env var removed** — use `UPM_CACHE_ROOT`.

---

## Unity 6.1 (6000.1)

### Rendering
- **`_FORWARD_PLUS` shader keyword renamed to `_CLUSTER_LIGHT_LOOP`** — custom shaders referencing this keyword must be updated manually (API Updater does not handle shader source).
- **DirectX 12 becomes default Auto Graphics API on Windows** — existing projects retain their previous setting unless Auto API is re-enabled. New projects default to DX12.

### Physics
- **`Rigidbody.SetDensity(float)` deprecated** — use `Rigidbody.mass` directly.

### Textures
- **PVRTC texture compression deprecated** — migrate to ASTC or ETC2.

### Android
- **Toolchain updated**: Gradle 8.11, AGP 8.7.2, NDK r27c.

### Editor
- **Window menu restructured** — custom keyboard shortcuts to Package Manager, Asset Store, Services must be recreated.

---

## Unity 6.2 (6000.2)

### URP — Scriptable Render Passes
- **`SetupRenderPasses` API deprecated** — migrate all custom `ScriptableRendererFeature` code to Render Graph using `AddRenderPasses`.
- **URP Compatibility Mode deprecated** — warning: will be removed in 6.4.
- **`AfterRendering` injection point timing changed** — now always runs after the final blit. To preserve old behavior, migrate to `AfterRenderingPostProcessing`.

### UI Toolkit
- **`VisualElement.transform` deprecated** — for setting: use `style.translate`, `style.rotate`, `style.scale`. For reading: use `resolvedStyle.*`.
- **`ExecuteDefaultAction` / `ExecuteDefaultActionAtTarget` deprecated** — use `HandleEventTrickleDown` / `HandleEventBubbleUp`.
- **`PreventDefault()` deprecated** — use `StopPropagation()`.
- **`UxmlFactory` / `UxmlTraits` deprecated** — use `[UxmlElement]` and `[UxmlAttribute]` attributes.

### Volume Framework
- **`CustomEditorForRenderPipelineAttribute` deprecated** — use `[CustomEditor]` + `[SupportedOnRenderPipeline]`.
- **`VolumeComponentMenuForRenderPipelineAttribute` deprecated** — use `[VolumeComponentMenu]` + `[SupportedOnRenderPipeline]`.
- **`RenderPipelineEditorUtility.FetchFirstCompatibleTypeUsingScriptableRenderPipelineExtension` deprecated** — use `GetDerivedTypesSupportedOnCurrentPipeline`.

### Addressables
- **Asset loading failures now throw exceptions** (not silent null). Add try/catch:

```csharp
try {
    var handle = Addressables.LoadAssetAsync<Sprite>("key");
    var sprite = await handle.Task;
} catch (Exception e) {
    Debug.LogError($"Failed to load: {e}");
}
```

---

## Unity 6.3 (6000.3 LTS)

LTS stabilization release. Continued deprecation warnings from 6.2. No major
additional breaking changes documented. Recommended for projects requiring long-term support.

---

## Unity 6.4 (6000.4)

### URP — Render Graph (MANDATORY, no fallback)
- **URP Compatibility Mode fully removed** — the `URP_COMPATIBILITY_MODE` scripting define is gone.
- **`SetupRenderPasses` removed** — was deprecated in 6.2, now a compile error.
- All custom render passes MUST use the Render Graph API. See `current-best-practices.md`.

### Volume Framework
- **`IVolume` interface removed** — replace with the new `VolumeGizmoDrawer` class.

### Textures
- **PVRTC texture compression removed** (deprecated in 6.1) — must use ASTC or ETC2. Existing PVRTC assets will fail to import.

### Lighting
- **`LightShadowCasterMode` enum renamed** (auto-updated by API Updater):
  - `NonLightmappedOnly` → `ShadowMask`
  - `Everything` → `DistanceShadowMask`

### GameObject Lifecycle
- **`OnDisable` callback hierarchy behavior changed** — now fires across the entire GameObject hierarchy on `Object.Destroy`, matching `DestroyImmediate`. Remove any manual hierarchy propagation workarounds.

### Asset Database
- **Artifact dependency behavior changed** — reimports only trigger when import results change. Importers relying on source-change triggers must explicitly declare both `DependsOnSourceAsset` and `DependsOnArtifact`.

### DOTS / ECS
- **`InstanceID` deprecated** — use the new `EntityId` type in ECS contexts.
- **Entities, Collections, Mathematics, Entities Graphics** are now Core packages — no Package Manager install needed.

### Mobile / XR
- **iOS/tvOS/visionOS runtime library path changed** — from `XcodeFolder/Libraries/` to `XcodeFolder/Frameworks/UnityRuntime.framework/UnityRuntime`. Simulator builds use static frameworks.
- **SRP Core physics dependency changed** from hard to soft — packages combining Physics and SRP Core must explicitly declare `com.unity.modules.physics`.
- **HoloLens 2 support deprecated** — migrate to the OpenXR plugin.

---

## Migration Checklist (2022 LTS → Unity 6.4)

- [ ] Audit all DOTS/ECS code — `ISystem` + `IJobEntity` replaces `ComponentSystem`
- [ ] Replace `Input` class with Input System package
- [ ] Migrate ALL custom render passes to Render Graph API (mandatory)
- [ ] Replace PVRTC textures with ASTC or ETC2
- [ ] Add exception handling to Addressables calls
- [ ] Update `_FORWARD_PLUS` shader keyword to `_CLUSTER_LIGHT_LOOP`
- [ ] Update `UxmlFactory`/`UxmlTraits` to `[UxmlElement]`/`[UxmlAttribute]`
- [ ] Replace `VisualElement.transform` with `style.translate/rotate/scale`
- [ ] Update Volume component attributes (see deprecated-apis.md)
- [ ] Run API Updater on open (handles `LightShadowCasterMode` renames)
- [ ] Test physics behavior — verify solver iterations match expectations
- [ ] Verify minimum platform versions (Android API 24, iOS 13)
- [ ] Update Android toolchain to Gradle 8.11 / AGP 8.7.2 / NDK r27c
