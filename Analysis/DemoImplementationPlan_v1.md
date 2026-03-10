# DarkestKnight — Unity Demo Implementation Plan v1.0

**Date:** March 10, 2026  
**Author:** Game Design & Architecture Lead  
**Target:** Playable 2D tactical RPG demo (1 mission, 5 nodes, 5 enemy types, 2-phase boss)  
**Team:** 2–4 developers  
**Engine:** Unity 6 LTS (2023.3+)  
**Estimated Timeline:** 16–18 weeks  

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Technology Stack & Packages](#2-technology-stack--packages)
3. [Project Structure](#3-project-structure)
4. [Milestone 0 — Project Bootstrap (Week 1)](#milestone-0--project-bootstrap-week-1)
5. [Milestone 1 — Phase 1: Data Layer (Weeks 2–3)](#milestone-1--phase-1-data-layer-weeks-23)
6. [Milestone 2 — Phase 2: Position & Turns (Week 4)](#milestone-2--phase-2-position--turns-week-4)
7. [Milestone 3 — Phases 3–4: Core Combat + Damage (Weeks 5–7) ★ VERTICAL SLICE](#milestone-3--phases-34-core-combat--damage-weeks-57--vertical-slice)
8. [Milestone 4 — Phase 5: Consequences (Week 8)](#milestone-4--phase-5-consequences-week-8)
9. [Milestone 5 — Phase 3b: Armor Abilities (Week 8, parallel)](#milestone-5--phase-3b-armor-abilities-week-8-parallel)
10. [Milestone 6 — Phase 6: Entities (Week 9)](#milestone-6--phase-6-entities-week-9)
11. [Milestone 7 — Phase 7: Mission & Save (Weeks 10–11)](#milestone-7--phase-7-mission--save-weeks-1011)
12. [Milestone 8 — Phase 8: Camelot Hub (Week 12)](#milestone-8--phase-8-camelot-hub-week-12)
13. [Milestone 9 — UI Layer (Weeks 10–14, parallel)](#milestone-9--ui-layer-weeks-1014-parallel)
14. [Milestone 10 — 2D Art Direction & Rendering (Weeks 6–14, parallel)](#milestone-10--2d-art-direction--rendering-weeks-614-parallel)
15. [Milestone 11 — Integration Tests & Polish (Weeks 13–15)](#milestone-11--integration-tests--polish-weeks-1315)
16. [Milestone 12 — Full Playthrough & Ship (Weeks 16–18)](#milestone-12--full-playthrough--ship-weeks-1618)
17. [Critical Path & Risk Assessment](#critical-path--risk-assessment)
18. [Stub vs Full Implementation Matrix](#stub-vs-full-implementation-matrix)
19. [Developer Role Assignment](#developer-role-assignment)
20. [Time Estimate Summary](#time-estimate-summary)

---

## 1. Executive Summary

DarkestKnight is a 2D tactical RPG adapted from the French tabletop RPG **"Knight"**. The demo features:

- **8 procedurally generated knights** across 4 armor classes (Warrior, Paladin, Priest, Rogue)
- **4 deployed per mission** on a Darkest Dungeon-style linear rank track (Rank 1–4 per side)
- **FFX-style initiative turn queue** with complex D6-based combat
- **Multi-layer armor system** (CdF → Bouclier → PA → PS) with 30+ weapon effects
- **5 enemy types** spanning all tiers (Nocte bande → Bestian → Faune → Behemot → Ours Corrompu boss)
- **1 demo mission** with 5 nodes (linear-with-branch), final boss has 2 phases
- **Camelot Hub** for between-mission roster management
- **Roguelite attrition** — no mid-combat save, no between-node healing (Nods only)
- **Deterministic RNG** per node for reproducible outcomes

The plan follows the canonical **SPEC-16 8-phase dependency order** and targets a **vertical slice** (headless core combat loop) by **Week 7**.

---

## 2. Technology Stack & Packages

### Unity Packages (Required)

| Package | Purpose | SPEC Reference |
|---------|---------|----------------|
| `Unity.Mathematics` | xxHash32 seed derivation (`math.hash()`), math utilities | SPEC-28 |
| `TextMeshPro` | Rich text for combat log, damage numbers, UI labels | SPEC-31 |
| `Unity Input System` | Modern input handling for combat actions | SPEC-32 |
| `Unity Test Framework` + `NUnit` | Headless EditMode/PlayMode tests | SPEC-32-AC1, SPEC-15 |
| `Unity 2D Animation` or `Spine Runtime` | Sprite animation for combat entities | Art direction |

### Unity Packages (Recommended)

| Package | Purpose | Notes |
|---------|---------|-------|
| `Addressables` | Async SO loading for encounters/missions | Optional for demo; helpful at scale |
| `Unity Localization` | French string management | Future-proofing; demo uses hardcoded French strings |
| `DOTween` (Asset Store) | Tweening for UI transitions, damage floats | Free/Pro; lighter than Animator for UI |

### UI Framework Decision

> **Recommendation: UI Toolkit**
> 
> UI Toolkit's UXML/USS architecture naturally enforces the SPEC-32 rule: **"UI is read-only view + command sender, never mutates combat state directly."** Its data-binding system cleanly separates presentation from logic. The CSS-like USS styling also accelerates iteration on the dark gothic aesthetic.
> 
> **Fallback:** uGUI is acceptable if the team has no UI Toolkit experience. Do NOT mix both in the same project.

### RNG Implementation

| Component | Choice | Reference |
|-----------|--------|-----------|
| PRNG Algorithm | **Xoshiro128\*\*** | SPEC-28 (prng.di.unimi.it) |
| Seed Derivation | **xxHash32** via `math.hash()` | SPEC-28, cross-platform deterministic |
| String Hashing | **FNV-1a** (stable across .NET runtimes) | SPEC-28 (`StableStringHash`) |
| Seed Type | `uint` (unsigned 32-bit throughout) | SPEC-28 |
| Forbidden | `UnityEngine.Random` in ANY gameplay code | SPEC-28-AC3, SPEC-32-AC3 |

---

## 3. Project Structure

Aligned with SPEC-32 recommended structure:

```
Assets/
├── Data/                          ← SO instances (authored data)
│   ├── Knights/                   ← Archetype, HautFait, Blason SOs
│   ├── Tarot/                     ← 22 TarotCard SOs
│   ├── Weapons/                   ← WeaponProfile SOs (8 demo + grenades)
│   ├── Enemies/                   ← EnemyBase SOs (5 types + boss phases)
│   ├── Encounters/                ← EncounterSO instances (5 nodes)
│   ├── Motivations/               ← Minor (10) + Major (5) template SOs
│   ├── Injuries/                  ← InjuryTable SO (4×6 matrix)
│   └── Missions/                  ← MissionDefinition SOs
├── Sprites/                       ← 2D art (placeholder → final)
│   ├── Knights/                   ← Per armor class (idle/attack/hit/death)
│   ├── Enemies/                   ← Per enemy type
│   ├── UI/                        ← Icons, portraits, badges
│   └── VFX/                       ← Particle textures
├── UI/                            ← UI Toolkit assets
│   ├── UXML/                      ← Layout documents
│   ├── USS/                       ← Stylesheets
│   └── Templates/                 ← Reusable UI components
├── Audio/                         ← SFX/Music (placeholder)
└── Scenes/
    └── Main.unity                 ← Single scene (SPEC-32)

Scripts/
├── Core/                          ← Cross-cutting systems
│   ├── GameStateMachine.cs        ← MainMenu→Hub→Squad→Deploy→Combat→NodeTransition→MissionEnd
│   ├── EventBus.cs                ← SPEC-27 event aggregator
│   ├── RNG/
│   │   ├── IRng.cs                ← Interface (SPEC-28)
│   │   ├── Xoshiro128StarStar.cs  ← PRNG implementation
│   │   ├── RngFactory.cs          ← NodeSeed derivation, stream creation
│   │   └── StableStringHash.cs    ← FNV-1a
│   ├── RunState.cs                ← Persistent state across Hub↔Combat (SPEC-32)
│   └── Constants/
│       └── Enums.cs               ← ALL enums from SPEC-14
├── Data/                          ← SO definitions (schemas)
│   ├── KnightBaseSO.cs            ← SPEC-01
│   ├── ArchetypeDataSO.cs         ← SPEC-02.9
│   ├── HautFaitDataSO.cs          ← SPEC-02.8
│   ├── BlasonDataSO.cs            ← SPEC-02.5
│   ├── TarotCardSO.cs             ← SPEC-24
│   ├── WeaponProfileSO.cs         ← SPEC-19
│   ├── EnemyBaseSO.cs             ← SPEC-18
│   ├── EncounterSO.cs             ← SPEC-29
│   ├── MotivationTemplateSO.cs    ← SPEC-12.1
│   ├── InjuryTableSO.cs           ← SPEC-23
│   └── MissionDefinitionSO.cs     ← SPEC-22
├── Combat/                        ← All combat runtime systems
│   ├── CombatManager.cs           ← Single source of truth (SPEC-32)
│   ├── CombatSession.cs           ← Ephemeral session state (SPEC-32)
│   ├── TurnQueue.cs               ← SPEC-15 FFX initiative
│   ├── PositionManager.cs         ← SPEC-03 rank system
│   ├── Resolvers/
│   │   ├── ComboRollResolver.cs   ← SPEC-04
│   │   ├── DamageResolver.cs      ← SPEC-05
│   │   ├── ArmourLayerResolver.cs ← SPEC-06
│   │   ├── ViolenceResolver.cs    ← SPEC-05 (violence chain)
│   │   ├── WeaponEffectResolver.cs← SPEC-20
│   │   └── DispersionResolver.cs  ← SPEC-20.5
│   ├── State/
│   │   ├── CombatantState.cs      ← SPEC-30
│   │   ├── EspoirSystem.cs        ← SPEC-07
│   │   ├── InjurySystem.cs        ← SPEC-08/23
│   │   ├── HeroismSystem.cs       ← SPEC-09
│   │   ├── FoldStateSystem.cs     ← SPEC-10
│   │   └── NodSystem.cs           ← SPEC-33
│   ├── Abilities/
│   │   ├── WarriorTypeSystem.cs   ← SPEC-17.1
│   │   ├── PaladinShrineSystem.cs ← SPEC-17.2
│   │   ├── PaladinWatchtowerSystem.cs ← SPEC-17.2
│   │   ├── PriestNanoCSystem.cs   ← SPEC-17.3
│   │   ├── PriestMechanicSystem.cs← SPEC-17.3
│   │   └── RogueGhostSystem.cs    ← SPEC-17.4
│   ├── AI/
│   │   ├── EnemyAIController.cs   ← SPEC-18.7/18.11
│   │   ├── HybridAIPattern.cs     ← SPEC-18.12
│   │   ├── DespairAIController.cs ← SPEC-07 (hostile knight AI)
│   │   └── BandeController.cs     ← SPEC-11
│   ├── BossPhaseController.cs     ← SPEC-14
│   └── MotivationDetector.cs      ← SPEC-12
├── Meta/                          ← Hub, progression, save
│   ├── CamelotHubManager.cs       ← SPEC-25
│   ├── MissionManager.cs          ← SPEC-22
│   ├── SaveSystem.cs              ← SPEC-13
│   ├── PGEconomyManager.cs        ← SPEC-25.3
│   └── GenerationPipeline.cs      ← SPEC-02 (8-knight procedural gen)
├── UI/                            ← Presentation layer (read-only)
│   ├── Combat/
│   │   ├── CombatHUDController.cs
│   │   ├── InitiativeTimelineView.cs
│   │   ├── RollBreakdownPanel.cs
│   │   ├── CombatLogView.cs
│   │   ├── ActionMenuController.cs
│   │   ├── RankDisplayView.cs
│   │   └── CountdownTimerView.cs  ← Hémorragie, Despair, DoT
│   ├── Hub/
│   │   ├── RosterView.cs
│   │   ├── InfirmaryPanel.cs
│   │   ├── MissionSelectView.cs
│   │   └── MemorialView.cs
│   ├── Deployment/
│   │   └── DeploymentScreenController.cs
│   └── Shared/
│       ├── DamageNumberPopup.cs
│       └── TooltipSystem.cs

Tests/
├── EditMode/                      ← Headless (no UI) unit + integration
│   ├── RNGDeterminismTests.cs     ← SPEC-28-AC1, AC5
│   ├── KnightGenerationTests.cs   ← SPEC-02 acceptance criteria
│   ├── ComboRollTests.cs          ← SPEC-04 all ACs
│   ├── ArmourLayerTests.cs        ← SPEC-06 all worked examples
│   ├── DamageResolverTests.cs     ← SPEC-05 all ACs
│   ├── EspoirSystemTests.cs      ← SPEC-07 all ACs
│   ├── InjurySystemTests.cs       ← SPEC-08/23 all ACs
│   ├── NodSystemTests.cs          ← SPEC-33 all ACs
│   ├── BandeControllerTests.cs    ← SPEC-11 all ACs
│   ├── BossPhaseTests.cs          ← SPEC-14 all ACs
│   ├── IntegrationScenario1.cs    ← Fold→Despair→Hémorragie cascade
│   ├── IntegrationScenario2.cs    ← Ghost alpha-strike + Exploit + dual-wield
│   ├── IntegrationScenario3.cs    ← Barrage + boss phase transition
│   ├── IntegrationScenario4.cs    ← Grenade friendly fire + Despair + stabilization
│   └── IntegrationScenario5.cs    ← Full node transition attrition
└── PlayMode/                      ← With UI (smoke tests)
    └── FullMissionSmokeTest.cs
```

---

## Milestone 0 — Project Bootstrap (Week 1)

**Developers:** 1–2  
**Goal:** Foundational infrastructure that ALL other systems depend on.

### Tasks

| # | Task | SPEC | Acceptance Criteria |
|---|------|------|---------------------|
| 0.1 | Create Unity project, install packages (`Mathematics`, `TMP`, `Input System`, `Test Framework`) | SPEC-32 | Project compiles cleanly |
| 0.2 | Implement `GameStateMachine` — explicit states: `MainMenu → CamelotHub → SquadSelect → Deployment → Combat → NodeTransition → MissionEnd` | SPEC-32 | State transitions work, no scene loads in combat (SPEC-32-AC4) |
| 0.3 | Implement SPEC-28 deterministic RNG: `Xoshiro128**`, `xxHash32` via `math.hash()`, `StableStringHash` (FNV-1a), `IRng` interface, 3-stream separation | SPEC-28 | SPEC-28-AC1: Same seed → identical sequence. SPEC-28-AC5: Cross-implementation determinism |
| 0.4 | Define ALL enums and constants from `14_Constants.md` | SPEC-14 | Every enum compiles, matches spec exactly |
| 0.5 | Set up `EventBus` for SPEC-27 hooks (typed C# events) | SPEC-27 | Events fire, multiple subscribers work |
| 0.6 | Scaffold `RunState` (persistent) and `CombatSession` (ephemeral) with field boundary from SPEC-32 | SPEC-32 | Clear separation, `ApplyCombatResults` method stub |
| 0.7 | Set up EditMode test project, first test: RNG determinism | SPEC-28 | Green tests |

### Deliverable
- Compilable Unity project with core infrastructure
- Deterministic RNG proven by automated test
- All enums available for downstream agents

---

## Milestone 1 — Phase 1: Data Layer (Weeks 2–3)

**Developers:** 2 (parallel — Data Agent + Weapon & Content Agent)  
**Goal:** All ScriptableObject schemas defined and demo instances authored.

### Data Agent Tasks

| # | Task | SPEC | Key Details |
|---|------|------|-------------|
| 1.1 | `KnightBaseSO` schema | SPEC-01 | 5 Aspects, 15 Characteristics (`int[15]`), derived values (Defense, Reaction, Initiative, maxPS, maxPEs), meta-armor fields, equipment, combat state, Nod inventory |
| 1.2 | Derived value computation functions | SPEC-01 | `Defense = highestCharIn(BETE) + odOf(that)`, `Reaction = highestCharIn(MACHINE) + odOf(that)`, `Initiative = highestCharIn(MASQUE) + odOf(that)`, `maxPS = 10 + 6 × Max(Force,Endurance,Deplacement)`, `maxPEs = 50 + tarotMods - (implants×3)` |
| 1.3 | `TarotCardSO` — 22 instances | SPEC-24 | `linkedAspect`, `advantage`, `disadvantage`, `charPoints`, weighted draw algorithm (weight 3 for primary aspects), Amnésique post-processing, incompatibility (XIII + XVI) |
| 1.4 | `ArchetypeDataSO` — 17 instances | SPEC-02.9 | `bonus1`, `bonus2`, `choicePool`, `choiceBonusSlot`. Choice archetypes (#6,#7,#8) = 50/50 random |
| 1.5 | `HautFaitDataSO` — 14 instances | SPEC-02.8 | Dual-aspect resolution (#4, #11, #12) with armor class weighting (75/25 or 50/50). Condition checking (ASPECT or EITHER_CHAR) |
| 1.6 | `BlasonDataSO` — 9 instances (Le Dragon deferred) | SPEC-02.5 | `voeuEventTag`, uniqueness check vs drawn MN tags |
| 1.7 | `GenerationPipeline` — full 10-step algorithm | SPEC-02 | Base stats → Archetype → Tarot (weighted draw, 5 cards, distribute points using Weighted Distribution Table) → Haut Fait → Blason → Motivations → Meta-Armor → OD → Equipment → Derived values |
| 1.8 | Unit tests: SPEC-01 all ACs, SPEC-02 seed determinism | SPEC-01/02 | Same seed → same 8 knights |

### Weapon & Content Agent Tasks

| # | Task | SPEC | Key Details |
|---|------|------|-------------|
| 1.9 | `WeaponProfileSO` schema + 8 demo weapons + grenades | SPEC-19 | `DamageProfile[]`, `ProfileSwitchCost`, range → max gap mapping, grenade types (5) |
| 1.10 | `WeaponEffectId` tag system + effect resolution stubs | SPEC-20 | Full implementation for YES-scoped effects, no-op stubs for SCAFFOLDED |
| 1.11 | `EnemyBaseSO` schema + 5 demo enemies | SPEC-18/21 | Nocte, Bestian, Faune, Behemot, Ours Corrompu (P1+P2). Pre-bake Exceptionnel bonuses. `EnemyAI` profiles per stat blocks |
| 1.12 | `EncounterSO` schema + 5 mission node encounters | SPEC-29 | Per-node cover layouts, initial enemies, reinforcement queues, bande config, triggers (boss phase nocte respawn) |
| 1.13 | `InjuryTableSO` — 4×6 matrix | SPEC-23 | Column 1–4 (Catastrophic → Light), Row 1–6. Each entry: `injuryId`, `effects[]`, `isPermanent` |
| 1.14 | `MotivationTemplateSO` — 10 minor + 5 major | SPEC-12.1 | `eventTag`, `type`, `displayText` |

### Milestone 1 Deliverable
- All SO schemas defined with authoring validation
- All 60+ SO instances authored and serialized
- 8 procedurally generated knights from a test seed

---

## Milestone 2 — Phase 2: Position & Turns (Week 4)

**Developers:** 1 (Combat Core Agent)  
**Goal:** Battlefield spatial system and turn ordering.

### Tasks

| # | Task | SPEC | Key Details |
|---|------|------|-------------|
| 2.1 | `PositionManager` — 4-rank linear track per side | SPEC-03 | Gap formula: `(aRank-1)+(tRank-1)`, range band → max gap validation, rank modifiers (R1: +1 Def, R4: −1 Def/−1 React) |
| 2.2 | Cover system | SPEC-03 | `hasCover` per rank, +3 Reaction vs ranged, −3 dice from cover, Tir en Sécurité bypass, enemy cover symmetry, Cover Removal Procedure |
| 2.3 | Movement system | SPEC-03 | Swap with adjacent ally (1 Move Action), move into empty rank (1 Move Action), 2-rank move (spend Combat Action), forced displacement (chain-swap rule, push into occupied = swap) |
| 2.4 | Death & empty ranks | SPEC-03 | Knight death = permanent empty rank, enemy reinforcement spawn rules (queue at turn start, initiative insertion) |
| 2.5 | `TurnQueue` — FFX-style initiative | SPEC-15 | `3D6 + Initiative` (once per encounter), Trop Prudent = 2D6, Bandes = 1, Masque Majeur = 30, ties: higher score first → random. Delay mechanic |
| 2.6 | Turn structure enforcement | SPEC-15/04 | 1 Combat + 1 Movement per turn, Combat→Movement substitution (free), action tracking |
| 2.7 | Deployment phase logic | SPEC-03 | Player assigns 4 knights to Ranks 1–4, default suggestion (W→R1, Pa→R2, Pr→R3, Ro→R4), enemy placement from EncounterSO |

### Milestone 2 Deliverable
- Headless position/initiative test: deploy 4 knights + 2 enemies, roll initiative, verify turn order, move/swap/attack range checks

---

## Milestone 3 — Phases 3–4: Core Combat + Damage (Weeks 5–7) ★ VERTICAL SLICE

**Developers:** 2 (Combat Core Agent + Combat State Agent start)  
**Goal:** Complete combat resolution pipeline. Prove the core loop works headless.

### Combat Core Agent Tasks

| # | Task | SPEC | Key Details |
|---|------|------|-------------|
| 3.1 | `ComboRollResolver` | SPEC-04 | Build pool (Base + Combo + style mods + PEs penalty + injury penalties), floor at 0 = auto-fail, roll D6s, count evens, add OD auto-successes. Exploit (all even → reroll once, +1 Héroïsme/+1 PEs), Failure Critique (all odd), pool=0 = no OD |
| 3.2 | Style modifier system | SPEC-04 | 9 styles: Standard, Agressif (+3 dice, −2 Def/React), Défensif (−3 dice, +2 Def), Mise à couvert (−3, +2 React, requires cover), Puissant (−N, sacrifice → bonus dmg dice, requires Lourd), Pilonnage (−2, accumulating bonus, requires Deux Mains ranged), Précis (3rd char, costs full turn), Ambidextre, Akimbo |
| 3.3 | Dual-wield system | SPEC-04 | Ambidextre: 2 separate attacks, −3 dice (Jumelé → −1). Akimbo: 1 combined, −3 dice (Jumelé → −1). Suppress with both weapons for 1 Combat Action |
| 3.4 | Assistance system | SPEC-04 | Up to 3 allies, 1 Movement Action each, unique Characteristic per assistant, Solitaire +1 threshold, PEs penalty per assistant |
| 3.5 | Action economy enforcement | SPEC-04 | Full action cost table. Profile switch costs (Free vs Movement). Weapon switch = 1 Movement. Style switch = Free |
| 3.6 | `DamageResolver` — 8-step chain | SPEC-05 | Hit check → roll weapon dice (SUM) → flat bonus → Force/Précision/Orfèvrerie/Ghost bonuses → Puissant/Pilonnage dice → Dégâts Maximum → route to armour. Violence chain (SUM → Cohesion for bandes). Excess successes (none by default, Assistance à l'Attaque +1/excess, Mode Héroïque → extra D6) |
| 3.7 | `ArmourLayerResolver` — 8-step canonical | SPEC-06 | CdF (Pénétrant X, Ignore CdF) → Bouclier → Chair Mineur flat reduction → Destructeur +2D6 check → PA (Perce Armure X) + passthrough (`floor(paAbsorbed/5)`, Infatigable override) → Meurtrier +2D6 check (Chair Majeur immune) → PS → Agony trigger |
| 3.8 | Anathème damage route | SPEC-06 | CdF → remainder hits PEs (PA bypassed). PEs Recovery on Anathème Kill |
| 3.9 | Colosse 10:1 rule | SPEC-18.5 | Anti-Véhicule bypasses. `floor(rawDamage / 10)` effective |
| 3.10 | Weapon effect resolver (YES-scoped effects) | SPEC-20 | Barrage X (Suppress action), Choc X (standard + auto variants, Chair Majeur immune), Dispersion X (8-position linear track model, friendly fire), Dégâts Continus X (DoT, no stacking), Lumière X (vs Hypersensibilité), Parasitage X, Ultraviolence (reroll 1-2 on violence), Perce Armure, Pénétrant, Destructeur, Meurtrier, all bypass effects |

### Combat State Agent Tasks (starting Week 6)

| # | Task | SPEC | Key Details |
|---|------|------|-------------|
| 3.11 | `EspoirSystem` | SPEC-07 | `pesPenalty = max(0, 10 − currentPEs)`, state table (Résolu→Désespoir), loss/recovery source tracking |
| 3.12 | `NodSystem` | SPEC-33 | 3 types, 3D6 restore, Guérison Rapide +3, Agony targeting exception (Soin only), Fold PE freeze (Énergie blocked), between-node free use |

### ★ Vertical Slice Acceptance Criteria (End of Week 7)

Run headless (no UI) combat test via EditMode:
1. Generate 4 knights from seed
2. Deploy vs 2 Bestians
3. Initiative rolled, turns cycle
4. Knight attacks → ComboRoll → DamageResolver → ArmourLayerResolver → enemy PS reduced
5. Enemy attacks back → Anathème route → PEs reduction → PEs penalty applies
6. Knight uses Nod de Soin → PS restored
7. Enemy dies → combat ends
8. Console log shows full resolution at each step

> **If this milestone is green, the core game works. Everything else is layered on top.**

---

## Milestone 4 — Phase 5: Consequences (Week 8)

**Developers:** 1 (Combat State Agent)

### Tasks

| # | Task | SPEC | Key Details |
|---|------|------|-------------|
| 4.1 | `InjurySystem` — full 4×6 table | SPEC-08/23 | Severity D6 → column (1=Catastrophic, 2-3=Severe, 4-5=Moderate, 6=Light), Row D6. Apply stat reductions, recalculate derived values. Trompe la Mort reroll (once/mission) |
| 4.2 | Agony state | SPEC-08 | PS=0 → Injury roll → Incapacitated. Ignorer l'Agonie (1 Héroïsme, once/combat, stay at 1 PS, no injury roll) |
| 4.3 | Hémorragie system | SPEC-08 | Countdown 3→2→1→0(dead). Tick at start of bleeding knight's turn. Nod de Soin saves (Hémorragie removed, non-permanent exception). Between-node carry-over. Ignorer l'Agonie prevents (no injury roll occurs) |
| 4.4 | `HeroismSystem` | SPEC-09 | 0–6 cap. Earn: Exploit +1, T3+ kill +1, Stabilize +1, clean combat +1 all. Spend: Ignorer Agonie (1), Ignorer Désespoir (1), Relancer (1), Dégâts Maximum (1), Mode Héroïque (6, Le Fou at 4). Mode Héroïque scaffold (persist until 0 PS/PEs, auto-ally-assist, 0 PE, excess → damage dice) |
| 4.5 | `FoldStateSystem` | SPEC-10 | PA=0 → Guardian Suit (PA=5, CdF=5). Systems offline (OD, modules, abilities, PE spending, styles forced Standard). Exit when PA>5 via Nod/Mechanic. Shrine external CdF still applies on top of 5 |
| 4.6 | Despair resolution (full) | SPEC-07 | 1D6 duration, forced Agressif, random ally targeting, stabilization (Parole Base + Aura Combo, difficulty 3), Ignorer Désespoir (1 Héroïsme), permanent at duration=0, edge cases (Despair+Fold: Standard overrides Agressif) |
| 4.7 | `CombatantState` enforcement | SPEC-30 | Single `IsSquadDefeated()`. Every system queries CombatantState (not ad-hoc flags) |

---

## Milestone 5 — Phase 3b: Armor Abilities (Week 8, parallel with M4)

**Developers:** 1 (Armor Abilities Agent)

### Tasks

| # | Task | SPEC | Key Details |
|---|------|------|-------------|
| 5.1 | Warrior Types | SPEC-17.1 | 5 Types (Soldier/Hunter/Scholar/Herald/Scout), +1 OD to aspect's 3 chars, 1 PE/turn maintenance, activation = 1 Movement Action, max 1 switch/turn. OD cap push to 6 (only Type). Demo: 3 per Warrior (Soldier + Hunter + 1 random). Fold deactivates |
| 5.2 | Paladin Shrine | SPEC-17.2 | Free Action to place (2 PE), +6 CdF at rank ±1, anchored to rank, 1 PE/turn maintenance. Disable on: PE=0, Fold, Agony, Despair, Death. Re-place = Free + 2 PE. Scaffolded entry restriction |
| 5.3 | Paladin Watchtower | SPEC-17.2 | 1 Movement Action (2 PE), immobile, Reaction halved (floor(base/2), then rank/cover/style/barrage mods), +1 ranged Combat Action per turn (starting next turn). Deactivate = Free. Incompatible with Ambidextre |
| 5.4 | Priest NanoC | SPEC-17.3 | 4 predefined constructions: Cover Wall (Simple, 3PE, Movement, `hasCover`=true), Barricade (Simple, 3PE, blocks forced displacement), Repair Platform (Detailed, 6PE, +1D6 Mechanic), Nano-Trap (Mechanical, 9PE, −1 action single-use). 10 turns duration. Fold/death removes all |
| 5.5 | Priest Mechanic | SPEC-17.3 | Contact: 3D6+6 PA (4 PE, 1 Movement). Long: 2D6+6 PA (4 PE, 1 Movement). Cannot target Agony. CAN target Fold (secondary exit path) |
| 5.6 | Rogue Ghost | SPEC-17.4 | Free Action activation (2 PE/turn), stealth roll (Discrétion Base + 3 auto-successes), per-enemy detection (`floor(Machine/2)` D6 + Exceptionnel). Machine Majeur auto-detect. Alpha-strike (+Discrétion dice & damage), immediate deactivation after first attack. Ambidextre: first-only bonus. Weapon restrictions (Silencieux for ranged, no Lumière for contact) |

---

## Milestone 6 — Phase 6: Entities (Week 9)

**Developers:** 2 (Enemy AI Agent + Mission & Hub Agent start)

### Enemy AI Agent Tasks

| # | Task | SPEC | Key Details |
|---|------|------|-------------|
| 6.1 | `EnemyAIController` — 4-phase turn execution | SPEC-18.11 | (1) Target selection (priority sort + tiebreakers), (2) Movement (ADVANCE/HOLD/RETREAT/FLANK), (3) Attack (standard Combo Roll per 18.9, Actions Multiples loop), (4) Capacities (Charge Brutale, Régénération) |
| 6.2 | Hybrid AI pattern | SPEC-18.12 | Melee/ranged preference weighting at spawn. Per-turn situational overrides (close distance if HOLD can't reach, fire ranged if ADVANCE can't close) |
| 6.3 | Enemy attack procedure | SPEC-18.9 | Full Aspect dice (no combo), Exceptionnel auto-successes, Exploit (re-roll, excess → bonus D6), Failure Critique (isExposed flag → +2 to first knight attack) |
| 6.4 | Charge Brutale | SPEC-18.13 | Once per combat/phase. Gap ≤ 4 range. Teleport to adjacent rank. Pre-baked damage + 2× Bête. AI trigger: high-priority target at gap > 1 |
| 6.5 | Peur mechanic | SPEC-18.10 | Encounter start, per Peur source. Combo Roll (highest Sang-Froid/Hargne + Combo) vs `floor(highestAspect/2)`. Success: −1 die all tests. Failure: −X dice, −X Def/React. Échec Critique: paralyzed XD6 turns |
| 6.6 | `BandeController` | SPEC-11 | Cohesion pool, Violence-only damage from knights, Débordement (`turn × score`) to all opposing faction, initiative 1, One Bande Rule, Nocte capacities (Ignore CdF, Hypersensibilité Lumineuse) |

### Mission & Hub Agent Tasks (starting)

| # | Task | SPEC | Key Details |
|---|------|------|-------------|
| 6.7 | `MotivationDetector` | SPEC-12 | Event bus listener, tag matching across all knights' motivations, +1D6 PEs (minor) or +25 PEs current+max (major). Voeu triggers from Blason system |

---

## Milestone 7 — Phase 7: Mission & Save (Weeks 10–11)

**Developers:** 1–2 (Mission & Hub Agent + Enemy AI Agent finishing boss)

### Tasks

| # | Task | SPEC | Key Details |
|---|------|------|-------------|
| 7.1 | `BossPhaseController` | SPEC-14 | P1→P2 on PS=0 (override Agony). Apply P2 stats from SPEC-21.5 (PS reset 120, Bouclier 12, Défense 10, Actions Multiples 2). Unlock Ombre dévorante + Régénération. AI shift. −1D6 PEs all knights. Bande reinforcement (AddCohesion 200 or fresh spawn). Charge cooldown reset |
| 7.2 | `SaveSystem` | SPEC-13 | Serialize `RunState` at node transitions (JSON). No mid-combat save. Resume = reload pre-combat state + rebuild RNG. Auto-save on: campaign creation, node transitions, Camelot return |
| 7.3 | `MissionManager` — 5-node graph | SPEC-22 | Node 1 (Nocte 200), Node 2A/2B (branch choice), Node 3 (Faune+2 Bestians), Node 4 (narrative), Node 5 (boss). Combat end = all enemies + bandes eliminated. No retreat from Node 5 |
| 7.4 | Narrative event system | SPEC-22.3 | 4 events: survivor (auto, +1D6 PEs all), Point Faible (1D6 roll, triggers on 6, reveal Sang-Froid), pre-battle prayer (choice, spend 1 Héroïsme → +1D6 PEs all), darkness erupts (phase transition, −1D6 PEs all) |
| 7.5 | Node transition logic | SPEC-13 | No auto-recovery. State carries over. Nod use allowed (free). Dead stay dead. `ApplyCombatResults` atomic commit |
| 7.6 | Victory/defeat/retreat | SPEC-22.4 | Boss P2 PS=0 = victory. All 4 dead/incapacitated = defeat. Retreat from Node 1–4 = survive with current state |

---

## Milestone 8 — Phase 8: Camelot Hub (Week 12)

**Developers:** 1 (Mission & Hub Agent)

### Tasks

| # | Task | SPEC | Key Details |
|---|------|------|-------------|
| 8.1 | Hub state machine | SPEC-25 | View Roster, auto-refill (grenades 5, Nods 3/3/3, Chargeur), PS/PA/PE full restore (free), PEs unchanged |
| 8.2 | Infirmary | SPEC-25 | Cybernetic Implant (20 PG, remove 1 injury, maxPEs−3, floor 10). Reconstruction Therapy (100 PG, remove all, reset implants, restore maxPEs). Prisonnier UI warning |
| 8.3 | PG economy | SPEC-25.3 | Full earning table (mission complete 30, all alive +10, boss +15, colosse +5, etc.). Spending table. Running total persists |
| 8.4 | Squad selection | SPEC-03/25 | 4-of-8 selection, lock, transition to deployment. Composition guidance (1 melee, 1 tank, 1 support, 1 flex) |
| 8.5 | Campaign end conditions | SPEC-25.4 | All 8 dead = game over. Boss cleared = victory. Memorial screen |

---

## Milestone 9 — UI Layer (Weeks 10–14, parallel)

**Developers:** 1–2 (UI Agent, runs in parallel with M7–M8)

### Tasks

| # | Task | SPEC | Priority |
|---|------|------|----------|
| 9.1 | **Combat HUD** — Initiative timeline (horizontal, portraits), resource bars per combatant (PS/PA/CdF/PE/PEs), state badges | SPEC-26/31 | P0 |
| 9.2 | **Action menu** — Attack, Move, Nod, Style Switch, Ability, Suppress, End Turn. Disabled actions show reason string (SPEC-31-AC3) | SPEC-31 | P0 |
| 9.3 | **Target selection** — Highlight valid targets per weapon range, gap indication, cover indicator, "out of range" feedback | SPEC-31 | P0 |
| 9.4 | **Roll breakdown panel** — Dice pool, OD autos, evens, total successes, threshold, hit/miss. Damage: dice, sum, flat, per-layer absorption | SPEC-31 | P0 |
| 9.5 | **Combat log** — Scrollable, last ~50 events, formatted entries (SPEC-31-AC2) | SPEC-31 | P0 |
| 9.6 | **Countdown timers** — Hémorragie (red pulsing, always visible), Despair (turns to permanent), DoT (turns + damage), Choc/Parasitage | SPEC-31 | P0 |
| 9.7 | **Rank display** — 2D side-view: 4 knight sprites left, 4 enemy sprites right, cover icons, selection highlights | SPEC-26 | P0 |
| 9.8 | **Dispersion blast zone preview** — Before confirming, highlight all affected positions, require confirmation if allies in zone | SPEC-31/20.5 | P1 |
| 9.9 | **Deployment screen** — 4 rank slots, drag-drop, enemy preview, cover layout, confirm button | SPEC-26 | P1 |
| 9.10 | **Hub screens** — Roster grid, Infirmary panel, Mission select, Memorial | SPEC-26 | P1 |
| 9.11 | **Node map** — 5-node path, branch at Node 2, party status summary, "no auto-heal" warning | SPEC-26 | P2 |
| 9.12 | **Enemy intent preview** — Target + action type icon on enemy portraits | SPEC-31 | P2 |

### SPEC-31 Mandatory Acceptance Criteria
- **AC1:** Player can always answer "Why did I miss?" and "Where did the damage go?" without guessing
- **AC2:** Every combat action produces a log entry with roll result + damage breakdown
- **AC3:** Any disabled action displays a reason string (not just greyed out)

---

## Milestone 10 — 2D Art Direction & Rendering (Weeks 6–14, parallel)

**Developers:** 1 artist (parallel track), or developer-produced placeholders

### Art Style

> **Darkest Dungeon-inspired dark gothic palette** — heavy blacks, muted reds/blues/golds, hand-drawn linework with rough edges. French medieval-futuristic fusion (armored knights in sci-fi exoskeletons fighting cosmic horror).

### Sprite Specifications

| Asset | Dimensions | States | Notes |
|-------|------------|--------|-------|
| Knight (per class) | 128–256px tall | Idle (4f), Attack (6f), Hit (3f), Death (4f), Fold (2f), Ghost (transparency) | Side-view, facing right. 4 classes × 5 states = 20 sprite sets |
| Enemy (per type) | Variable (tier-scaled) | Idle (4f), Attack (6f), Hit (3f), Death (4f) | Nocte = small swarm, Bestian = medium, Faune = tall humanoid, Behemot = massive, Ours = boss-sized |
| UI Icons | 32–64px | Static | Weapon icons, Nod icons, status badges (Fold/Agony/Despair/Ghost/Cover), Tarot cards |
| Portraits | 64×64 or 128×128 | Static | Per-knight (8), per-enemy-type (5) |

### Rendering Setup

| Component | Recommendation |
|-----------|---------------|
| Sprite rendering | `SpriteRenderer` (not UI Image) for combat entities |
| Sorting | Sort by rank (Rank 4 behind Rank 1, painter's algorithm) |
| Animation | Unity 2D Animation package (bone-based) or frame-by-frame spritesheets. For demo budget: frame-by-frame is faster |
| VFX | Particle systems: damage numbers (floating TMP), CdF flash (blue), PA absorb (orange), PS hit (red), PEs drain (purple wisps), Fold activation (blue energy collapse), Charge Brutale (dust + shake) |
| Camera | Fixed orthographic. Entire 8-rank battlefield fits. Optional: slight shake on Charge/phase transition |

### Placeholder Strategy (Weeks 1–6)
- Colored rectangles with TMP labels (class name, tier)
- State indicated by color (green = normal, yellow = fold, red = agony, purple = despair)
- Replace with commissioned art starting Week 6

---

## Milestone 11 — Integration Tests & Polish (Weeks 13–15)

**Developers:** 1–2

### Integration Test Scenarios (from 15_IntegrationTests.md)

| Scenario | Systems Exercised | Fixed Seed? |
|----------|-------------------|-------------|
| **1: Fold → Despair → Hémorragie Cascade** | SPEC-05, 06, 08, 10, 33, 18.13, 30 | Yes |
| **2: Ghost Alpha-Strike → Exploit → Dual-Wield** | SPEC-17.4, 04 (Exploit, Ambidextre, Jumelé), 05 (Orfèvrerie, Silencieux), 09, 07 | Yes |
| **3: Barrage + Boss Phase Transition** | SPEC-20 (Barrage), 17.2 (Watchtower), 14 (phase), 11 (AddCohesion), 18.13 (Charge reset), 21.5, 18.8 | Yes |
| **4: Grenade Friendly Fire + Despair + Stabilization** | SPEC-19.4, 20.5 (Dispersion), 20.1 (Meurtrier, Ultraviolence), 06 (Anathème), 07 (Despair), 30, 20.3 (Choc) | Yes |
| **5: Full Node Transition Attrition** | SPEC-13, 33 §33.4, 33 §33.7, 32 (field boundary), 08 (Hémorragie carry-over) | Yes |

### Quality Checks

| Check | Method | SPEC |
|-------|--------|------|
| No `UnityEngine.Random` in gameplay | `grep -r "UnityEngine.Random" Scripts/` — must return 0 | SPEC-28-AC3, 32-AC3 |
| Cross-run determinism | Same RunSeed, 2 full runs → byte-identical combat log | SPEC-28-AC1 |
| `IsSquadDefeated()` single implementation | One usage site (CombatManager) | SPEC-30-AC2 |
| Combat runs headless | All 5 integration scenarios pass in EditMode (no UI) | SPEC-32-AC1 |
| No direct state mutation from UI | Code review / interface enforcement | SPEC-32-AC2 |
| State machine transitions only | No `SceneManager.LoadScene` in combat code | SPEC-32-AC4 |
| Destroy CombatSession = no NRE in Hub | Null-safety test | SPEC-32-AC5 |

---

## Milestone 12 — Full Playthrough & Ship (Weeks 16–18)

**Developers:** All

### Tasks

| # | Task | Details |
|---|------|---------|
| 12.1 | Full demo playthrough | Camelot → Squad Select → Deploy → 5 nodes → Boss (2 phases) → Victory → Camelot. And: defeat path, retreat path, partial loss path |
| 12.2 | Balance tuning | Nocte Cohesion values, enemy damage, PG earning rates, Nod economy pressure, boss Régénération balance |
| 12.3 | Edge case sweep | Despair + Fold, Hémorragie between nodes, all 4 dead = TPK, Mode Héroïque, all Tarot disadvantage triggers, Prisonnier PEs block |
| 12.4 | Build & package | Target platforms: WebGL (easy sharing) + Windows standalone. Main menu: New Campaign / Continue |
| 12.5 | README / Player guide | Brief French-language guide explaining combat mechanics for playtesters |

---

## Critical Path & Risk Assessment

```
M0 (RNG/Infra) → M1 (Data SOs) → M2 (Position) → M3 (Core Combat ★) → M4 (Consequences)
                                                                         ↑ parallel
                                                    M5 (Abilities) ──────┘
                                                                    → M6 (Entities/AI) → M7 (Mission) → M8 (Hub)
                                                                                                               ↓
M9 (UI) runs parallel from Week 10 ────────────────────────────────────────────────────────────────── → M11 (Tests) → M12 (Ship)
M10 (Art) runs parallel from Week 6 ──────────────────────────────────────────────────────────────────────┘
```

### Risk Matrix

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| **ArmourLayerResolver bugs** (8 steps, 10+ tags) | 🔴 Critical — cascades everywhere | Medium | Exhaustive unit tests for EVERY worked example in SPEC-06. This is the #1 system to get right |
| **Combo Roll + Style combinatorial explosion** | 🔴 Critical — player-facing, visible | Medium | `ComboRollBuilder` pattern with step-by-step logging. Test each style independently. Integration Scenario 2 catches Ghost + Exploit + Dual-wield |
| **Boss Phase 2 multi-system interaction** | 🟡 High — Régénération + Bande + AI shift | Medium | Integration Scenario 3 specifically covers this. Implement early in M7 |
| **Deterministic RNG leaks** | 🟡 High — breaks reproducibility | Low | `IRng` interface enforced via DI. Static grep as CI gate. Zero-tolerance for `UnityEngine.Random` |
| **UI scope creep** | 🟡 High — 6+ screens, roll breakdown, log | High | Start UI last (Week 10). Headless tests prove logic before UI exists. Minimal viable HUD first |
| **2D art production bottleneck** | 🟢 Medium — placeholders work | Medium | Colored rectangles + labels for demo. Commission art in parallel starting Week 6 |
| **Tarot disadvantage edge cases** (22 unique disadvantages) | 🟢 Medium | Low | Implement stat-affecting ones fully; "once per mission" narrative disadvantages can be stubs with TODO markers |

---

## Stub vs Full Implementation Matrix

| System | Implementation Level | Justification |
|--------|---------------------|---------------|
| **ArmourLayerResolver** | ❌ **FULL** | Central damage pipeline — no shortcuts |
| **ComboRollResolver** | ❌ **FULL** | Player-facing, every edge case matters |
| **Deterministic RNG** | ❌ **FULL** | Architectural foundation |
| **Enemy AI decision tree** | ❌ **FULL** | Core gameplay — believable enemies |
| **All 4 armor class abilities** | ❌ **FULL** | Each class must feel distinct |
| **Nod system** | ❌ **FULL** | Primary recovery mechanic |
| **Espoir/Despair** | ❌ **FULL** | Key narrative/mechanical tension |
| **Boss phase controller** | ❌ **FULL** | Demo climax |
| **YES-scoped weapon effects** (~20) | ❌ **FULL** | Used by demo weapons/enemies |
| SCAFFOLDED weapon effects (~10) | ✅ **STUB** (interface + no-op) | No demo weapon uses them |
| Modules (equippedModules) | ✅ **STUB** | Out of demo scope per spec |
| Armor Evolutions (150/200/250 PG) | ✅ **STUB** | Scaffolded. Lock UI buttons |
| Mode Héroïque objectives | ✅ **STUB** | Use simplified "persist until 0 PS/PEs" |
| Points de Contact | ✅ **STUB** | Spec says do NOT compute for demo |
| Camelot weapon shop | ✅ **STUB** | Default loadouts only |
| Difficulty scaling | ✅ **STUB** | Wire dropdown, keep fixed |
| Audio/VFX | ✅ **STUB** | Fire SPEC-27 events, attach SFX later |
| Mid-mission save | ✅ **STUB** | Only node-boundary saves needed |
| 22 Tarot advantages (stat-affecting) | ⚠️ **PARTIAL** | Full for stat-affecting; stub for narrative "once per mission" triggers |
| 22 Tarot disadvantages | ⚠️ **PARTIAL** | Full for combat-affecting (Solitaire, Vétéran, Prisonnier, Trop Prudent); stub for narrative |

---

## Developer Role Assignment

The spec defines 9 agent roles. With 2–4 developers, assign as follows:

### 2-Developer Team

| Developer | Agent Roles | Milestones |
|-----------|-------------|------------|
| **Dev A** (Systems) | Architecture + Combat Core + Enemy AI | M0, M2, M3 (3.1–3.10), M6 (6.1–6.6), M7 (7.1) |
| **Dev B** (Content + State) | Data + Weapon/Content + Combat State + Armor Abilities + Mission/Hub + UI | M1, M3 (3.11–3.12), M4, M5, M6 (6.7), M7 (7.2–7.6), M8, M9 |

### 4-Developer Team

| Developer | Agent Roles | Milestones |
|-----------|-------------|------------|
| **Dev A** (Architecture) | Architecture + Combat Core | M0, M2, M3 (3.1–3.10) |
| **Dev B** (Data) | Data + Weapon/Content | M1 (all) |
| **Dev C** (Combat State) | Combat State + Armor Abilities + Enemy AI | M3 (3.11–3.12), M4, M5, M6 (6.1–6.7) |
| **Dev D** (Meta + UI) | Mission/Hub + UI | M7, M8, M9 |

---

## Time Estimate Summary

| Milestone | Weeks | Developers | Dependencies |
|-----------|-------|------------|-------------|
| **M0: Bootstrap** | 1 | 1–2 | None |
| **M1: Data Layer** | 2 | 2 (parallel) | M0 |
| **M2: Position & Turns** | 1 | 1 | M1 |
| **M3: Core Combat ★** | 3 | 2 | M2 |
| **M4: Consequences** | 1 | 1 | M3 |
| **M5: Armor Abilities** | 1 | 1 (parallel with M4) | M3 |
| **M6: Entities** | 1 | 2 (parallel) | M4, M5 |
| **M7: Mission & Save** | 2 | 1–2 | M6 |
| **M8: Camelot Hub** | 1 | 1 | M7 |
| **M9: UI Layer** | 4 | 1–2 (parallel M7–M8) | M3 (for combat UI), M8 (for hub UI) |
| **M10: Art Direction** | Ongoing | 1 artist (parallel from W6) | None |
| **M11: Integration Tests** | 2 | 1–2 | M8 |
| **M12: Ship** | 2 | All | M11 |
| **TOTAL** | **~16 weeks** | **2–4 devs** | |

### Gantt Overview (simplified)

```
Week:  1    2    3    4    5    6    7    8    9   10   11   12   13   14   15   16   17   18
       ├─M0─┤
            ├────M1────┤
                       ├─M2─┤
                            ├──────M3 (★ Vertical Slice)──────┤
                                                              ├─M4─┤
                                                              ├─M5─┤ (parallel)
                                                                   ├─M6─┤
                                                                        ├────M7────┤
                                                                                   ├─M8─┤
                                                   ├────────────M9 (UI, parallel)───────────┤
                                 ├────────────────M10 (Art, parallel)───────────────────────┤
                                                                                        ├──M11──┤
                                                                                                ├──M12──┤
```

---

## Appendix A: Key Architecture Decisions

### A.1 Single Scene (SPEC-32)
- One Unity scene: `Main.unity`
- `GameStateMachine` manages all state transitions
- No `SceneManager.LoadScene()` in combat code
- UI panels activated/deactivated by state machine

### A.2 State Isolation (SPEC-32)
- **RunState**: Persistent (survives combat exit). Owns: knight roster, PG, injuries, mission progress
- **CombatSession**: Ephemeral (created at deployment, destroyed on exit). Owns: turn queue, CombatantState instances, active effects, RNG streams
- **ApplyCombatResults**: Atomic commit from CombatSession → RunState on combat exit. Crash = no corruption
- **Hémorragie exception**: Only combat state that crosses to RunState

### A.3 CombatManager as Authority (SPEC-32)
- Single `CombatManager` owns: turn queue, legality checks, resolution, effect application, victory/defeat
- UI sends commands (`RequestAttack`, `RequestMove`, `RequestNod`) → CombatManager validates → executes → fires events
- UI subscribes to events for display updates — never reads internal combat collections directly

### A.4 French/English Naming (SPEC-32)
- **Enums:** French (`AGRESSIF`, `DEFENSIF`, `SOIN`)
- **Fields/Variables:** English (`isInAgony`, `currentPS`)
- **Display/UI strings:** French (player-facing)
- **Classes:** English (`KnightBase`, `BandeController`)

---

## Appendix B: Quick Reference — Global Rules

From SPEC-00/INDEX:

| Rule | Details |
|------|---------|
| **Math rounding** | All divisions floor (round down) |
| **Operation order** | Division/Multiplication always before Addition/Subtraction |
| **Successive doubling** | ×3 (not ×4) |
| **Hit threshold** | Successes must **strictly exceed** Defense (melee) or Reaction (ranged). Equal = miss |
| **Defense/Reaction floor** | Minimum 0 after all modifiers. 1 success > 0 = hit |
| **Duration "1 turn"** | From activation until start of activating combatant's next turn |
| **PG** | Meta-currency earned through play, spent at Camelot Hub |

---

*End of implementation plan. This document should be reviewed alongside the full specification suite (`/TechSpec/specifications/`) for detailed acceptance criteria per system.*

