# Phase 0 Research — Tactical RPG Demo

**Project constraints:**
- Unity 6.3 LTS, C#
- ScriptableObject-based data architecture
- Single scene architecture (Main.unity) with GameStateMachine
- No third-party frameworks (TextMeshPro OK)
- 2D sprites with SpriteRenderer
- PC standalone target (Windows/Mac/Linux)
- Programmer art for demo

---

## 1. Unity Version & Render Pipeline

**Decision:** Unity 6.3 LTS with URP (Universal Render Pipeline) 2D

**Rationale:** URP 2D Renderer is the official pipeline for 2D games in Unity 6. Built-in pipeline is legacy. URP 2D provides: 2D Lights, sprite sorting, Shader Graph for 2D, better performance on modern hardware. No need for HDRP features.

**Alternatives Considered:**
- Built-in Renderer — legacy, no 2D lights, limited shader support
- HDRP — overkill for 2D

---

## 2. UI Framework

**Decision:** UI Toolkit (UXML/USS)

**Rationale:** As recommended in DemoImplementationPlan_v1.md. UI Toolkit naturally enforces SPEC-32's rule: UI is read-only view + command sender, never mutates combat state. Data-binding separates presentation from logic. CSS-like USS styling enables rapid iteration. In Unity 6, UI Toolkit is production-ready for runtime UI.

**Alternatives Considered:**
- uGUI — acceptable fallback if team lacks UI Toolkit experience, but DO NOT mix both

---

## 3. RNG Implementation

**Decision:** Xoshiro128** PRNG with xxHash32 seed derivation via `Unity.Mathematics.math.hash()`

**Rationale:** SPEC-28 mandates this exact implementation. Xoshiro128** provides 2^128 period, fast execution, high quality randomness. xxHash32 via `math.hash()` is cross-platform deterministic.

**Key details:**
- FNV-1a for stable string hashing (`StableStringHash`)
- `uint` seeds throughout
- 3 streams: Combat, Loot, Cosmetic
- `UnityEngine.Random` FORBIDDEN in gameplay code

**Reference:** prng.di.unimi.it for Xoshiro128** implementation. C# implementation available: port the reference Java/C implementation to a C# struct implementing `IRng` interface.

**Alternatives Considered:**
- `UnityEngine.Random` — forbidden by spec, not deterministic across platforms
- `System.Random` — not reproducible across .NET versions

---

## 4. Animation Approach

**Decision:** Frame-by-frame spritesheet animation using Unity Animator

**Rationale:** Demo uses programmer art with simple sprites. Frame-by-frame is faster to produce than bone-based animation. Unity Animator with AnimationClips referencing spritesheet frames is the standard approach.

**States per entity:** Idle (4f), Attack (6f), Hit (3f), Death (4f), plus class-specific (Fold 2f, Ghost transparency)

**Alternatives Considered:**
- Unity 2D Animation (bone-based) — overkill for programmer art
- Spine — third-party, excluded by constraint

---

## 5. Input System

**Decision:** Unity Input System (new)

**Rationale:** Required package per DemoImplementationPlan. Modern input handling with action maps. Supports keyboard+mouse (NFR-002). Built-in rebinding support.

**Note:** Input Actions asset defines combat action map: `SelectTarget`, `ConfirmAction`, `CancelAction`, `CycleKnight`, `OpenMenu`, `EndTurn`.

**Alternatives Considered:**
- Legacy Input Manager — deprecated, no action maps, no rebinding

---

## 6. Testing Framework

**Decision:** Unity Test Framework + NUnit (EditMode + PlayMode)

**Rationale:** SPEC-32 mandates headless EditMode tests for combat systems. NUnit integration is standard. EditMode tests validate deterministic logic without MonoBehaviour overhead. PlayMode tests for smoke testing full scene.

**Key:** All 5 integration scenarios from SPEC-15 run as EditMode tests. All TechSpec worked examples used as test vectors.

**Alternatives Considered:**
- xUnit — not natively supported by Unity Test Runner
- External test harnesses — adds complexity with no benefit

---

## 7. Save System Serialization

**Decision:** JSON serialization using Unity's `JsonUtility` or `System.Text.Json`

**Rationale:** Single auto-save at Camelot (from spec clarifications). Serializes `RunState` to JSON file. No mid-combat save needed. Simple, human-readable, debuggable.

**Location:** `Application.persistentDataPath + "/save.json"`

**Alternatives Considered:**
- Binary serialization — faster but harder to debug
- ScriptableObject serialization — not suitable for runtime state

---

## 8. Project Assembly Structure

**Decision:** 3 assemblies for clean dependency enforcement

**Rationale:** Prevents accidental coupling between systems. Enables faster compilation.

**Layout:**
- **DarkestKnight.Core** (`Scripts/Core` + `Scripts/Data`) — no Unity runtime dependencies where possible
- **DarkestKnight.Runtime** (`Scripts/Combat` + `Scripts/Meta` + `Scripts/UI`) — depends on Core
- **DarkestKnight.Tests** (`Tests/`) — depends on both
