# Unity — Version Reference

| Field | Value |
|-------|-------|
| **Engine Version** | Unity 6.4 (6000.4) |
| **Internal Version String** | 6000.4.x (e.g., 6000.4.0f1) |
| **Release Date** | March 19, 2026 |
| **Project Pinned** | 2026-03-25 |
| **Last Docs Verified** | 2026-03-25 |
| **LLM Knowledge Cutoff** | May 2025 |
| **Risk Level** | HIGH — Unity 6.x is entirely beyond LLM training data |

## Knowledge Gap Warning

The LLM's training data likely covers Unity up to ~2022 LTS / early 2023.x.
The entire Unity 6 release series (6000.0 through 6000.4) introduced significant
breaking changes the model does NOT reliably know. Always cross-reference this
directory before suggesting Unity API calls.

**Most critical gap:** URP Compatibility Mode is fully removed in 6.4.
Any LLM-suggested `ScriptableRenderPass.Execute()` override is wrong — use Render Graph.

## Post-Cutoff Version Timeline

| Version | Release | Risk Level | Key Theme |
|---------|---------|------------|-----------|
| 6.0 (6000.0) | Oct 2024 | HIGH | Enlighten removed, Android overhaul, GraphicsFormat errors, Light Probe fix |
| 6.1 (6000.1) | ~Early 2025 | HIGH | DX12 default on Windows, `_CLUSTER_LIGHT_LOOP` rename, PVRTC deprecated |
| 6.2 (6000.2) | ~Mid 2025 | HIGH | `SetupRenderPasses` deprecated, URP Compatibility Mode deprecated, UI Toolkit changes |
| 6.3 (6000.3 LTS) | ~Late 2025 | HIGH | LTS stabilization; 2-year support until Dec 2027 |
| 6.4 (6000.4) | Mar 2026 | HIGH | URP Compatibility Mode **removed**, PVRTC removed, DirectStorage, Project Auditor built-in |

## Reference Files in This Directory

- `breaking-changes.md` — version-by-version breaking changes (6.0 → 6.4)
- `deprecated-apis.md` — old API → replacement table
- `current-best-practices.md` — new patterns since training cutoff

## Verified Sources

- Official docs: https://docs.unity3d.com/6000.4/Documentation/Manual/
- What's new in 6.4: https://docs.unity3d.com/6000.4/Documentation/Manual/WhatsNewUnity64.html
- Upgrade to 6.4: https://docs.unity3d.com/6000.4/Documentation/Manual/UpgradeGuideUnity64.html
- Upgrade to 6.2: https://docs.unity3d.com/6000.2/Documentation/Manual/UpgradeGuideUnity62.html
- Upgrade to 6.0: https://docs.unity3d.com/6000.1/Documentation/Manual/UpgradeGuideUnity6.html
- Release notes index: https://unity.com/releases/unity-6
- Unity 6.4 release notes: https://unity.com/releases/editor/whats-new/6000.4.0f1
