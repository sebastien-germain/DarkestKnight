# Architecture Guardrails & Core Contracts
> **SPECs:** 16, 28, 29, 30, 32
> **Domain:** `Scripts/Core`
> **Dependencies:** None (cross-cutting contracts)
> **Depended on by:** ALL systems — every agent must respect these guardrails
---
## SPEC-16: System Dependency Map
[v2] SPEC-17 added. [v3] SPEC-10 specified. [v4] DD ranks. [v5] SPEC-18–25 added (enemies, weapons, effects, mission, injury, tarot, hub). [v6] Critical fixes: C1 enemy attack procedure (SPEC-18), C2 Destructeur/Meurtrier corrected (SPEC-06/20), C3 cover system (SPEC-03), C4 tarot draw 5 from 22 (SPEC-02/24).


| Phase | System | Depends On |
| --- | --- | --- |
| 1 | SPEC-01: KnightBase | None |
| 1 | SPEC-02: Generation Pipeline | SPEC-01, SPEC-24 |
| 1 | [v5] SPEC-20: Weapon Effects Glossary | None (reference table) |
| 1 | [v5] SPEC-19: Weapon Data Model & Arsenal | SPEC-20 |
| 1 | [v5] SPEC-18: Enemy Data Model | SPEC-20, SPEC-19 |
| 1 | [v5] SPEC-24: Tarot Advantages/Disadvantages | None (data table) |
| 2 | SPEC-03: Position System | SPEC-01, SPEC-18 |
| 2 | SPEC-15: Turn Queue | SPEC-01, SPEC-18 |
| 3 | SPEC-04: Combo Roll | SPEC-01, SPEC-07 |
| 3 | SPEC-07: Espoir System | SPEC-01 |
| 3 | [v2] SPEC-17: Armor Abilities | SPEC-01, SPEC-03, SPEC-04 |
| 3 | SPEC-33: Nod System | SPEC-01, SPEC-03, SPEC-06, SPEC-07, SPEC-08, SPEC-10 |
| 4 | SPEC-05: Damage Resolver | SPEC-04, SPEC-06, SPEC-19, SPEC-20 |
| 4 | SPEC-06: Armour Layer Resolver | SPEC-01, SPEC-18 |
| 5 | SPEC-08: Injury & Agony | SPEC-05, SPEC-06, SPEC-07 |
| 5 | [v5] SPEC-23: Complete Injury Table | SPEC-08 |
| 5 | SPEC-09: Heroism | SPEC-04, SPEC-08 |
| 5 | SPEC-10: Fold State | SPEC-06, SPEC-17 |
| 6 | SPEC-11: Bande Controller | SPEC-05, SPEC-18 |
| 6 | SPEC-12: Motivation Detector | SPEC-07 |
| 6 | [v5] SPEC-21: Demo Enemy Roster | SPEC-18, SPEC-19, SPEC-20 |
| 7 | SPEC-13: Save System | All above |
| 7 | SPEC-14: Boss Phase | SPEC-11, SPEC-15, SPEC-21 |
| 7 | [v5] SPEC-22: Demo Mission Structure | SPEC-21, SPEC-13, SPEC-14 |
| 8 | [v5] SPEC-25: Camelot Hub & Mission Loop | SPEC-22, SPEC-02, All above |



---

# PART 4 — AGENT HANDOFF ADDENDUM (v8.5 draft)
Purpose
This addendum defines non-negotiable implementation guardrails and missing “contracts” that prevent divergent interpretations between parallel agents. It introduces .
Scope (Demo v1.6)
Applies to mission nodes, DD-rank combat, save/resume behavior, encounter authoring, combatant states, UI minimum info, and Unity project conventions.

## SPEC-28: Deterministic RNG & Node Seeds
### Purpose
Guarantee consistent outcomes for a given run/node to prevent quit-to-reroll, improve debug reproducibility, and stabilize AI simulations.
### Dependencies
SPEC-13 (Save System & Node Transitions), SPEC-15 (Turn Queue & Initiative), all roll systems (SPEC-04/05/06/07/08/14).
### Design Decision (Default Policy)
Default: Deterministic per node attempt using a NodeSeed derived from RunSeed + NodeIdentity.
Quitting mid-combat resumes at node start with the same NodeSeed.
Starting a new run generates a new RunSeed.
Optional toggle (not default): RNGPolicy = FullRNGOnRetry (only if you explicitly want reroll-on-retry behavior).
### Data Schema
enum RNGPolicy { DeterministicNodeSeed, FullRNGOnRetry }

struct RunRngContext {
  uint runSeed; // generated once at run start (unsigned 32-bit, per v8.5 H-2)
  string missionId;
  string branchPathId; // stable hash of player choices (demo: node branch choice)
  RNGPolicy policy; // default DeterministicNodeSeed
}

struct NodeRngContext {
  int nodeIndex; // 0..N
  string nodeId; // stable authoring id
  int attemptIndex; // increments only if policy == FullRNGOnRetry
  uint nodeSeed; // derived
}

enum RngStreamId { Combat, Loot, Cosmetic }

interface IRng {
  uint NextUInt();
  int Range(int minInclusive, int maxExclusive);
  bool Chance01(float p);
}
```

Hash Algorithm (Mandatory)
All seed derivation uses xxHash32 via Unity.Mathematics.math.hash(). This algorithm is deterministic cross-platform, fast (single-cycle throughput), and available natively in Unity via the Mathematics package. Seed types are uint (unsigned 32-bit) throughout — no signed int seeds. Do NOT use System.HashCode or C# string.GetHashCode() as neither is deterministic across .NET runtimes.
Canonical NodeSeed derivation pseudocode:
uint ComputeNodeSeed(uint runSeed, string missionId, string nodeId, string branchPathId, int attemptIndex) { uint h = math.hash(new uint4(runSeed, StableStringHash(missionId), StableStringHash(nodeId), (uint)attemptIndex)); return math.hash(new uint2(h, StableStringHash(branchPathId))); }
StableStringHash: use FNV-1a or djb2 — must produce identical results across all platforms and .NET versions.
Stream derivation: uint StreamSeed(uint nodeSeed, RngStreamId stream) { return math.hash(new uint2(nodeSeed, (uint)stream)); }
IRng implementation: use Xoshiro128** seeded with StreamSeed. Fast, high-quality PRNG with 2¹²⁸ period. Reference: prng.di.unimi.it
SPEC-28-AC5: Two independent agent implementations, given identical RunSeed + node parameters, produce identical nodeSeed values (cross-agent determinism test).

### Rules
No global RNG: UnityEngine.Random is forbidden for gameplay.
NodeSeed derivation (deterministic):
If policy == DeterministicNodeSeed: attemptIndex = 0 always.
If policy == FullRNGOnRetry: attemptIndex++ on retry/restart of the node.
Seed combine: nodeSeed = xxHash32(runSeed, missionId, nodeId, branchPathId, attemptIndex).
Stream separation: each system uses a stream derived from nodeSeed + stream constant.
Combat uses Combat stream only (initiative, hit rolls, damage rolls, AI tie-breaks, status durations).
Loot uses Loot stream only.
Cosmetic may be nondeterministic (allowed), but must not affect gameplay.
Deterministic “tie-breaks”: any “random target among equals” must use Combat RNG.
Save/Resume contract (demo):
Save stores runSeed, missionId, branchPathId, nodeId, and roster state.
Resume from combat node always rebuilds RNG contexts from these fields (no mid-combat RNG state persistence needed for demo).
### Flow
runSeed
nodeSeed and create rngCombat, rngLoot
rngCombat
### Acceptance Criteria
SPEC-28-AC1: With same runSeed + missionId + branchPathId + nodeId, two fresh launches produce identical: initiative order, AI target selection tie-breaks, hit/miss outcomes, damage totals, and status durations.
SPEC-28-AC2: Quitting mid-combat and resuming produces identical outcomes to playing continuously from node start.
SPEC-28-AC3: No gameplay code references UnityEngine.Random. (Static analysis / grep check passes.)
SPEC-28-AC4: Changing RngStreamId.Cosmetic behavior cannot change any gameplay result.

## SPEC-29: Encounter Authoring Schema (EncounterSO)
### Purpose
Define a single canonical encounter format so content/UI/combat agents don’t invent incompatible structures.
### Dependencies
SPEC-03 (Position System), SPEC-11 (Bandes), enemy roster specs (SPEC-21/22), SPEC-14 (Boss phases).
### Data Schema
[CreateAssetMenu]
```csharp
class EncounterSO : ScriptableObject {
  string encounterId; // stable
  string missionId;
  int nodeIndex;
  EncounterType type; // Combat / Boss / Event / Rest (demo: Combat+Boss)
  CoverLayout cover; // per rank

  PlayerStartLayout playerStart; // optional restrictions (demo: free)

  List<EnemySpawnEntry> initialEnemies; // max 4 on-track
  ReinforcementQueue reinforcement; // optional
  BandeConfig bande; // optional off-track

  List<EncounterTrigger> triggers; // optional minimal scripting
}

struct CoverLayout { bool r1, r2, r3, r4; }

struct EnemySpawnEntry {
  string enemyId;
  int startRank; // 1..4
  bool startsInCover; // if true, requires cover at that rank
  bool isElite; // informational tag
}

struct ReinforcementQueue {
  List<string> enemyIdsInOrder;
  ReinforcementRule rule; // WhenSlotFree / OnTurnN / OnTrigger
  int turnN; // used if OnTurnN
}

struct BandeConfig {
  bool enabled;
  string bandeId; // for UI/log
  int cohesionMax;
  int cohesionStart;
  int debordementScore; // flat integer per SPEC-11 (e.g. Nocte = 4). Damage = débordementTurn × debordementScore
  // plus any special flags already defined in SPEC-11
}

enum TriggerEvent { OnCombatStart, OnTurnStart, OnEnemyDeath, OnBossPhaseChanged }
enum TriggerActionType { SpawnReinforcement, ApplyEffect, ChangeCover, SwapAiProfile }

struct EncounterTrigger {
  // TriggerEvent ev removed — event type stored in TriggerCondition.triggerEvent only
  TriggerCondition cond;
  TriggerAction action;
}
```

The following structs define the fields used by EncounterTrigger.cond and EncounterTrigger.action:
Lightweight Generic Trigger Schemas:
struct TriggerCondition { TriggerEvent triggerEvent; string filterEnemyId; // null = match all. int filterTurnIndex; // -1 = any turn (used with OnTurnStart). int filterPhaseIndex; // -1 = any phase (used with OnBossPhaseChanged). }
struct TriggerAction { TriggerActionType actionType; string targetEnemyId; // SpawnReinforcement: which enemy to spawn. int targetRank; // SpawnReinforcement: rank to spawn at (0 = auto per SPEC-03 empty slot rules). string effectId; // ApplyEffect: effect identifier. int value; // Generic int payload (cohesion amount for bande, cover rank index, etc.). string aiProfileOverride; // SwapAiProfile: new AI profile id. }
Unused fields per action type are ignored. Demo usage: Ours Corrompu Phase 2 Nocte respawn uses TriggerEvent=OnBossPhaseChanged, filterPhaseIndex=2, actionType=SpawnReinforcement, targetEnemyId=nocte_bande, value=200 (cohesion).

### Rules
Initial on-track enemy limit: initialEnemies.Count <= 4.
Rank validity: ranks must be 1..4.
Cover consistency: startsInCover requires cover at that rank.
Reinforcement rule constraint: reinforcement spawns must never exceed 4 on-track enemies.
Bande constraint: bande is off-track, cannot occupy ranks, uses SPEC-11 resolution.
Authoring stability: encounterId and nodeId must be stable and never regenerated once shipped (seed derivation relies on it).
### Flow

During combat: reinforcement system listens to slot-free/turn/trigger events and spawns from queue
### Acceptance Criteria
SPEC-29-AC1: Invalid encounter data fails validation with a clear error message (which rule, which field).
SPEC-29-AC2: EncounterSO fully defines combat setup without hardcoded scene logic.
SPEC-29-AC3: Reinforcement queue never spawns if it would exceed 4 on-track enemies.

## SPEC-30: Canonical Combatant State Model & Defeat Evaluation
### Purpose
Prevent inconsistent “incapacitated / ally / defeat” handling across systems (Agony, Despair, Fold, Death).
### Dependencies
SPEC-07 (PEs), SPEC-08 (Injury & Agony), SPEC-10 (Fold), SPEC-11 (Bandes), SPEC-15 (Turn Queue).
### Data Schema
```csharp
enum LifeState { Alive, Dead }
enum ControlState { PlayerControlled, EnemyControlled } // who decides actions
enum ActionState { CanAct, Incapacitated } // can take turn actions
enum ArmorState { Normal, Folded } // from SPEC-10

struct CombatantState {
  LifeState life;
  ControlState control;
  ActionState action;
  ArmorState armor;
  bool isDespair; // derived from PEs system
  bool isDespairPermanent; // if stabilized failed or per spec rule
  int despairDuration; // -1 = not in despair, 1D6 initial, decrements per turn, 0 = permanent
  int hemorragieCountdown; // -1 = inactive, 3..1 = turns remaining before permadeath (SPEC-08). Affects targeting priority.
}
```


### Rules
Dead: life == Dead
Agony: sets action = Incapacitated.
Despair: sets control = EnemyControlled (temporary or permanent as per SPEC-07/08), but does not automatically mean incapacitated unless explicitly stated elsewhere.
Fold: armor = Folded affects available actions/modules per SPEC-10, but does not change action unless another rule does.
Single authoritative defeat function used everywhere:
Single authoritative defeat function — canonical implementation below.

```csharp
(Canonical implementation — this is the ONLY defeat evaluation function, no other code path may determine squad defeat: bool IsSquadDefeated(List<CombatantState> knights) { var living = knights.Where(k => k.life == Alive); if (!living.Any()) return true; return living.All(k => k.action == Incapacitated || (k.isDespair && k.isDespairPermanent && k.control == EnemyControlled)); })
``` Temporary Despair (isDespairPermanent == false) does NOT count as defeated — the knight may still be stabilized. Only permanent Despair (duration expired, isDespairPermanent == true) counts toward squad defeat. 
```
### Acceptance Criteria
SPEC-30-AC1: Every system queries CombatantState (not ad-hoc flags) to decide: can act, is ally/enemy, is targetable.
SPEC-30-AC2: Only one implementation of IsSquadDefeated() exists and is used by CombatManager.
SPEC-30-AC3: A squad with 2 living knights — one Incapacitated (Agony), one in permanent Despair (EnemyControlled) — evaluates IsSquadDefeated() == true. Same squad where the Despaired knight has despairDuration > 0 (not yet permanent): IsSquadDefeated() == false (the Despaired knight may still be stabilized).

---

> **SPEC-31 (UI Minimum Information Contract)** is defined in [11_UIAndPresentation.md](./11_UIAndPresentation.md). Do not duplicate here — refer to the canonical source.

## SPEC-32: Unity Architecture Guardrails & Project Conventions
### Purpose
Prevent merge-incompatible architectures across agents; standardize ownership of truth, data vs runtime, and state transitions.
### Global Rules
Single Source of Truth
CombatManager owns: turn queue, legality checks, resolution, applying effects, victory/defeat.
UI is read-only view + command sender, never mutates combat state directly.
Data vs Runtime
Authoring uses ScriptableObjects (KnightBase, EncounterSO, Enemy/Weapon defs).
Runtime state uses plain C# structs/classes (serializable where needed).
No gameplay logic in MonoBehaviour Update()
Use explicit state machine ticks or coroutines owned by managers.
Determinism rule
All gameplay randomness uses SPEC-28 RNG only.
**Naming Convention (French/English):**
- **Enum values:** Use French names from the tabletop source material (e.g., `AGRESSIF`, `DEFENSIF`, `MISE_A_COUVERT`, `SOIN`, `ARMURE`, `ENERGIE`). Exception: universally understood English terms are acceptable (e.g., `STANDARD`, `RANDOM`).
- **Field/variable names:** Use English (e.g., `isInAgony`, `isDead`, `currentPS`, `activeStyle`). No French field names.
- **Display/UI strings:** Use French (the game's localization language for the tabletop adaptation). All player-facing text is in French.
- **Class/struct names:** Use English (e.g., `KnightBase`, `BandeController`, `ArmourLayerResolver`). French nouns may appear in domain-specific type names where the English equivalent would be confusing (e.g., `BandeController` not `SwarmController`, `NodInventory` not `HealingPodInventory`).
### Recommended Project Structure (minimal)
Scripts/Core (state machine, services, RNG, logging)
Scripts/Combat (CombatManager, resolvers, effects, AI)
Scripts/Data (SO definitions)
Scripts/UI (views, presenters, widgets)
Scripts/Meta (hub, injuries, progression)
Assets/Data (SO instances: enemies, encounters, missions)
### Game State Machine (demo)
State transitions are explicit; no scene spaghetti required (single-scene recommended for demo). Combat State Isolation (Single-Scene Rule): Even in single-scene architecture, combat and hub operate in fully isolated state contexts. (1) CombatSession — created on Deployment entry, destroyed on combat exit. Owns: turn queue, all runtime CombatantState instances, active effects, RNG streams (SPEC-28), encounter runtime data. No reference to CombatSession may persist after combat exit. (2) HubSession — created on Hub entry, destroyed on Hub exit. Owns: shop state, UI navigation, treatment queue. (3) RunState (shared, persistent) — owns: knight roster (KnightBase[8]), inventory, PG total, injury list, mission progress, PEs values. This is the ONLY state that crosses the Hub↔Combat boundary. (4) Read-only contract during combat: RunState is snapshot-copied into CombatSession at combat start. Combat mutates the SESSION copy only. On combat exit, mutations are batched and applied back to RunState in a single atomic commit (ApplyCombatResults). This prevents partial state corruption on crash. (5) No direct cross-references between CombatSession and HubSession. Communication goes through RunState or the global state machine. SPEC-32-AC5: Destroying CombatSession after combat produces zero NullReferenceExceptions in Hub systems.

#### [v8.9] RunState vs CombatSession — Explicit Field Boundary

The following table defines exactly which KnightBase fields belong to RunState (persistent, survives combat exit) vs CombatSession (ephemeral, created at combat start, destroyed on exit). This is the authoritative boundary — agents must not guess.

**RunState fields** (snapshot-copied INTO CombatSession at combat start, written BACK on combat exit via `ApplyCombatResults`):

| Field | Persists between missions? | Notes |
|-------|---------------------------|-------|
| `knightId`, `displayName`, `callsign` | Yes | Identity — immutable after generation. |
| `archetype`, `drawnCards`, `activeAdvantages`, `activeDisadvantage` | Yes | Generation data — immutable. |
| `hautFait`, `blason`, `minorMotivations`, `majorMotivation` | Yes | Generation data — immutable (except MJ fulfillment, future). |
| `chair`, `bete`, `machine`, `dame`, `masque` (Aspects) | Yes | Mutable only by Camelot training/therapy. |
| `characteristics[15]` | Yes | Mutable by injuries (permanent) and training. |
| `armorClass`, `maxPA`, `maxPE`, `baseCdF`, `odLevels[15]`, `slotCounts` | Yes | Meta-Armor — immutable after assignment. |
| `currentPS`, `maxPS` | Yes | Persists between combats and missions. Modified by combat damage, Nods, rest. |
| `currentPA` | Yes | Persists. Restored at Camelot or by Nod d'Armure between nodes. |
| `currentPE` | Yes | Persists. Restored at Camelot or by Nod d'Énergie between nodes. |
| `currentPEs`, `maxPEs` | Yes | Persists. Modified by motivation triggers, kills, implants, therapy. |
| `heroisme` | Yes | Persists across combat encounters within a mission. Resets at mission start. |
| `currentCdF` | Yes | Persists. Usually = baseCdF unless modified by effects. |
| `activeInjuries` | Yes | Permanent injuries persist until implant/therapy. Hémorragie is non-permanent (SPEC-08). |
| `implantCount` | Yes | Modified at Camelot Infirmary only. |
| `equippedWeapons`, `activeWeaponIndex`, `activeProfileIndex` | Yes | Equipment loadout persists. |
| `equippedModules` | Yes | (Out of demo scope.) |
| `nods` | Yes | Quantity persists. Refilled at Camelot between missions. |
| `isDead` | Yes | Permanent. |

**CombatSession-only fields** (created fresh at combat start, NOT written back to RunState — they are ephemeral):

| Field | Reset value at combat start | Notes |
|-------|----------------------------|-------|
| `position` | Set by deployment (SPEC-03) | Rank 1–4. Meaningless outside combat. |
| `activeStyle` | `Standard` | Reset to Standard at each combat start. |
| `isInAgony` | `false` | Combat state only. If knight exits combat in Agony, `currentPS = 0` is written back. |
| `isInFoldState` | `false` | Combat state only. Fold is exited between combats (armor repaired to currentPA). |
| `isDespair` | `false` | Combat state only. If Despair persists at combat end, PEs value is written back. |
| `isGhostActive` | `false` | Combat state only. Ghost is disabled at combat end and deactivates between encounters. No stealth checks exist outside combat in demo. |
| `hasHémorragie`, `hémorragieCountdown` | `false`, `-1` | **Exception:** If Hémorragie is active between combat nodes (SPEC-08), these ARE carried forward via RunState. |
| `activeWarriorTypeIndex` | `null` | Type deactivates between encounters. |
| `activeStatusEffects` | Empty list | **All status effects are cleared** at combat start. No status persists between encounters. |
| Turn queue, initiative order | Computed fresh | Per SPEC-15. |
| Active Barrage debuffs | Empty | Per SPEC-20. |
| Choc/Parasitage counters | 0 | Cleared between encounters. |
| Shrine active state | `false` | Priest must re-place shrine each combat. |
| Motivation trigger tracking per encounter | Reset | `espritDeContradictionUsed`, etc. |

**ApplyCombatResults — Fields written back atomically on combat exit:**
```csharp
// Pseudocode — the ONLY place where CombatSession mutates RunState
void ApplyCombatResults(RunState run, CombatSession session) {
  for each knight in session.deployedKnights:
    run.knights[knight.id].currentPS = knight.currentPS;
    run.knights[knight.id].currentPA = knight.currentPA;
    run.knights[knight.id].currentPE = knight.currentPE;
    run.knights[knight.id].currentPEs = knight.currentPEs;
    run.knights[knight.id].currentCdF = knight.currentCdF;
    run.knights[knight.id].heroisme = knight.heroisme;
    run.knights[knight.id].activeInjuries = knight.activeInjuries; // includes new injuries
    run.knights[knight.id].isDead = knight.isDead;
    run.knights[knight.id].nods = knight.nods; // Nods consumed in combat
    // Hémorragie: if still active, carry forward
    run.knights[knight.id].hasHémorragie = knight.hasHémorragie;
    run.knights[knight.id].hémorragieCountdown = knight.hémorragieCountdown;
  // Mission-level
  run.pgTotal += session.earnedPG;
  run.missionProgress = session.updatedProgress;
}
```

**Key rules:**
1. Combat NEVER directly modifies Aspects, Characteristics, OD levels, maxPA, maxPE, baseCdF, equipment, or generation data. Those are modified only at Camelot Hub.
2. If combat is aborted (crash, quit), NO RunState changes occur — the atomic commit never fires. Player resumes at node start with pre-combat RunState.
3. Hémorragie is the one "combat state" that bleeds (pun intended) across encounters via RunState. All other status effects are ephemeral.
### Serialization (demo)
Save stores: RunSeed context, current node id, roster runtime state, inventory/ammo, injuries, hub progress.
No mid-combat serialization required (per your design); resume at node start.
### Acceptance Criteria
SPEC-32-AC1: Combat can run headless (no UI) via unit/integration tests using EncounterSO + roster state.
SPEC-32-AC2: UI does not reference or mutate internal combat collections directly (no List<T> exposed).
SPEC-32-AC3: Grep check: no UnityEngine.Random in gameplay assemblies.
SPEC-32-AC4: All state transitions pass through the state machine (no direct scene loads in combat logic).
