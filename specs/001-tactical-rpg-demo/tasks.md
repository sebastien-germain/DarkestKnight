# Tasks: Knight Into the Darkness — Tactical RPG Demo

**Input**: Design documents from `/specs/001-tactical-rpg-demo/`
**Prerequisites**: plan.md, spec.md, data-model.md, contracts/, research.md, quickstart.md
**Constitution**: Testing is MANDATORY (Principle VI). One commit per task (Principle IX). SPEC-16 dependency order enforced (Principle VII).

**Organization**: Tasks follow SPEC-16 8-phase dependency order. Each task is tagged with the user story it serves. Tasks within a phase respect internal dependencies (data → logic → tests).

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies on incomplete tasks)
- **[Story]**: Which user story (US1–US7). Setup/foundational tasks have no story label.

---

## Phase 1: Setup — Project Bootstrap (M0)

**Purpose**: Foundational infrastructure that ALL systems depend on. No user story work can begin until this phase is complete.

**⚠️ CRITICAL**: Corresponds to SPEC-16 Phase 0. All downstream phases depend on this.

- [ ] T001 Create Unity 6.3 LTS project with URP 2D, install packages (Unity.Mathematics, TextMeshPro, Input System, Test Framework) per research.md in Assets/
- [ ] T002 Create assembly definitions: DarkestKnight.Core.asmdef (Scripts/Core + Scripts/Data), DarkestKnight.Runtime.asmdef (Scripts/Combat + Scripts/Meta + Scripts/UI), DarkestKnight.Tests.asmdef (Tests/) per research.md
- [ ] T003 [P] Define ALL enums and constants from SPEC-14 in Scripts/Core/Constants/Enums.cs — reference /TechSpec/specifications/14_Constants.md
- [ ] T004 [P] Implement GameStateMachine with states MainMenu→CamelotHub→SquadSelect→Deployment→Combat→NodeTransition→MissionEnd in Scripts/Core/GameStateMachine.cs — reference SPEC-32
- [ ] T005 [P] Implement EventBus typed event aggregator with all events from contracts/eventbus-events.md in Scripts/Core/EventBus.cs — reference SPEC-27
- [ ] T006 [P] Implement deterministic RNG: IRng interface in Scripts/Core/RNG/IRng.cs, Xoshiro128StarStar in Scripts/Core/RNG/Xoshiro128StarStar.cs, StableStringHash (FNV-1a) in Scripts/Core/RNG/StableStringHash.cs, RngFactory with NodeSeed derivation in Scripts/Core/RNG/RngFactory.cs — reference SPEC-28
- [ ] T007 [P] Scaffold RunState (persistent) and CombatSession (ephemeral) with field boundary from SPEC-32 in Scripts/Core/RunState.cs and Scripts/Combat/CombatSession.cs
- [ ] T008 Write EditMode tests for RNG determinism (same seed → identical sequence, cross-stream isolation, NodeSeed derivation) in Tests/EditMode/RNGDeterminismTests.cs — reference SPEC-28-AC1, AC5
- [ ] T009 Create Main.unity scene with GameStateMachine root object in Assets/Scenes/Main.unity — reference SPEC-32

**Checkpoint**: Project compiles. RNG determinism proven by green tests. All enums available.

---

## Phase 2: Foundational Data Layer (M1 — SPEC-16 Phase 1)

**Purpose**: All ScriptableObject schemas defined and demo instances authored. Serves ALL user stories.

**⚠️ CRITICAL**: No combat or gameplay work can begin until this phase is complete.

### SO Schema Definitions

- [ ] T010 [P] Implement KnightBaseSO schema with all fields from data-model.md §2.1 and derived value computation methods (Defense, Reaction, Initiative, maxPS, maxPEs) in Scripts/Data/KnightBaseSO.cs — reference SPEC-01
- [ ] T011 [P] Implement ArchetypeDataSO schema (archetypeId, bonus1, bonus2, choicePool, choiceBonusSlot) in Scripts/Data/ArchetypeDataSO.cs — reference SPEC-02.9
- [ ] T012 [P] Implement HautFaitDataSO schema (bonusAspect, bonusAspectAlt for dual-aspect, conditionType, surnoms) in Scripts/Data/HautFaitDataSO.cs — reference SPEC-02.8
- [ ] T013 [P] Implement BlasonDataSO schema (voeuEventTag, uniqueness rule) in Scripts/Data/BlasonDataSO.cs — reference SPEC-02.5
- [ ] T014 [P] Implement TarotCardSO schema (linkedAspect, charPoints, advantage, disadvantage) in Scripts/Data/TarotCardSO.cs — reference SPEC-24
- [ ] T015 [P] Implement WeaponProfileSO schema with DamageProfile struct (damageDice, violenceDice, range, effects, forceMode, profileSwitchCost) in Scripts/Data/WeaponProfileSO.cs — reference SPEC-19
- [ ] T016 [P] Implement EnemyBaseSO schema (tier system, AspectExceptionnel, Colosse 10:1 rule, boss phases) in Scripts/Data/EnemyBaseSO.cs — reference SPEC-18
- [ ] T017 [P] Implement EncounterSO schema (CoverLayout, EnemySpawnEntry, ReinforcementQueue, BandeConfig, EncounterTrigger) in Scripts/Data/EncounterSO.cs — reference SPEC-29
- [ ] T018 [P] Implement InjuryTableSO schema (4×6 severity matrix with InjuryEntry: injuryId, effects, isPermanent) in Scripts/Data/InjuryTableSO.cs — reference SPEC-23
- [ ] T019 [P] Implement MotivationTemplateSO schema (eventTag, type) and MissionDefinitionSO schema (7-node graph) in Scripts/Data/MotivationTemplateSO.cs and Scripts/Data/MissionDefinitionSO.cs — reference SPEC-12.1, SPEC-22

### SO Instance Authoring

- [ ] T020 [P] Author 17 ArchetypeDataSO instances in Assets/Data/Knights/ — reference SPEC-02.9 archetype table
- [ ] T021 [P] Author 14 HautFaitDataSO instances with dual-aspect resolution weights in Assets/Data/Knights/ — reference SPEC-02.8
- [ ] T022 [P] Author 9 BlasonDataSO instances (Le Dragon deferred) with voeu event tags in Assets/Data/Knights/ — reference SPEC-02.5
- [ ] T023 [P] Author 22 TarotCardSO instances (Major Arcana) with stat bonuses, advantages, disadvantages in Assets/Data/Tarot/ — reference SPEC-24, /TechSpec/specifications/09_TarotSystem.md
- [ ] T024 [P] Author 8 demo WeaponProfileSO instances (Pistolet de Service, Marteau-Épieu, Couteau de Combat, Fusil de Précision, Fusil d'Assaut, Lance-Grenade, MUSIC, Noyau Nod) + 5 grenade types in Assets/Data/Weapons/ — reference SPEC-19 §19.3, /TechSpec/specifications/07_WeaponsAndEffects.md
- [ ] T025 [P] Author 5 EnemyBaseSO instances (Nocte bande, Bestian T2, Faune T3, Behemot T4, Ours Corrompu T5 with P1+P2 phases) in Assets/Data/Enemies/ — reference SPEC-21, /TechSpec/specifications/08_ContentData.md
- [ ] T026 [P] Author InjuryTableSO instance (4×6 matrix) in Assets/Data/Injuries/ — reference SPEC-23, /TechSpec/specifications/08_ContentData.md
- [ ] T027 [P] Author 10 minor + 5 major MotivationTemplateSO instances in Assets/Data/Motivations/ — reference SPEC-12.1
- [ ] T028 [P] Author 5 EncounterSO instances (nodes 1–5 with enemy placements, cover, bande config, boss phase trigger) and 1 MissionDefinitionSO in Assets/Data/Encounters/ and Assets/Data/Missions/ — reference SPEC-29, SPEC-22

### Data Layer Tests

- [ ] T029 Write EditMode tests for KnightBaseSO derived values (Defense, Reaction, Initiative, maxPS, maxPEs with all ACs from SPEC-01) in Tests/EditMode/KnightDataModelTests.cs

**Checkpoint**: All 60+ SO instances authored. Project compiles. Derived value tests green.

---

## Phase 3: US1 — Turn-Based Combat Encounter (M2+M3+M4 — SPEC-16 Phases 2–5)

**Goal**: Complete core combat loop — 4 knights vs enemies on linear track with full damage pipeline, status effects, and win/loss conditions.

**Independent Test**: Place 4 knights against enemies, resolve combat to completion. Delivers core tactical experience.

### Position & Turn System (M2 — SPEC-16 Phase 2)

- [ ] T030 [US1] Implement PositionManager with 4-rank linear track, gap formula, range band validation, rank modifiers (R1: +1 Def, R4: -1 Def/-1 React) in Scripts/Combat/PositionManager.cs — reference SPEC-03
- [ ] T031 [US1] Implement cover system (hasCover per rank, +3 Reaction vs ranged, -3 dice from cover, cover removal procedure) in Scripts/Combat/PositionManager.cs — reference SPEC-03
- [ ] T032 [US1] Implement movement system (swap adjacent ally, move to empty rank, 2-rank move costs combat action, forced displacement chain-swap) in Scripts/Combat/PositionManager.cs — reference SPEC-03
- [ ] T033 [US1] Implement TurnQueue with FFX-style initiative (3D6 + Initiative, ties: higher first → random, delay mechanic, Bande always last) in Scripts/Combat/TurnQueue.cs — reference SPEC-15
- [ ] T034 [US1] Implement deployment phase logic (player assigns 4 knights to Ranks 1–4, enemy placement from EncounterSO) in Scripts/Combat/PositionManager.cs — reference SPEC-03

### Core Combat Resolution (M3 — SPEC-16 Phases 3–4)

- [ ] T035 [US1] Implement CombatantState with LifeState, ControlState, ActionState, ArmorState, IsSquadDefeated() canonical function in Scripts/Combat/State/CombatantState.cs — reference SPEC-30
- [ ] T036 [US1] Implement ComboRollResolver: dice pool building (Base + Combo + style mods + PEs penalty + injury penalties), D6 roll, even counting, OD auto-successes, Exploit (all even → reroll + heroism), Failure Critique (all odd) in Scripts/Combat/Resolvers/ComboRollResolver.cs — reference SPEC-04
- [ ] T037 [US1] Implement combat style modifier system (9 styles: Standard, Agressif, Défensif, Mise à Couvert, Puissant, Pilonnage, Précis, Ambidextre, Akimbo) in Scripts/Combat/Resolvers/ComboRollResolver.cs — reference SPEC-04
- [ ] T038 [US1] Implement DamageResolver: 8-step damage chain (hit check → weapon dice SUM → flat bonus → stat bonuses → style bonus dice → Dégâts Maximum → route to armour) and Violence chain (SUM → Cohesion) in Scripts/Combat/Resolvers/DamageResolver.cs — reference SPEC-05
- [ ] T039 [US1] Implement ArmourLayerResolver: 8-step canonical pipeline (CdF → Bouclier → Chair Mineur → Destructeur → PA + passthrough floor(paAbsorbed/5) → Meurtrier → PS → Agony trigger) in Scripts/Combat/Resolvers/ArmourLayerResolver.cs — reference SPEC-06
- [ ] T040 [US1] Implement Anathème damage route (CdF → remainder hits PEs, PA bypassed) in Scripts/Combat/Resolvers/ArmourLayerResolver.cs — reference SPEC-06
- [ ] T041 [US1] Implement EspoirSystem: PEs penalty calculation (max(0, 10 - currentPEs)), state table (Résolu→Désespoir), loss/recovery source tracking in Scripts/Combat/State/EspoirSystem.cs — reference SPEC-07
- [ ] T042 [US1] Implement NodSystem: 3 types (Soin/Armure/Énergie), 3D6 restore, Guérison Rapide +3, Agony targeting (Soin only), Fold PE freeze, between-node free use in Scripts/Combat/State/NodSystem.cs — reference SPEC-33
- [ ] T043 [US1] Implement action economy enforcement: 1 Move + 1 Combat per turn, combat→movement substitution, profile switch costs, weapon switch = 1 Movement, style switch = Free in Scripts/Combat/CombatManager.cs — reference SPEC-04, FR-006b

### Consequences (M4 — SPEC-16 Phase 5)

- [ ] T044 [US1] Implement InjurySystem: Agony trigger (PS=0), injury table roll (D6 column + D6 row), stat reduction, derived value recalculation, Trompe la Mort reroll in Scripts/Combat/State/InjurySystem.cs — reference SPEC-08/23
- [ ] T045 [US1] Implement Hémorragie system: 3-turn countdown (ticks at start of bleeding knight's turn), Nod de Soin saves, between-node carry-over, Ignorer l'Agonie prevents in Scripts/Combat/State/InjurySystem.cs — reference SPEC-08
- [ ] T046 [US1] Implement HeroismSystem: 0–6 cap, earn triggers (Exploit +1, T3+ kill +1, Stabilize +1, clean combat +1 all), spend actions (Ignorer Agonie, Ignorer Désespoir, Relancer, Dégâts Maximum, Mode Héroïque scaffold) in Scripts/Combat/State/HeroismSystem.cs — reference SPEC-09
- [ ] T047 [US1] Implement FoldStateSystem: PA=0 → Guardian Suit (PA=5, CdF=5), systems offline (OD, abilities, PE spending, styles forced Standard), exit when PA>5 via Nod/Mechanic in Scripts/Combat/State/FoldStateSystem.cs — reference SPEC-10
- [ ] T048 [US1] Implement Despair resolution: 1D6 duration, forced Agressif, random ally targeting, stabilization roll (Parole Base + Aura Combo, difficulty 3), Ignorer Désespoir, permanent at duration=0, Despair+Fold override in Scripts/Combat/State/EspoirSystem.cs — reference SPEC-07
- [ ] T049 [US1] Implement CombatManager as sole combat authority: turn loop, legality checks, command validation (from contracts/command-interface.md), victory/defeat evaluation, EventBus event firing in Scripts/Combat/CombatManager.cs — reference SPEC-32

### Combat Tests

- [ ] T050 [US1] Write EditMode tests for ComboRollResolver: all 9 styles, Exploit, Failure Critique, PEs penalty, pool=0 auto-fail, OD auto-successes in Tests/EditMode/ComboRollTests.cs — reference SPEC-04 ACs
- [ ] T051 [US1] Write EditMode tests for ArmourLayerResolver: all worked examples from SPEC-06 (CdF, PA passthrough, Anathème route, Colosse 10:1) in Tests/EditMode/ArmourLayerTests.cs
- [ ] T052 [US1] Write EditMode tests for DamageResolver: damage chain, violence chain, excess successes in Tests/EditMode/DamageResolverTests.cs — reference SPEC-05 ACs
- [ ] T053 [US1] Write EditMode tests for EspoirSystem: PEs penalty scaling, Despair trigger, stabilization in Tests/EditMode/EspoirSystemTests.cs — reference SPEC-07 ACs
- [ ] T054 [US1] Write EditMode tests for InjurySystem: Agony trigger, injury table lookup, Hémorragie countdown, Trompe la Mort in Tests/EditMode/InjurySystemTests.cs — reference SPEC-08/23 ACs
- [ ] T055 [US1] Write EditMode tests for NodSystem: 3 types, Agony exception, Fold PE freeze, quantity tracking in Tests/EditMode/NodSystemTests.cs — reference SPEC-33 ACs

**Checkpoint**: ★ VERTICAL SLICE — Headless combat runs: 4 knights vs enemies, full turn cycle, damage pipeline, Nod usage, status effects, victory/defeat. All combat tests green.

---

## Phase 4: US2 — Knight Roster & Squad Selection (M1b + UI)

**Goal**: 8 procedurally generated knights with unique identities. Roster screen for viewing and selecting 4 for deployment.

**Independent Test**: Generate 8 knights from seed, display stats, select 4 for deployment.

**Dependencies**: Phase 2 (data layer SOs)

- [ ] T056 [US2] Implement GenerationPipeline: full 10-step algorithm (Base stats → Archetype → Tarot weighted draw → distribute points using Weighted Distribution Table → Haut Fait with dual-aspect resolution → Blason with voeu uniqueness → Motivations → Meta-Armor → OD → Equipment → Derived values) in Scripts/Meta/GenerationPipeline.cs — reference SPEC-02
- [ ] T057 [US2] Implement Tarot weighted draw algorithm (weight 3 for primary aspects, incompatibility check XIII+XVI, 5-card selection: 2 advantages + 1 disadvantage, Amnésique post-processing) in Scripts/Meta/GenerationPipeline.cs — reference SPEC-24, /TechSpec/specifications/09_TarotSystem.md
- [ ] T058 [US2] Implement Armor Class Weighted Distribution Table for Tarot/HautFait characteristic point distribution (per-class weights, cap enforcement, capped redistribution) in Scripts/Meta/GenerationPipeline.cs — reference SPEC-02 §Weighted Distribution Table
- [ ] T059 [US2] Write EditMode tests for GenerationPipeline: 8 distinct knights from seed, correct class distribution (2 each), no characteristic exceeds parent Aspect, deterministic generation in Tests/EditMode/KnightGenerationTests.cs — reference SPEC-02 ACs
- [ ] T060 [US2] Implement roster integration in RunState: store 8 knights, track dead/alive, squad selection (4-of-8), transition to deployment in Scripts/Core/RunState.cs — reference SPEC-03/25

**Checkpoint**: Generate 8 unique knights from seed. Same seed = identical results. Tests green.

---

## Phase 5: US3 — Armor Class Abilities (M5 — SPEC-16 Phase 3b)

**Goal**: Each armor class has mechanically distinct abilities. Can be tested in combat.

**Independent Test**: Activate each class ability in combat, verify mechanical effect.

**Dependencies**: Phase 3 (core combat)

- [ ] T061 [P] [US3] Implement WarriorTypeSystem: 5 Types (Soldier/Hunter/Scholar/Herald/Scout), +1 OD to aspect's 3 chars (can push to 6), 1 PE/turn maintenance, activation = 1 Movement Action, Fold deactivates in Scripts/Combat/Abilities/WarriorTypeSystem.cs — reference SPEC-17.1
- [ ] T062 [P] [US3] Implement PaladinShrineSystem: Free Action deploy (2 PE), +6 CdF at rank ±1, anchored to rank, 1 PE/turn maintenance, disable conditions (PE=0, Fold, Agony, Despair, Death) in Scripts/Combat/Abilities/PaladinShrineSystem.cs — reference SPEC-17.2
- [ ] T063 [P] [US3] Implement PaladinWatchtowerSystem: 1 Movement Action (2 PE), immobile, Reaction halved, +1 ranged Combat Action per turn, incompatible with Ambidextre in Scripts/Combat/Abilities/PaladinWatchtowerSystem.cs — reference SPEC-17.2
- [ ] T064 [P] [US3] Implement PriestNanoCSystem: 4 constructions (Cover Wall 3PE, Barricade 3PE, Repair Platform 6PE, Nano-Trap 9PE), 10 turns duration, Fold/death removes in Scripts/Combat/Abilities/PriestNanoCSystem.cs — reference SPEC-17.3
- [ ] T065 [P] [US3] Implement PriestMechanicSystem: Contact 3D6+6 PA (4 PE), Long 2D6+6 PA (4 PE), cannot target Agony, CAN target Fold in Scripts/Combat/Abilities/PriestMechanicSystem.cs — reference SPEC-17.3
- [ ] T066 [P] [US3] Implement RogueGhostSystem: Free Action activation (2 PE/turn), stealth roll, per-enemy detection, alpha-strike bonus, immediate deactivation after attack, weapon restrictions in Scripts/Combat/Abilities/RogueGhostSystem.cs — reference SPEC-17.4

**Checkpoint**: All 4 class abilities functional in headless combat. Warrior stat boost, Paladin shield/turret, Priest repair/cover, Rogue stealth.

---

## Phase 6: US6 — Weapon Effects & Grenades (extends M3)

**Goal**: Weapon effects resolve correctly. Dual-wield works. Grenades apply tactical effects.

**Independent Test**: Equip different weapons, use grenades in combat, verify effects.

**Dependencies**: Phase 3 (core combat resolvers)

- [ ] T067 [US6] Implement WeaponEffectResolver for all YES-scoped effects: Barrage X, Choc X, Dispersion X, Dégâts Continus X, Lumière X, Parasitage X, Ultraviolence, Perce Armure, Pénétrant, Destructeur, Meurtrier, Silencieux, Tir en Sécurité, Assassin, Cadence, Deux Mains, Lourd, Chargeur in Scripts/Combat/Resolvers/WeaponEffectResolver.cs — reference SPEC-20
- [ ] T068 [US6] Implement DispersionResolver: 8-position linear track blast zone, friendly fire calculation, confirmation requirement in Scripts/Combat/Resolvers/DispersionResolver.cs — reference SPEC-20.5
- [ ] T069 [US6] Implement dual-wield system: Ambidextre (2 separate attacks, -3 dice, Jumelé → -1), Akimbo (1 combined, -3 dice, Jumelé → -1), suppress with both weapons in Scripts/Combat/Resolvers/ComboRollResolver.cs — reference SPEC-04
- [ ] T070 [US6] Implement assistance system: up to 3 allies, 1 Movement Action each, unique Characteristic per assistant, Solitaire +1 threshold, PEs penalty per assistant in Scripts/Combat/Resolvers/ComboRollResolver.cs — reference SPEC-04
- [ ] T071 [US6] Create no-op stubs for SCAFFOLDED weapon effects (~10: En Chaîné, Fureur, Artillerie, Désignation, Cadence, etc.) with interface + empty body in Scripts/Combat/Resolvers/WeaponEffectResolver.cs — reference SPEC-20, plan.md stub matrix

**Checkpoint**: All demo weapon effects resolve. Dual-wield and grenades work in combat.

---

## Phase 7: US7 — Enemy Faction Behaviors (M6 — SPEC-16 Phase 6)

**Goal**: 4 enemy factions with distinct AI behaviors. Bande swarm mechanics work.

**Independent Test**: Run combat against each faction, verify AI patterns and special attacks.

**Dependencies**: Phase 3 (core combat), Phase 6 (weapon effects for enemy weapons)

- [ ] T072 [US7] Implement EnemyAIController: 4-phase turn execution (target selection → movement → attack → capacities), priority sort with tiebreakers in Scripts/Combat/AI/EnemyAIController.cs — reference SPEC-18.11
- [ ] T073 [US7] Implement HybridAIPattern: melee/ranged preference weighting at spawn, per-turn situational overrides in Scripts/Combat/AI/HybridAIPattern.cs — reference SPEC-18.12
- [ ] T074 [US7] Implement enemy attack procedure: full Aspect dice (no combo), Exceptionnel auto-successes, Exploit, Failure Critique (isExposed flag) in Scripts/Combat/AI/EnemyAIController.cs — reference SPEC-18.9
- [ ] T075 [US7] Implement Charge Brutale: once per combat/phase, gap ≤ 4 range, teleport to adjacent rank, pre-baked damage + 2× Bête in Scripts/Combat/AI/EnemyAIController.cs — reference SPEC-18.13
- [ ] T076 [US7] Implement Peur mechanic: encounter start per Peur source, combo roll vs floor(highestAspect/2), failure = dice/Def/React penalty, Échec Critique = paralyzed in Scripts/Combat/AI/EnemyAIController.cs — reference SPEC-18.10
- [ ] T077 [US7] Implement BandeController: Cohesion pool, Violence-only damage, Débordement (turn × score) to all opposing, initiative 1, One Bande Rule, Nocte capacities in Scripts/Combat/AI/BandeController.cs — reference SPEC-11
- [ ] T078 [US7] Implement DespairAIController: hostile knight targeting (random ally, forced Agressif) in Scripts/Combat/AI/DespairAIController.cs — reference SPEC-07
- [ ] T079 [US7] Write EditMode tests for BandeController (Cohesion, Débordement escalation, Violence-only) in Tests/EditMode/BandeControllerTests.cs — reference SPEC-11 ACs

**Checkpoint**: All 4 factions functional. Bande swarms, beasts charge, Dame targets morale, Ophidien debuffs.

---

## Phase 8: US4 — Mission Flow & Boss (M7 — SPEC-16 Phase 7)

**Goal**: 7-node mission with branching, node transitions, boss phase transition, save system, narrative events.

**Independent Test**: Run full mission node 1 through node 5, verify resource carryover and boss phase transition.

**Dependencies**: Phase 7 (enemy AI for encounters), Phase 3 (combat for node resolution)

- [ ] T080 [US4] Implement BossPhaseController: P1→P2 on PS=0 (override Agony), apply P2 stats, unlock new abilities, -1D6 PEs all knights, Bande reinforcement, Charge cooldown reset in Scripts/Combat/BossPhaseController.cs — reference SPEC-14
- [ ] T081 [US4] Implement MissionManager: 5-node graph with branching at node 2, combat end = all enemies eliminated, no retreat from Node 5 in Scripts/Meta/MissionManager.cs — reference SPEC-22
- [ ] T082 [US4] Implement node transition logic: no auto-recovery, state carries over, Nod free use between nodes, dead stay dead, ApplyCombatResults atomic commit in Scripts/Meta/MissionManager.cs — reference SPEC-13, SPEC-32
- [ ] T083 [US4] Implement narrative event system: 4 events (survivor +1D6 PEs, Point Faible reveal, pre-battle prayer, darkness erupts) in Scripts/Meta/MissionManager.cs — reference SPEC-22.3
- [ ] T084 [US4] Implement SaveSystem: serialize RunState to JSON at node transitions and Camelot return, resume = reload pre-combat state + rebuild RNG, single auto-save slot in Scripts/Meta/SaveSystem.cs — reference SPEC-13, FR-035/036/037
- [ ] T085 [US4] Implement MotivationDetector: event bus listener, tag matching across all knights' motivations, +1D6 PEs (minor) or +25 PEs current+max (major), Voeu triggers in Scripts/Combat/MotivationDetector.cs — reference SPEC-12
- [ ] T086 [US4] Implement victory/defeat/retreat conditions: Boss P2 PS=0 = victory, all 4 dead = defeat, retreat from nodes 1–4 in Scripts/Meta/MissionManager.cs — reference SPEC-22.4
- [ ] T087 [US4] Write EditMode tests for BossPhaseController (P1→P2 transition, stat changes, reinforcement spawn) in Tests/EditMode/BossPhaseTests.cs — reference SPEC-14 ACs

**Checkpoint**: Full mission completes: 7 nodes, branching, boss phase transition, save/load works.

---

## Phase 9: US5 — Camelot Hub & Post-Mission Management (M8 — SPEC-16 Phase 8)

**Goal**: Camelot hub closes the gameplay loop. PG spending, injury treatment, recruitment, roster management.

**Independent Test**: Return from mission with injured/dead knights and PG, spend PG on therapy and recruitment.

**Dependencies**: Phase 8 (mission completion provides PG and injured knights)

- [ ] T088 [US5] Implement CamelotHubManager: PS/PA full restore, PE unchanged, auto-refill (grenades 5, Nods 3/3/3), view roster, state machine transitions in Scripts/Meta/CamelotHubManager.cs — reference SPEC-25
- [ ] T089 [US5] Implement Infirmary: Cybernetic Implant (20 PG, remove 1 injury, maxPEs-3, floor 10), Reconstruction Therapy (100 PG, remove all, reset implants, restore maxPEs) in Scripts/Meta/CamelotHubManager.cs — reference SPEC-25
- [ ] T090 [US5] Implement PGEconomyManager: earning table (per node completed, boss bonus) per FR-029b, spending table, running total in RunState in Scripts/Meta/PGEconomyManager.cs — reference SPEC-25.3
- [ ] T091 [US5] Implement squad selection: 4-of-8 selection, lock, transition to deployment in Scripts/Meta/CamelotHubManager.cs — reference SPEC-03/25
- [ ] T092 [US5] Implement recruitment: replace dead knights with procedurally generated recruits (uses GenerationPipeline), campaign end conditions (all 8 dead = game over) in Scripts/Meta/CamelotHubManager.cs — reference SPEC-25.4

**Checkpoint**: Full game loop: Camelot → Mission → Camelot. PG economy works. Dead knights replaceable.

---

## Phase 10: UI Layer (M9 — parallel track)

**Purpose**: All player-facing screens. UI is read-only view + command sender per SPEC-32.

**Dependencies**: Phase 3 (combat systems for HUD), Phase 9 (hub systems for hub screens)

### Combat UI

- [ ] T093 [P] [US1] Implement CombatHUDController with resource bars (PS/PA/CdF/PE/PEs), state badges (Fold/Agony/Despair/Ghost), PEs penalty tooltip in Scripts/UI/Combat/CombatHUDController.cs using Assets/UI/UXML/ — reference SPEC-26/31
- [ ] T094 [P] [US1] Implement InitiativeTimelineView: horizontal timeline with portraits, current turn highlight in Scripts/UI/Combat/InitiativeTimelineView.cs — reference SPEC-26
- [ ] T095 [P] [US1] Implement ActionMenuController: Attack, Move, Nod, Style Switch, Ability, Suppress, End Turn with disabled action reason strings (SPEC-31-AC3) in Scripts/UI/Combat/ActionMenuController.cs — reference SPEC-31
- [ ] T096 [P] [US1] Implement RollBreakdownPanel: dice pool, OD autos, evens, total successes, threshold, hit/miss, per-layer damage absorption in Scripts/UI/Combat/RollBreakdownPanel.cs — reference SPEC-31
- [ ] T097 [P] [US1] Implement CombatLogView: scrollable last ~50 events with formatted entries (SPEC-31-AC2) in Scripts/UI/Combat/CombatLogView.cs — reference SPEC-31
- [ ] T098 [P] [US1] Implement CountdownTimerView: Hémorragie (red pulsing, always visible), Despair, DoT, Choc/Parasitage countdown badges in Scripts/UI/Combat/CountdownTimerView.cs — reference SPEC-31
- [ ] T099 [US1] Implement RankDisplayView: 2D side-view with 4 knight sprites left + 4 enemy sprites right, cover icons, selection highlights, target highlight, range/gap indication in Scripts/UI/Combat/RankDisplayView.cs — reference SPEC-26

### Deployment & Hub UI

- [ ] T100 [P] [US2] Implement DeploymentScreenController: 4 rank slots, drag-drop knight placement, enemy preview, cover layout, confirm button in Scripts/UI/Deployment/DeploymentScreenController.cs — reference SPEC-26
- [ ] T101 [P] [US5] Implement RosterView: roster grid showing all 8 knights with full state (PS/PA/PE/PEs, injuries, equipment, Tarot) in Scripts/UI/Hub/RosterView.cs — reference SPEC-26
- [ ] T102 [P] [US5] Implement InfirmaryPanel: injury list with treatment costs, implant/therapy buttons, maxPEs impact display in Scripts/UI/Hub/InfirmaryPanel.cs — reference SPEC-26
- [ ] T103 [P] [US4] Implement MissionSelectView and NodeMapView: 5-node path, branch at Node 2, party status summary in Scripts/UI/Hub/MissionSelectView.cs — reference SPEC-26
- [ ] T104 [US5] Implement MemorialView for dead knights and campaign end screen in Scripts/UI/Hub/MemorialView.cs — reference SPEC-25.4

### Shared UI

- [ ] T105 [P] Implement DamageNumberPopup (floating TMP numbers) and TooltipSystem in Scripts/UI/Shared/DamageNumberPopup.cs and Scripts/UI/Shared/TooltipSystem.cs

---

## Phase 11: Art & Placeholder Rendering (M10 — parallel track)

**Purpose**: Programmer art sprites and VFX placeholders per NFR-003.

- [ ] T106 [P] Create placeholder knight sprites (colored rectangles with TMP class labels, 4 classes × states: idle/attack/hit/death/fold) in Assets/Sprites/Knights/
- [ ] T107 [P] Create placeholder enemy sprites (colored rectangles with tier labels, 5 types × states: idle/attack/hit/death) in Assets/Sprites/Enemies/
- [ ] T108 [P] Create UI icons (weapon icons, Nod icons, status badges, Tarot card placeholders, portraits) in Assets/Sprites/UI/
- [ ] T109 Setup 2D rendering: SpriteRenderer for combat entities, sorting by rank, fixed orthographic camera fitting 8-rank battlefield, particle systems for damage indicators in Assets/Scenes/Main.unity

---

## Phase 12: Integration Tests & Polish (M11+M12)

**Purpose**: Cross-system integration validation and final polish.

**Dependencies**: All previous phases complete.

- [ ] T110 Write Integration Scenario 1: Fold → Despair → Hémorragie Cascade (SPEC-05/06/08/10/33/18.13/30) in Tests/EditMode/IntegrationScenario1.cs — reference /TechSpec/specifications/15_IntegrationTests.md
- [ ] T111 Write Integration Scenario 2: Ghost Alpha-Strike → Exploit → Dual-Wield (SPEC-17.4/04/05/09/07) in Tests/EditMode/IntegrationScenario2.cs
- [ ] T112 Write Integration Scenario 3: Barrage + Boss Phase Transition (SPEC-20/17.2/14/11/18.13/21.5) in Tests/EditMode/IntegrationScenario3.cs
- [ ] T113 Write Integration Scenario 4: Grenade Friendly Fire + Despair + Stabilization (SPEC-19.4/20.5/20.1/06/07/30/20.3) in Tests/EditMode/IntegrationScenario4.cs
- [ ] T114 Write Integration Scenario 5: Full Node Transition Attrition (SPEC-13/33/32/08) in Tests/EditMode/IntegrationScenario5.cs
- [ ] T115 Write PlayMode smoke test: full mission playthrough (Camelot → 5 nodes → boss → victory → Camelot) in Tests/PlayMode/FullMissionSmokeTest.cs
- [ ] T116 Run quality checks: grep for UnityEngine.Random (must return 0), cross-run determinism (same seed = identical log), IsSquadDefeated single implementation, no SceneManager.LoadScene in combat
- [ ] T117 Full demo playthrough: all paths (victory, defeat, retreat, partial loss), verify edge cases (Despair+Fold, all 4 dead TPK, all 8 dead game over)
- [ ] T118 Balance tuning: Nocte Cohesion values, enemy damage, PG earning rates, Nod economy pressure, boss Régénération balance in Assets/Data/
- [ ] T119 Build PC standalone (Windows + Mac + Linux), verify launch and full loop completion

**Checkpoint**: All 5 integration scenarios green. Full playthrough validated. Build ships.

---

## Dependencies & Execution Order

### Phase Dependencies

```
Phase 1 (Setup) → Phase 2 (Data Layer) → Phase 3 (US1: Combat) → Phase 4 (US2: Roster)
                                                                  → Phase 5 (US3: Abilities)
                                                                  → Phase 6 (US6: Weapons)
                                                                  → Phase 7 (US7: Enemy AI)
                                                                     → Phase 8 (US4: Mission)
                                                                        → Phase 9 (US5: Hub)
Phase 10 (UI) can start after Phase 3 for combat UI, after Phase 9 for hub UI
Phase 11 (Art) can run in parallel from Phase 3 onward
Phase 12 (Integration) requires all phases 1–9 complete
```

### SPEC-16 Phase Mapping

| SPEC-16 Phase | Tasks | User Stories |
|---------------|-------|-------------|
| Phase 1 (Data) | T010–T029 | All (foundational) |
| Phase 2 (Position) | T030–T034 | US1 |
| Phase 3 (Core Combat) | T035–T043, T061–T066, T069–T070 | US1, US3, US6 |
| Phase 4 (Damage) | T038–T040, T067–T068 | US1, US6 |
| Phase 5 (Consequences) | T044–T048 | US1 |
| Phase 6 (Entities) | T072–T079, T085 | US7 |
| Phase 7 (Mission) | T080–T087 | US4 |
| Phase 8 (Meta) | T088–T092 | US5 |

### User Story Dependencies

- **US1 (P1)**: Requires Phase 1 + 2. Core path.
- **US2 (P2)**: Requires Phase 2 data. Can parallel with US1 (generation is data-only).
- **US3 (P3)**: Requires US1 Phase 3 (core combat). Can parallel with US6/US7.
- **US6 (P6)**: Requires US1 Phase 3 (combat resolvers). Can parallel with US3/US7.
- **US7 (P7)**: Requires US1 Phase 3 (combat). Can parallel with US3/US6.
- **US4 (P4)**: Requires US7 (enemy AI for encounters), US1 (combat for nodes).
- **US5 (P5)**: Requires US4 (mission provides PG/injuries).

### Parallel Opportunities

Within each phase, tasks marked [P] can run concurrently:
- **Phase 1**: T003, T004, T005, T006, T007 all target different files
- **Phase 2**: All SO schemas (T010–T019) and all SO instances (T020–T028) are independent files
- **Phase 5**: All 6 ability systems (T061–T066) are independent files
- **Phase 10**: All UI controllers (T093–T105) are independent files
- **Phase 11**: All art tasks (T106–T109) are independent

### Within Each User Story

1. Data schemas → SO instances → Logic systems → Tests
2. Core implementation before integration
3. Tests validate each system independently

---

## Parallel Example: Phase 2 (Data Layer)

```bash
# All SO schemas can be authored in parallel (different files):
T010: KnightBaseSO.cs
T011: ArchetypeDataSO.cs
T012: HautFaitDataSO.cs
T013: BlasonDataSO.cs
T014: TarotCardSO.cs
T015: WeaponProfileSO.cs
T016: EnemyBaseSO.cs
T017: EncounterSO.cs
T018: InjuryTableSO.cs
T019: MotivationTemplateSO.cs + MissionDefinitionSO.cs

# Then all SO instances in parallel:
T020–T028: All asset authoring
```

## Parallel Example: Phase 5 (Armor Abilities)

```bash
# All 6 ability systems are independent files:
T061: WarriorTypeSystem.cs
T062: PaladinShrineSystem.cs
T063: PaladinWatchtowerSystem.cs
T064: PriestNanoCSystem.cs
T065: PriestMechanicSystem.cs
T066: RogueGhostSystem.cs
```

---

## Implementation Strategy

### MVP First (US1 Only — Vertical Slice)

1. Complete Phase 1: Setup (M0)
2. Complete Phase 2: Data Layer (M1)
3. Complete Phase 3: US1 Core Combat (M2+M3+M4)
4. **STOP and VALIDATE**: Headless combat runs end-to-end with all tests green
5. This proves the core game works — everything else layers on top

### Incremental Delivery

1. Setup + Data Layer → Foundation ready
2. US1 (Core Combat) → ★ Vertical slice (MVP!)
3. US2 (Roster) → Knights have identity
4. US3 (Abilities) → Tactical diversity
5. US6 (Weapons) + US7 (Enemy AI) → Rich combat variety
6. US4 (Mission) → Cohesive experience
7. US5 (Hub) → Full gameplay loop closed
8. UI + Art + Integration → Playable demo

### 2-Developer Parallel Strategy

| Developer | Focus | Phases |
|-----------|-------|--------|
| Dev A (Systems) | Architecture + Combat Core + Enemy AI | 1, 3 (T030–T055), 7, 8 (T080–T081) |
| Dev B (Content + State) | Data + Generation + Abilities + Hub + UI | 2, 4, 5, 6, 9, 10 |

---

## Notes

- [P] tasks = different files, no dependencies on incomplete tasks in same phase
- [USn] label maps task to specific user story for traceability
- Constitution Principle VI: Tests are MANDATORY — each system must have EditMode tests
- Constitution Principle VII: SPEC-16 dependency order strictly enforced
- Constitution Principle IX: One commit per task, referencing SPEC number
- All TechSpec files at /TechSpec/specifications/ are authoritative — implement exactly what they prescribe
- Stop at any checkpoint to validate independently
