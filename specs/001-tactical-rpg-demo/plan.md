# Implementation Plan: Knight Into the Darkness — Tactical RPG Demo

**Branch**: `001-tactical-rpg-demo` | **Date**: 2026-03-10 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-tactical-rpg-demo/spec.md`

## Summary

Build a 2D side-scrolling tactical RPG demo adapted from the tabletop RPG "Knight" by Antre Monde Editions. The demo features 8 procedurally generated knights (4 armor classes), Darkest Dungeon-style 4-rank combat with D6 dice pools, multi-layer armor pipeline, 4 faction enemy types, a 7-node mission with 2-phase boss, and a Camelot hub for between-mission management. Architecture follows SPEC-32: single scene, ScriptableObject data, CombatManager as sole authority, EventBus for UI decoupling, deterministic RNG (SPEC-28).

## Technical Context

**Language/Version**: C# / Unity 6.3 LTS
**Primary Dependencies**: Unity.Mathematics (xxHash32), TextMeshPro (UI text), Unity Input System, Unity Test Framework + NUnit
**Storage**: JSON file at `Application.persistentDataPath/save.json` (single auto-save slot, SPEC-13)
**Testing**: EditMode tests (headless combat, all SPEC worked examples as test vectors) + PlayMode smoke tests
**Target Platform**: PC standalone (Windows, Mac, Linux) — NFR-001
**Project Type**: Unity desktop game (2D tactical RPG)
**Performance Goals**: 60 fps during combat, <1s turn resolution
**Constraints**: No third-party frameworks for core systems (TextMeshPro OK). Programmer art. No Spine/skeletal animation.
**Scale/Scope**: 1 demo mission (7 nodes), 8 knights, 5 enemy types, ~50 UI screens/panels, ~60 ScriptableObject instances

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Evidence |
|-----------|--------|----------|
| I. Tabletop Fidelity First | PASS | All mechanics reference TechSpec SPECs which align with tabletop source in `/Context/` |
| II. Agent-Ready Specifications | PASS | Plan references SPEC files directly; no invented mechanics |
| III. Architecture Conventions (SPEC-32) | PASS | Single scene, RunState/CombatSession isolation, CombatManager authority, EventBus, French/English naming |
| IV. Math Rules (SPEC-03) | PASS | Floor division, strict exceed threshold, successive doubling = ×3 — encoded in research.md |
| V. Data Architecture | PASS | All game data as ScriptableObject instances; runtime state as plain C# — see data-model.md |
| VI. Testing Discipline | PASS | EditMode tests use TechSpec worked examples; 5 integration scenarios from SPEC-15 |
| VII. Dependency Phase Order (SPEC-16) | PASS | Build order aligns with 8-phase dependency map. Plan milestones follow M0→M1→M2→M3→...→M12 |
| VIII. Demo Scope | PASS | Modules OUT OF SCOPE, Point Faible OUT OF SCOPE, SCAFFOLDED items have no-op stubs |
| IX. Commit Strategy | PASS | One commit per task, SPEC reference in message |

**Post-Phase 1 Re-check**: All principles remain satisfied. Data model maps SO schemas from SPEC-01/18/19. Contracts define EventBus events from SPEC-27. Research resolves all NEEDS CLARIFICATION items.

## Project Structure

### Documentation (this feature)

```text
specs/001-tactical-rpg-demo/
├── plan.md              # This file
├── research.md          # Phase 0: technology decisions
├── data-model.md        # Phase 1: SO schemas and runtime state
├── quickstart.md        # Phase 1: end-to-end validation guide
├── contracts/           # Phase 1: EventBus events + command interface
│   ├── eventbus-events.md
│   └── command-interface.md
└── tasks.md             # Phase 2 output (/speckit.tasks — NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
Assets/
├── Data/                          # SO instances (authored data)
│   ├── Knights/                   # Archetype, HautFait, Blason SOs
│   ├── Tarot/                     # 22 TarotCard SOs
│   ├── Weapons/                   # WeaponProfile SOs (8 demo + grenades)
│   ├── Enemies/                   # EnemyBase SOs (5 types + boss phases)
│   ├── Encounters/                # EncounterSO instances (5 nodes)
│   ├── Motivations/               # Minor (10) + Major (5) template SOs
│   ├── Injuries/                  # InjuryTable SO (4×6 matrix)
│   └── Missions/                  # MissionDefinition SOs
├── Sprites/                       # 2D programmer art (placeholder)
│   ├── Knights/                   # Per armor class (idle/attack/hit/death)
│   ├── Enemies/                   # Per enemy type
│   ├── UI/                        # Icons, portraits, badges
│   └── VFX/                       # Particle textures
├── UI/                            # UI Toolkit assets
│   ├── UXML/                      # Layout documents
│   ├── USS/                       # Stylesheets
│   └── Templates/                 # Reusable UI components
├── Input/
│   └── CombatActions.inputactions # Combat action map (SPEC-32, research.md §5)
└── Scenes/
    └── Main.unity                 # Single scene (SPEC-32)

Scripts/
├── Core/                          # Cross-cutting systems
│   ├── GameStateMachine.cs        # MainMenu→Hub→Squad→Deploy→Combat→NodeTransition→MissionEnd
│   ├── EventBus.cs                # SPEC-27 typed event aggregator
│   ├── RNG/
│   │   ├── IRng.cs                # Interface (SPEC-28)
│   │   ├── Xoshiro128StarStar.cs  # PRNG implementation
│   │   ├── RngFactory.cs          # NodeSeed derivation, stream creation
│   │   └── StableStringHash.cs    # FNV-1a
│   ├── RunState.cs                # Persistent state across Hub↔Combat (SPEC-32)
│   └── Constants/
│       └── Enums.cs               # ALL enums from SPEC-14
├── Data/                          # SO definitions (schemas)
│   ├── KnightBaseSO.cs            # SPEC-01
│   ├── ArchetypeDataSO.cs         # SPEC-02.9
│   ├── HautFaitDataSO.cs          # SPEC-02.8
│   ├── BlasonDataSO.cs            # SPEC-02.5
│   ├── TarotCardSO.cs             # SPEC-24
│   ├── WeaponProfileSO.cs         # SPEC-19
│   ├── EnemyBaseSO.cs             # SPEC-18
│   ├── EncounterSO.cs             # SPEC-29
│   ├── MotivationTemplateSO.cs    # SPEC-12.1
│   ├── InjuryTableSO.cs           # SPEC-23
│   └── MissionDefinitionSO.cs     # SPEC-22
├── Combat/                        # All combat runtime systems
│   ├── CombatManager.cs           # Single source of truth (SPEC-32)
│   ├── CombatSession.cs           # Ephemeral session state (SPEC-32)
│   ├── TurnQueue.cs               # SPEC-15 FFX initiative
│   ├── PositionManager.cs         # SPEC-03 rank system
│   ├── Resolvers/
│   │   ├── ComboRollResolver.cs   # SPEC-04
│   │   ├── DamageResolver.cs      # SPEC-05
│   │   ├── ArmourLayerResolver.cs # SPEC-06
│   │   ├── ViolenceResolver.cs    # SPEC-05 (violence chain)
│   │   ├── WeaponEffectResolver.cs# SPEC-20
│   │   └── DispersionResolver.cs  # SPEC-20.5
│   ├── State/
│   │   ├── CombatantState.cs      # SPEC-30
│   │   ├── EspoirSystem.cs        # SPEC-07
│   │   ├── InjurySystem.cs        # SPEC-08/23
│   │   ├── HeroismSystem.cs       # SPEC-09
│   │   ├── FoldStateSystem.cs     # SPEC-10
│   │   └── NodSystem.cs           # SPEC-33
│   ├── Abilities/
│   │   ├── WarriorTypeSystem.cs   # SPEC-17.1
│   │   ├── PaladinShrineSystem.cs # SPEC-17.2
│   │   ├── PaladinWatchtowerSystem.cs # SPEC-17.2
│   │   ├── PriestNanoCSystem.cs   # SPEC-17.3
│   │   ├── PriestMechanicSystem.cs# SPEC-17.3
│   │   └── RogueGhostSystem.cs    # SPEC-17.4
│   ├── AI/
│   │   ├── EnemyAIController.cs   # SPEC-18.7/18.11
│   │   ├── HybridAIPattern.cs     # SPEC-18.12
│   │   ├── DespairAIController.cs # SPEC-07 (hostile knight AI)
│   │   └── BandeController.cs     # SPEC-11
│   ├── BossPhaseController.cs     # SPEC-14
│   └── MotivationDetector.cs      # SPEC-12
├── Meta/                          # Hub, progression, save
│   ├── CamelotHubManager.cs       # SPEC-25
│   ├── MissionManager.cs          # SPEC-22
│   ├── SaveSystem.cs              # SPEC-13
│   ├── PGEconomyManager.cs        # SPEC-25.3
│   └── GenerationPipeline.cs      # SPEC-02 (8-knight procedural gen)
└── UI/                            # Presentation layer (read-only)
    ├── Combat/
    │   ├── CombatHUDController.cs
    │   ├── InitiativeTimelineView.cs
    │   ├── RollBreakdownPanel.cs
    │   ├── CombatLogView.cs
    │   ├── ActionMenuController.cs
    │   ├── RankDisplayView.cs
    │   └── CountdownTimerView.cs  # Hémorragie, Despair, DoT
    ├── Hub/
    │   ├── RosterView.cs
    │   ├── InfirmaryPanel.cs
    │   ├── MissionSelectView.cs
    │   └── MemorialView.cs
    ├── Deployment/
    │   └── DeploymentScreenController.cs
    ├── Shared/
    │   ├── DamageNumberPopup.cs
    │   └── TooltipSystem.cs
    └── MainMenuScreen.cs          # New Game, Continue, Quit

Tests/
├── EditMode/                      # Headless unit + integration
│   ├── RNGDeterminismTests.cs     # SPEC-28-AC1, AC5
│   ├── KnightGenerationTests.cs   # SPEC-02 acceptance criteria
│   ├── ComboRollTests.cs          # SPEC-04 all ACs
│   ├── ArmourLayerTests.cs        # SPEC-06 all worked examples
│   ├── DamageResolverTests.cs     # SPEC-05 all ACs
│   ├── EspoirSystemTests.cs      # SPEC-07 all ACs
│   ├── InjurySystemTests.cs       # SPEC-08/23 all ACs
│   ├── NodSystemTests.cs          # SPEC-33 all ACs
│   ├── BandeControllerTests.cs    # SPEC-11 all ACs
│   ├── BossPhaseTests.cs          # SPEC-14 all ACs
│   ├── IntegrationScenario1.cs    # Fold→Despair→Hémorragie cascade
│   ├── IntegrationScenario2.cs    # Ghost alpha-strike + Exploit + dual-wield
│   ├── IntegrationScenario3.cs    # Barrage + boss phase transition
│   ├── IntegrationScenario4.cs    # Grenade friendly fire + Despair + stabilization
│   └── IntegrationScenario5.cs    # Full node transition attrition
└── PlayMode/                      # With UI (smoke tests)
    └── FullMissionSmokeTest.cs
```

**Structure Decision**: Unity game project with single scene architecture (SPEC-32). Source follows the recommended `Scripts/Core`, `Scripts/Combat`, `Scripts/Data`, `Scripts/UI`, `Scripts/Meta` layout from SPEC-32. All authored data in `Assets/Data/` as SO instances. Tests split EditMode (headless logic) and PlayMode (UI smoke). Structure directly maps from `Analysis/DemoImplementationPlan_v1.md` §3.

## Milestone Alignment

This plan aligns with the 12-milestone, 16-week structure from `/Analysis/DemoImplementationPlan_v1.md`. Each milestone maps to the SPEC-16 dependency phases:

| Milestone | Phase | Weeks | Key Deliverable | SPEC References |
|-----------|-------|-------|----------------|-----------------|
| M0: Bootstrap | — | 1 | RNG, EventBus, GameStateMachine, enums | SPEC-14, 27, 28, 32 |
| M1: Data Layer | 1 | 2–3 | All SO schemas + 60+ instances | SPEC-01, 02, 18, 19, 20, 24 |
| M2: Position & Turns | 2 | 4 | Rank system, cover, movement, initiative | SPEC-03, 15 |
| M3: Core Combat ★ | 3–4 | 5–7 | ComboRoll, Damage, ArmourLayer (vertical slice) | SPEC-04, 05, 06, 07, 33 |
| M4: Consequences | 5 | 8 | Injury, Agony, Heroism, Fold, Despair | SPEC-08, 09, 10, 23 |
| M5: Armor Abilities | 3b | 8 (parallel) | Warrior Types, Paladin, Priest, Rogue | SPEC-17 |
| M6: Entities | 6 | 9 | Enemy AI, Bande, Motivation | SPEC-11, 12, 18, 21 |
| M7: Mission & Save | 7 | 10–11 | Boss phases, save, mission graph, events | SPEC-13, 14, 22 |
| M8: Camelot Hub | 8 | 12 | Hub state, infirmary, PG economy, squad select | SPEC-25 |
| M9: UI Layer | — | 10–14 (parallel) | Combat HUD, action menu, hub screens | SPEC-26, 27, 31 |
| M10: Art Direction | — | 6–14 (parallel) | Programmer art sprites, VFX placeholders | — |
| M11: Integration Tests | — | 13–15 | 5 integration scenarios, quality checks | SPEC-15 (tests) |
| M12: Ship | — | 16–18 | Full playthrough, balance, build | — |

**Critical Path**: M0 → M1 → M2 → M3 (★ vertical slice) → M4/M5 → M6 → M7 → M8 → M11 → M12

**Vertical Slice Gate (End of Week 7)**: Headless combat test — generate 4 knights, deploy vs enemies, full turn cycle with ComboRoll → DamageResolver → ArmourLayerResolver, Nod usage, enemy death, combat end. If green, core game works.

## Key Architecture Decisions

### A.1 Single Scene (SPEC-32)
One Unity scene: `Main.unity`. `GameStateMachine` manages all state transitions. No `SceneManager.LoadScene()` in combat code. UI panels activated/deactivated by state machine.

### A.2 State Isolation (SPEC-32)
- **RunState**: Persistent (survives combat exit). Owns: knight roster, PG, injuries, mission progress.
- **CombatSession**: Ephemeral (created at deployment, destroyed on exit). Owns: turn queue, CombatantState instances, active effects, RNG streams.
- **ApplyCombatResults**: Atomic commit from CombatSession → RunState on combat exit. Crash = no corruption.
- **Hémorragie exception**: Only combat state that crosses to RunState between encounters.

### A.3 CombatManager as Authority (SPEC-32)
Single `CombatManager` owns: turn queue, legality checks, resolution, effect application, victory/defeat. UI sends commands → CombatManager validates → executes → fires events → UI subscribes.

### A.4 Deterministic RNG (SPEC-28)
Xoshiro128** PRNG. xxHash32 seed derivation. FNV-1a string hashing. 3 streams (Combat, Loot, Cosmetic). `UnityEngine.Random` forbidden in all gameplay code.

### A.5 French/English Naming (SPEC-32)
- Enums: French (`AGRESSIF`, `DEFENSIF`, `SOIN`)
- Fields/Variables: English (`isInAgony`, `currentPS`)
- Display/UI strings: French
- Classes: English (`KnightBase`, `BandeController`)

## Stub vs Full Implementation Matrix

| System | Level | Justification |
|--------|-------|---------------|
| ArmourLayerResolver | FULL | Central damage pipeline |
| ComboRollResolver | FULL | Player-facing, every edge case matters |
| Deterministic RNG | FULL | Architectural foundation |
| Enemy AI | FULL | Core gameplay — believable enemies |
| All 4 armor class abilities | FULL | Each class must feel distinct |
| Nod system | FULL | Primary recovery mechanic |
| Espoir/Despair | FULL | Key narrative/mechanical tension |
| Boss phase controller | FULL | Demo climax |
| YES-scoped weapon effects (~20) | FULL | Used by demo weapons/enemies |
| SCAFFOLDED weapon effects (~10) | STUB | No demo weapon uses them |
| Modules (equippedModules) | STUB | Out of demo scope |
| Armor Evolutions (150/200/250 PG) | STUB | Scaffolded. Lock UI buttons |
| Mode Héroïque objectives | STUB | Simplified "persist until 0 PS/PEs" |
| Points de Contact | STUB | SPEC says do NOT compute for demo |
| Camelot weapon shop | STUB | Default loadouts only — no task needed, knights use preset equipment |
| Audio/VFX | STUB | Fire SPEC-27 events, attach SFX later |

## Generated Artifacts

| Artifact | Path | Description |
|----------|------|-------------|
| research.md | [research.md](./research.md) | Phase 0: all technology decisions resolved |
| data-model.md | [data-model.md](./data-model.md) | Phase 1: SO schemas mapped from SPEC-01/18/19 |
| eventbus-events.md | [contracts/eventbus-events.md](./contracts/eventbus-events.md) | Phase 1: EventBus event definitions from SPEC-27 |
| command-interface.md | [contracts/command-interface.md](./contracts/command-interface.md) | Phase 1: UI→CombatManager command contracts from SPEC-32 |
| quickstart.md | [quickstart.md](./quickstart.md) | Phase 1: end-to-end validation guide |

## Complexity Tracking

No constitution violations to justify. The plan follows all 9 principles without deviation.
