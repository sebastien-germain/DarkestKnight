# Enemy System: Data Model, AI, Bandes & Boss Phases
> **SPECs:** 11, 14, 18 (incl. 18.12 Hybrid AI)
> **Domain:** `Scripts/Combat` + `Scripts/Data`
> **Dependencies:** [03_CombatRolls.md](./03_CombatRolls.md) (SPEC-04, SPEC-05), [07_WeaponsAndEffects.md](./07_WeaponsAndEffects.md) (SPEC-19, SPEC-20), [02_PositionAndTurns.md](./02_PositionAndTurns.md) (SPEC-03, SPEC-15)
> **Depended on by:** [04_ArmorAndState.md](./04_ArmorAndState.md) (SPEC-06), [08_ContentData.md](./08_ContentData.md) (SPEC-21)
---
## [v5] SPEC-18: Enemy Data Model (EnemyBase)
### Purpose
Runtime data structure for all enemy combatants. Mirrors KnightBase symmetry. Every enemy in the game — from disposable bande fodder to the final patron — instantiates from this model.

### 18.1 Enemy Tier System
The tabletop defines 5 enemy tiers with distinct mechanical profiles. Each tier determines which stat fields are populated and how damage is received.

| Tier | Fr. Name | Description | HP Type | Individual? | Example |
| --- | --- | --- | --- | --- | --- |
| T1 | Bande | Swarm / mob | Cohésion pool | No (group entity) | Nocte, Désespéré |
| T2 | Hostile | Named grunt | PS (health) | Yes | Bestian, Humain Commun |
| T3 | Salopard | Elite fighter | PS + PA optional | Yes | Faune, Enfant de la Chair |
| T4 | Colosse | Huge tank | PS + PA + Colosse rule | Yes | Behemot |
| T5 | Patron | Boss | PS + PA + Bouclier + Point Faible | Yes | Ours Corrompu, Le Prédateur |


### 18.2 Core Data Schema
```csharp
class EnemyBase : ScriptableObject {
  string enemyId; string displayName;
  EnemyTier tier; // T1_BANDE, T2_HOSTILE, T3_SALOPARD, T4_COLOSSE, T5_PATRON
  string seigneur; // LaBete, LaMachine, LaChair, LaDame, LeMasque

  // Aspects (all 5, always present)
  int chair, bete, machine, dame, masque; // 0–20

  // Aspects Exceptionnels (nullable)
  AspectExceptionnel chairExc, beteExc, machineExc, dameExc, masqueExc;
  // struct AspectExceptionnel { bool isMajeur; int score; }
  // score (1–10): +N auto-successes on all tests using this aspect
  // Majeur includes Mineur. Each aspect has UNIQUE Mineur/Majeur effects — see §18.3

  // Derived Values (all tiers)
  int defense;    // Opposition threshold for melee
  int reaction;   // Opposition threshold for ranged
  int initiative;

  // Tier-dependent health
  // T1 Bande:
  int cohesion;    // 100/200/300/450 — reduced by Violence
  int debordement; // Cumulative damage per turn (see SPEC-11)

  // T2+ Individual:
  int maxPS; int currentPS; // Health (Points de Santé)
  int maxPA; int currentPA; // Armor (Points d'Armure) — [v8] default 0 if not specified
  int bouclier; // Shield — [v8] default 0 if not specified. T3+ only

  // T4 Colosse special:
  bool isColosse; // If true: 10 raw damage = 1 effective PS/PA loss

  // [v8.9] Displacement immunity (per-enemy, any tier):
  bool immuneToForcedDisplacement; // If true: all forced displacement effects (Choc push, charge knockback, any capacity that forces rank change) are negated. The combatant stays at their current rank. Voluntary movement is unaffected. Checked by the displacement resolver before applying any forced rank change. Default: false. Demo usage: Behemot (T4) has this set to true — too massive to push.

  // [v8.9] Point Faible (any tier, per-enemy):
  string pointFaible; // Characteristic name (e.g. 'Endurance'). Nullable — not all enemies have one.
  
  // T5 Patron special:
  BossPhase[] phases; // Phase transitions (see SPEC-14)

  // Combat capabilities
  WeaponProfile[] weapons;     // Contact and/or ranged
  EnemyCapacity[] capacities;  // Special abilities
  EnemyAI aiProfile;           // Targeting logic

  // Runtime state
  int ddRank; // 1–4 in DD rank system
  List<StatusEffect> activeStatusEffects;
  int combatActionsRemaining; // [v8.9] Base 1 + N from Actions Multiples. Movement Action is always 1 (separate).
  int movementActionsRemaining; // [v8.9] Always 1. Not affected by Actions Multiples.
  bool isDead;
}
```


### 18.3 Aspects Exceptionnels Rules
[v8.9] Many T3+ enemies have exceptional aspects. Each of the 5 aspects has **unique** Mineur and Majeur effects. Majeur always includes Mineur (if an enemy has Majeur, it also gets the Mineur benefit). Exceptionnel score ranges from 1–10 and adds as automatic successes on ALL tests using that aspect (like knight ODs).  

**General rule (all aspects):** Exceptionnel score (N) = +N automatic successes on all rolls using this aspect (attack, resist, capacity). Functions identically to knight OD auto-successes.

#### Per-Aspect Exceptionnel Effects

**Chair Exceptionnelle:**

| Level | Effect | Implementation |
|-------|--------|----------------|
| Mineur (N) | Subtract Chair Exceptionnel score from ALL incoming damage AND violence. | Applied as flat reduction in ArmourLayerResolver Step 4 (SPEC-06): `remaining -= min(target.chairExc, remaining)`. Applies to every hit, including DoT ticks. |
| Majeur (N) | Same as Mineur PLUS: **Ignore** Barrage X, Choc X, and Meurtrier weapon effects entirely. | When resolving weapon effects, skip Barrage, Choc, and Meurtrier if target has Chair Majeur. Meurtrier bonus dice are NOT rolled. |

**Bête Exceptionnelle:**

| Level | Effect | Implementation |
|-------|--------|----------------|
| Mineur (N) | Add Exceptionnel score (N) as flat bonus to contact weapon damage. | Pre-baked in weapon stat block. Do NOT add at runtime. |
| Majeur (N) | Same as Mineur PLUS: add full Bête Aspect score as additional flat bonus to contact weapon damage. | Pre-baked in weapon stat block. Total contact flat bonus = N + Bête score. Example: Faune Bête 10, Majeur(6) → +6 + 10 = +16 (already in "4D6+16"). |

**Machine Exceptionnelle:**

| Level | Effect | Implementation |
|-------|--------|----------------|
| Mineur (N) | Add Machine Exceptionnel score to **Reaction**. | Pre-baked in stat block Reaction value. Do NOT add at runtime. |
| Majeur (N) | Same as Mineur PLUS: **Immune** to sensory-based abilities — Ghost mode (SPEC-17.4), invisibility, illusions, lures. The enemy sees through all deception. | When checking Ghost stealth threshold vs this enemy: Ghost auto-fails. Enemy always detects stealthy targets. |

**Dame Exceptionnelle:**

| Level | Effect | Implementation |
|-------|--------|----------------|
| Mineur (N) | Can redirect damage it receives to allied PNJs within Courte range. | On taking damage, if allied PNJ is within gap ≤ 2: transfer damage to ally instead. AI decides when to use (preserve high-value target). |
| Majeur (N) | Same as Mineur PLUS: knight wanting to attack this PNJ must first pass a **Hargne test** (Combo Roll, Base Hargne) with difficulty = Dame Exceptionnel score. Failure = subjugated, cannot attack this PNJ this turn, turn is wasted. | Before attack resolution: attacker rolls Hargne-based Combo Roll. Successes must exceed N. On failure: attack is cancelled, Combat Action is consumed. |

**Masque Exceptionnelle:**

| Level | Effect | Implementation |
|-------|--------|----------------|
| Mineur (N) | Add Masque Exceptionnel score to **Defense**. | Pre-baked in stat block Defense value. Do NOT add at runtime. |
| Majeur (N) | Same as Mineur PLUS: **Always acts first** in initiative order, as if initiative = 30. No initiative roll needed. | Set `initiative = 30` in initiative queue. Bypasses normal initiative calculation. If multiple Masque Majeur enemies exist, they act in order of their Masque scores (highest first). |

#### Pre-Baked vs Runtime Values

Some Exceptionnel effects are **pre-baked** into stat block values (agents must NOT add them again):
- Bête Mineur/Majeur damage bonuses → already in weapon flat bonus
- Machine Mineur Reaction bonus → already in stat block Reaction
- Masque Mineur Defense bonus → already in stat block Defense

Some Exceptionnel effects are **runtime mechanics** (agents must implement):
- Chair Mineur flat damage reduction → ArmourLayerResolver Step 4 (SPEC-06)
- Chair Majeur effect immunity → weapon effect resolution (skip Barrage/Choc/Meurtrier)
- Machine Majeur sensory immunity → Ghost detection check
- Dame Mineur damage redirect → damage resolution
- Dame Majeur Hargne test → pre-attack check
- Masque Majeur initiative override → initiative queue

#### Demo Enemy Exceptionnel Summary

| Enemy | Exceptionnels | Runtime effects to implement |
|-------|--------------|------------------------------|
| Nocte (T1) | None | — |
| Bestian (T2) | Bête Mineur (2) | None (pre-baked in weapon +2 contact dmg, +2 auto-successes) |
| Faune (T3) | Bête Majeur (6), Masque Mineur (1) | Bête: pre-baked. Masque: pre-baked in Defense. |
| Behemot (T4) | Chair Majeur (5), Bête Majeur (6) | **Chair Mineur: −5 flat dmg/violence reduction on ALL incoming hits. Chair Majeur: immune to Barrage, Choc, Meurtrier.** Bête: pre-baked. |
| Ours P1 (T5) | Chair Mineur (3), Bête Majeur (6) | **Chair Mineur: −3 flat dmg/violence reduction on ALL incoming hits.** Bête: pre-baked. |
| Ours P2 (T5) | Same as P1 | Same as P1. |

[v8.9] **Defense/Reaction values are authoritative per stat block.** Each enemy's Defense and Reaction are hand-authored values listed in SPEC-21 stat blocks (already include Machine Mineur Reaction bonus and Masque Mineur Defense bonus). Do NOT derive them from a formula. Agents must read `defense` and `reaction` from the EnemyBase data and never recalculate them at runtime.

### 18.4 Bouclier vs Champ de Force
Bouclier functions identically to knight CdF (subtracted from raw damage before PA/PS), but with one critical difference:
Pénétrant X ignores CdF but does NOT ignore Bouclier.
Anti-Anathème ignores Bouclier AND Chair Exceptionnelle.
Bouclier regenerates only if a specific capacity says so (it does not auto-regen).

### 18.5 Colosse Damage Rule (T4)
Colosses require complete tranches of 10 raw damage to lose 1 PS or PA. Remainder is discarded.


Anti-Véhicule weapon effect bypasses this rule: damage applies normally (1:1).

### 18.6 Point Faible
[v8.9] Any enemy may have a declared weak point (a Characteristic), defined per-enemy at design time. There is no tier restriction — Point Faible is not limited to T5 Patrons. Each enemy's stat block specifies whether it has a Point Faible and which Characteristic it is. When a knight attacks using that Characteristic in their combo:
+1D6 bonus damage (additive, rolled and summed).
Ignore Bouclier for that attack.
[v8] UI must display the Point Faible once discovered. Discovery in demo: only random narrative events during mission nodes can reveal it. There is no in-combat discovery, no auto-reveal at any specific node, and no guaranteed discovery path. **Design intent:** This limitation is intentional for the demo. The low discovery rate (1/6 at Node 4) creates strategic tension and makes Point Faible a bonus rather than an expectation. In the full game, additional discovery paths will be added requiring strategic preparation: Warmaster class abilities, analysis modules, intelligence gathering at Camelot, and multi-mission reconnaissance. These future paths are designed to reward pre-mission planning and team composition. In-combat Perception-based discovery is NOT available in the demo — the tabletop allows this via the Warmaster class or specific modules, neither of which are in demo scope. Future implementation note: when Warmaster is added, discovery requires 1 Combat Action + Combo Roll (Perception Base), difficulty per tabletop rules.

### 18.7 Enemy AI Profile (EnemyAI)
Each enemy has a targeting profile that determines behavior in combat. This replaces a human GM's tactical decisions.
```csharp
struct EnemyAI {
  TargetPriority primary;   // Which knight stat to sort by
  TargetPriority secondary; // Tiebreaker
  bool preferHighest;       // true = attack highest stat, false = lowest
  RankBehavior rankBehavior; // HOLD, ADVANCE, RETREAT, FLANK
  int aggressionLevel;      // 1–3 (1=cautious, 3=suicidal)
}

enum TargetPriority {
  HIGHEST_COMBAT, HIGHEST_BETE, HIGHEST_DAME, HIGHEST_AURA,
  LOWEST_PS, LOWEST_PA, LOWEST_PEs, NEAREST_RANK, RANDOM
}

enum RankBehavior {
  HOLD,    // Stay in current rank
  ADVANCE, // Move toward Rank 1 (front) if empty
  RETREAT, // Move toward Rank 4 (back) if empty
  FLANK    // Target rear-rank knights if possible
}
```


### 18.8 Actions Multiples
[v8.9] T3+ enemies may have Actions Multiples (N): grants **N extra Combat Actions** per turn. Movement Actions are NOT affected — the enemy always has exactly 1 Movement Action per turn regardless of Actions Multiples.

**Standard turn (no Actions Multiples):** 1 Combat Action + 1 Movement Action.
**With Actions Multiples (N):** (1 + N) Combat Actions + 1 Movement Action.

Since Combat Actions can be used as Movement Actions (SPEC-04), an enemy with extra Combat Actions has increased flexibility — it can trade any Combat Action for a Movement Action if needed (e.g., to close distance).

| Extra Combat Actions (N) | Total Combat Actions (1+N) | Movement Actions | Total Actions | Example |
|--------------------------|---------------------------|-----------------|---------------|---------|
| 0 (standard — no Actions Multiples) | 1 | 1 | 2 | Bestian, Nocte |
| 1 | 2 | 1 | 3 | Faune, Behemot, Ours P1 |
| 2 | 3 | 1 | 4 | Ours P2 |

Each Combat Action can target a different knight. All actions execute at the same initiative count (single turn, single initiative slot — the enemy does NOT get multiple turns in the round).

### 18.9 Enemy Attack Procedure
[v6] NEW — Critical C1 resolution. Complete step-by-step procedure for enemy attack execution. This is the enemy-side mirror of SPEC-04 (knight Combo Roll). Enemies use a fundamentally simpler system: single aspect, no combo, no OD equivalent.

IMPORTANT DISTINCTION:
ATTACKING (enemy's turn): Enemy uses full Aspect score as dice pool.
DEFENDING (opposition score): [v8.9] Enemy uses the `defense` and `reaction` values from the SPEC-21 stat block directly. These are authoritative, hand-authored values — do NOT derive them from aspects or Exceptionnels at runtime. See §18.3 for details.

Step 1 — Select Weapon:
AI profile determines contact vs ranged based on current rank, target availability, and weapon range bands (SPEC-03).
If enemy has multiple weapons, priority: (1) weapon whose range band covers the current target, (2) highest damage weapon if multiple qualify.

Step 2 — Select Aspect:
Each enemy weapon specifies its attack aspect in the stat block. **Attack Aspect is REQUIRED on every enemy weapon profile.** If not explicitly specified, use defaults as fallback for validation/error recovery only:
Contact weapons: highest of Chair or Bête
Ranged weapons: highest of Machine or Masque
The tabletop allows MJ to choose freely; for digital, the aspect is deterministic per weapon. Agents authoring new enemy weapons must always include the Attack Aspect field.

Step 3 — Build Dice Pool:
Pool = full Aspect score (NOT divided by 2).
No combo. PNJ never combo — this is explicitly a player privilege.
No OD equivalent. Aspect Exceptionnel adds auto-successes (Step 5), not dice.

Minimum pool: 0 dice (auto-fail if reduced to 0). Aspect Exceptionnel auto-successes are NOT applied at pool 0 — same rule as knight OD (see SPEC-04).

Step 4 — Roll & Count Successes:
Roll D6s equal to pool size.
Count even results (2, 4, 6) = 1 success each.

Step 5 — Add Aspect Exceptionnel:
If the attack aspect has an Exceptionnel (Mineur or Majeur), add N automatic successes to the total.
Example: Bestian attacks with Bête 12, Bête Exceptionnel Mineur (3). Rolls 12D6, gets 5 evens. Total = 5 + 3 = 8 successes.

Step 6 — Compare vs Target Threshold:
Contact attack: successes > target Defense = HIT.
Ranged attack: successes > target Reaction = HIT.
Must exceed (strictly greater than), not equal.
Exploit (all even): [v8] Enemy exploit — no Heroism earned (enemies never earn Heroism). Mechanically identical to knight Exploit: re-roll entire pool, ADD new successes to original. Combined excess successes above threshold = bonus D6 damage dice (roll and SUM), same as Mode Héroïque resolution. This makes enemy Exploits dangerous — high excess can significantly spike damage.
Failure Critique (all odd): Attack auto-misses. Enemy is exposed — the **first** knight attack against this specific enemy after the Failure Critique gets +2 auto-successes. Once consumed by one attack, the bonus is removed. Track as a boolean `isExposed` flag on the enemy, cleared after the first knight attack resolves against it. If no knight attacks the exposed enemy before its next turn, the bonus expires at the start of that enemy's next turn.

Step 7 — Damage Roll (on hit):
Roll weapon damage dice. SUM all face values + weapon flat bonus. PNJ weapon stat blocks (SPEC-21) include ALL flat bonuses pre-baked (Force/Tir bonus, weapon base bonus). Do NOT add Aspect-derived bonuses again at roll time. Agent implementation: use damageFormula string as-is (e.g. Faune "4D6+16" = roll 4D6, add faces, add 16). No further modifier lookup required for PNJ damage.
**Pre-baking convention:** The pre-baked flat bonus convention applies to **contact** weapons via Bête Exceptionnel (Mineur adds Exceptionnel score, Majeur adds full Bête Aspect score — see SPEC-18.3). **Ranged** enemy weapons: if no flat bonus is listed in the stat block, there is none. No demo enemy has ranged weapon flat bonuses derived from aspects. If a future enemy weapon has Précision or Orfèvrerie, the corresponding Aspect bonus would be pre-baked into the stat block value — agents should never derive ranged flat bonuses at runtime.
[v8.9] **Exception — Charge Brutale (SPEC-18.13):** Charge Brutale adds 2× Bête score as an ADDITIONAL flat bonus on top of the pre-baked weapon damage. This is the only capacity that adds a bonus beyond the pre-baked stat block value. The Charge bonus is separate from the weapon's built-in Exceptionnel bonuses — both apply.
Apply weapon effects per SPEC-20 (Destructeur, Meurtrier, Dégâts Continus, etc.).
[v8] For weapon effects requiring a Characteristic score (Précision adds Tir, Orfèvrerie adds Dextérité): PNJ use full relevant Aspect score for all offensive calculations (attack pools, damage bonuses, capacity bonuses such as Charge Brutale). [v8.9] PNJ Defense/Reaction are read from stat block — see §18.3.

Step 8 — Route total through target's ArmourLayerResolver (SPEC-06).

Same-Name Optimization (performance):
Multiple enemies of the same type MAY share one attack roll and one damage roll per the tabletop rules.
Digital implementation: individual rolls recommended for clear UI feedback, but shared rolls acceptable for large groups.

[v7] 18.11 Enemy AI Decision Flowchart
[v7] Step-by-step enemy turn execution. Each enemy executes this at its initiative:
```python
function executeEnemyTurn(enemy, battlefield):
  // Phase 1: TARGET SELECTION
  validTargets = getKnightsMatchingPriority(enemy.targetPriority)
  if validTargets.isEmpty(): validTargets = allLivingKnights()
  target = validTargets.sortBy(enemy.targetPriority).first()
  if tie: prefer knight with lowest current PS
  // Phase 2: MOVEMENT (before attack)
  canReachTarget = isInWeaponRange(enemy, target)
  switch(enemy.rankBehavior):
    ADVANCE: if NOT canReachTarget: moveToward(target, 1 rank)
             if canReachTarget: stay (do not overextend)
    HOLD:    never move. If no target in range, skip attack.
    RETREAT: if in Rank 1-2: move to Rank 3-4. Attack from range if possible.
    FLANK:   prefer targeting Rank 3-4 knights. Move to minimize rank gap to rear targets.
  // Phase 3: ATTACK
  canReachTarget = isInWeaponRange(enemy, target)  // re-check after move
  if canReachTarget:
    // [v8.9] Combat Actions = 1 + Actions Multiples count. Each can attack or be used as movement.
    for each combatAction (1 + enemy.actionsMultiples):
      executeAttack(enemy, target)  // per SPEC-18.9
      // Cadence X handling (SPEC-20.4)
      if weapon.hasCadence(X):
        // [v8.9] Extra targets must be on adjacent enemy ranks (within 1 rank of primary target)
        extraTargets = getEnemiesAdjacentToTarget(target, X, weaponRange)
        for each extraTarget: executeAttack(enemy, extraTarget, dicePenalty=-3)
  else:
    skip attack (wasted turn — HOLD enemy with no ranged weapon)
  // Phase 4: SPECIAL ABILITIES
  executeCapacities(enemy)  // Charge Brutale, Régénération, etc.
```
[v7] Tiebreaker Rules:
Multiple valid targets with same priority score: prefer lowest PS. Still tied: prefer closest rank. Still tied: random.
[v7] FLANK in 4-Rank System:
FLANK enemies target Rank 3-4 knights (backline). If no Rank 3-4 knights exist, target Rank 2, then Rank 1. FLANK enemies with contact weapons move toward rear ranks; with ranged weapons, they attack from current position.
### 18.12 Hybrid AI Pattern (RankBehavior Split)
Some enemies have a preferred combat style (melee or ranged) but can adapt situationally. This is modeled as a Hybrid RankBehavior that combines a spawn-time preference with per-turn situational adaptation. Generic pattern — reusable for any enemy with multiple weapon types.
Hybrid AI Schema: struct HybridRankBehavior { float meleeWeight; // 0.0–1.0. Bestian = 0.75. RankBehavior spawnPreference; // Rolled once at spawn: random(0,1) < meleeWeight ? ADVANCE : HOLD. Locked for this enemy instance. RankBehavior currentBehavior; // May be overridden per-turn by situational rules below. }
Per-turn situational override (applied in 18.11 Phase 2 MOVEMENT): (1) If spawnPreference = ADVANCE AND enemy is at Rank 1 AND contact target is in range: use ADVANCE (attack in melee). No override needed. (2) If spawnPreference = ADVANCE AND enemy is NOT at Rank 1: ADVANCE toward Rank 1 to close distance. (3) If spawnPreference = HOLD AND enemy has ranged weapon AND target is in ranged weapon range: HOLD (fire from current position). (4) OVERRIDE — If spawnPreference = HOLD BUT no target is within ranged weapon range AND enemy has contact weapon: temporarily switch to ADVANCE for this turn (close distance to attack). (5) OVERRIDE — If spawnPreference = ADVANCE BUT enemy is at Rank 3-4 with ranged weapon AND contact target is 2+ ranks away: temporarily switch to HOLD for this turn (fire ranged instead of wasting a turn moving). (6) If no weapon can reach any target from any behavior: skip attack (wasted turn).

Stat block notation: AI: RankBehavior field uses HYBRID_MELEE_XX or HYBRID_RANGED_XX where XX = melee/ranged weight as percentage (e.g. HYBRID_MELEE_75 = 75% melee preference). For non-hybrid enemies, use the existing HOLD/ADVANCE/RETREAT/FLANK enums as before.

### 18.13 Charge Brutale (Enemy Capacity)
Source: Knight Livre de Base, Bestiaire. Canonical enemy capacity used by Faune, Behemot, and Ours Corrompu.

**Mechanic:**
1. **Cooldown:** Once per combat for standard enemies. [v8.9] **Multi-phase bosses (generic rule):** Charge Brutale cooldown **resets** on any boss phase transition — the boss may Charge once per phase. This is a generic multi-phase boss rule, not specific to Ours Corrompu. Any future multi-phase boss with Charge Brutale benefits from this reset automatically.
2. **Range:** Enemy can instantly close distance to any target at **Moyenne range or less** (gap ≤ 4). Ignores normal movement restrictions — the enemy teleports to the target's adjacent rank.
3. **Attack roll:** Standard Combo Roll per SPEC-18.9 (enemy rolls Aspect dice, count evens, add Exceptionnels). Must hit (successes > target Defense).
4. **Damage on hit:** [v8.9] Pre-baked weapon damage + **2× Bête Aspect score** (flat bonus added on top of the weapon's stat block value). The weapon stat block already includes all normal Exceptionnel bonuses — Charge Brutale adds 2× Bête as an ADDITIONAL modifier, not a replacement. Do NOT strip out the weapon's pre-baked flat bonus before adding the Charge bonus. Formula: `chargeDamage = weaponDamageDice + weaponFlatBonus + (2 × enemy.bête)`.
5. **Resolution:** Routes through ArmourLayerResolver (SPEC-06) like any normal attack.

**Per-enemy Charge Brutale values:**

| Enemy | Bête | Contact Weapon | Charge Damage | Cooldown |
|---|---|---|---|---|
| Faune | 10 | 4D6+16 | 4D6+16 + 20 = **4D6+36** | Once per combat |
| Behemot | 16 | 3D6+22 | 3D6+22 + 32 = **3D6+54** | Once per combat |
| Ours Corrompu (P1) | 14 | 5D6+18 | 5D6+18 + 28 = **5D6+46** | Once per phase |

**AI trigger (18.11 Phase 4):** Enemy uses Charge Brutale when: (a) it has not yet used it this combat/phase, AND (b) the highest-priority target is at gap > 1 (not already in contact range). The charge replaces one Combat Action for the turn.

### 18.10 Peur (Fear) Mechanic
Enemies with Peur (X) force a fear check at encounter start. Source: Knight Livre de Base, Bestiaire.

**Trigger:** At the start of a combat encounter (phase de conflit), once per Peur source.
**Roll:** Peur test is a **standard Combo Roll** (SPEC-04). Base Characteristic = highest of (Sang-Froid, Hargne) for the tested knight. Player (or auto-gen) selects a Combo Characteristic per standard rules (any Characteristic except the chosen Base). OD auto-successes from both Base and Combo Characteristics apply. PEs penalty applies (SPEC-07 — all Combo Rolls are affected). Pool floor at 0 = automatic failure (SPEC-04). Opposition threshold = `floor(enemy.highestAspect / 2)`. Successes must **strictly exceed** the opposition (equal = failure).

| Outcome | Condition | Effect | Duration |
|---|---|---|---|
| Success | Successes > opposition | **−1 die** to all tests. | Entire encounter. |
| Failure | Successes ≤ opposition (but not all odd) | **−X dice** to all tests AND **−X Defense** AND **−X Reaction** (where X = Peur level). Minimum 0 for Defense/Reaction. | Entire encounter. |
| Échec Critique | All dice show odd results (0 successes) | **Paralyzed** for **XD6 turns** (no actions — skip combat and movement actions). X = Peur level. | XD6 turns (rolled once). |

**Stacking:** Each knight tests once per encounter per Peur source. Multiple Peur sources require separate checks but effects do NOT stack — apply the **worst result** only. A knight cannot suffer Peur effects multiple times per turn.

**Per-enemy Peur levels in demo:**

| Enemy | Peur Level | Opposition (highest Aspect / 2) |
|---|---|---|
| Behemot | Peur (1) | floor(16/2) = 8 |
| Ours Corrompu | Peur (2) | floor(14/2) = 7 |

### Acceptance Criteria
[v6] Enemy attack: Bestian (Bête 12, Bête Exc Mineur 3) attacks knight (Def 6). Rolls 12D6, gets 5 evens + 3 auto-successes = 8 total. 8 > 6: hit.
[v6] Enemy attack: Faune (Machine 10) fires ranged weapon at knight (React 4). Rolls 10D6, gets 3 evens. 3 < 4+1: miss (must exceed, not equal).
[v6] Enemy Failure Critique: all odd on 8D6. Attack auto-misses. Enemy is exposed (isExposed = true). Next knight attack vs this specific enemy: +2 auto-successes, then isExposed cleared. If E1 and E2 both Failure Critique in the same round: knight attacking E1 gets +2, knight attacking E2 gets +2 — per-enemy, not additive. Bonus expires at start of exposed enemy's next turn if not consumed.
Bande with Cohésion 200, Débordement 6: turn 3 deals 18 damage. Violence reduces Cohésion.



Salopard with Actions Multiples (1): 2 Combat Actions + 1 Movement Action. Executes 2 attacks at its initiative (or 1 attack + 1 combat-as-movement).  
[v8.9] Ours P2 with Actions Multiples (2): 3 Combat Actions + 1 Movement Action. Can attack 3 different knights in one turn, or trade combat actions for movement to close distance.    
[v8.9] Bête Exceptionnel pre-baked: Faune attacks with "4D6+16" — the +16 already includes +10 (Bête Majeur: Aspect score) + +6 (Exceptionnel score). Agent rolls 4D6, adds faces, adds 16. No further modifier needed.  
[v8.9] Chair Mineur: Knight hits Ours Corrompu (Chair Mineur 3) for 30 raw damage. ArmourLayerResolver Step 4: remaining = 30 − 3 = 27. Remaining routes to PA/PS.  
[v8.9] Chair Majeur: Knight attacks Behemot (Chair Majeur 5) with Shotgun (Meurtrier, Choc 1, Barrage 2). Chair Mineur: −5 flat dmg reduction. Chair Majeur: Meurtrier +2D6 NOT rolled, Choc NOT applied, Barrage NOT applied. Behemot is immune to all three effects.  
[v8.9] Masque Mineur: Faune has Masque Mineur (1). Defense stat block already includes this bonus. Agent reads Defense = 6 as-is. Do NOT add +1 at runtime.  



---

## SPEC-11: Bande Controller
### Purpose
Manages Bande entities. [v2] Completely rewritten. Bandes have NO attacks. Débordement only.
### [v2] Core Rules
Bande = single entity representing 3–12+ creatures.
Cohesion: Damaged by Violence only. Dégâts (weapon damage) are completely ignored against Bandes. Knights must use high-Violence weapons (Fusil d’Assaut, grenades) to reduce Cohesion effectively.
[v2] Débordement is the Bande's ONLY offensive mechanic. No attacks, no hit rolls.
Débordement: [v8.9] Automatic damage to ALL combatants of the **opposing faction** each turn (for an enemy Bande: all knights; **FUTURE:** for an allied Bande reinforcement: all enemy on-track combatants). Implement using faction-based targeting (`bande.faction != target.faction`) — do NOT hardcode "damages knights." Cumulative: turn N = N × débordementScore. [v8] Débordement damage is resolved through the standard Knight Armour Layer chain (SPEC-06). CdF is the knight's total (base + Shrine bonus); this total CdF is subject to all normal modifiers including IGNORE_CDF and Pénétrant X. Apply any Débordement Tags (e.g., IGNORE_CDF for Nocte) during resolution. PA absorbs normally. No attack roll — damage is automatic, bypassing hit determination. Clarification: Shrine (SPEC-17.2) adds +6 to the knight's CdF total. It is NOT a separate defensive layer — it is part of CdF. Therefore IGNORE_CDF bypasses both base CdF and Shrine bonus entirely. Pénétrant X reduces the combined total (base + Shrine). Agents must NOT implement Shrine as a separate shield; it modifies the CdF value that SPEC-06 Step 2 already handles. Worked Example — Nocte Débordement vs Shrine: Nocte bande, Turn 3, débordementScore 4. Damage = 3 × 4 = 12 per knight. Target: Paladin at Rank 2 with base CdF 8 + Shrine +6 = total CdF 14. Nocte Débordement carries IGNORE_CDF tag. Resolution: SPEC-06 Step 2 — IGNORE_CDF active, skip CdF step entirely. CdF 14 (including Shrine) provides zero protection. Full 12 damage → PA absorbs 12. PA: 120 → 108. Passthrough: floor(12/5) = 2 PS lost. Shrine is useless against Nocte. This is the intended threat model — Nocte bandes pressure through Shrine, forcing Violence-based weapons to reduce Cohesion rather than relying on CdF tanking.
Initiative: Bandes always act last (initiative 1).

**One Bande Rule:** Only one Bande entity can exist per encounter at any time. If a second Bande would spawn (e.g., boss Phase 2 transition, reinforcement wave):
- **Existing Bande still alive:** Add the new Bande's Cohesion to the existing Bande via `AddCohesion(newCohesion)`. The existing Bande's `débordementTurn` counter is **not** reset — pressure continues accumulating. `débordementScore` remains unchanged (use the existing Bande's score).
- **Existing Bande destroyed:** Spawn a fresh Bande entity with the new Cohesion, `débordementScore` from the Bande's spawn definition (e.g., Nocte = 4, see SPEC-21.1), and `débordementTurn = 1`.
Do not create a second `BandeController` instance. The Bande occupies no specific rank — it is off-track (see Targeting Rules below).

### [v2] Targeting Rules
Bandes do not target. Débordement hits all opposing-faction combatants automatically. No position restriction.
Débordement ignores stealth. Ghost Mode does not protect against Débordement.
Knights target Bande from any position using Courte+ range. No gap check.
Smart Grenades deal Violence to Cohesion from any position.
### [v8] Knight vs Bande Attack Procedure
Knights attack the Bande using a standard Combo Roll (SPEC-04). Compare total successes vs the Bande's Defense (contact attacks) or Reaction (ranged attacks). [v8.9] Bande Defense/Reaction are fixed values from the SPEC-21 stat block (e.g., Nocte: Defense 5, Reaction 1) — they are NOT derived from aspects. On hit: apply the weapon's Violence dice to Cohesion. Weapon damage (Dégâts) is completely ignored — only Violence reduces Cohesion. On miss: no effect. Exploit and Failure Critique rules apply normally. Excess successes with Assistance à l'Attaque convert to +1 Violence (not damage). Range: Courte+ from any position, no gap check (Bandes are off-track, see core rules above).

### Data Model
```csharp
class BandeController {
  int cohesion;
  int debordementScore;
  int debordementTurn; // starts at 1, increments AFTER dealing damage at Bande's initiative
  bool isActive;

  // Called at the Bande's initiative (always last in round, initiative 1).
  // Sequence: (1) deal damage, (2) increment counter.
  int GetDebordementDamage() => debordementTurn * debordementScore;

  void ExecuteBandeTurn() {
    int damage = GetDebordementDamage(); // e.g., turn 1: 1×4=4, turn 2: 2×4=8
    ApplyDebordementToAllOpposingFaction(damage); // SPEC-06 per target
    debordementTurn++; // increment AFTER damage is dealt
  }

  void TakeDamage(int rawViolence) { // [v8] Only Violence reduces Cohesion. Dégâts are ignored.
    cohesion -= rawViolence;
    // [v8] Dégâts do not affect Bandes — only Violence reduces Cohesion
    if (cohesion <= 0) Destroy();
  }
  void AddCohesion(int amount) { cohesion += amount; }
}
```


### Acceptance Criteria
Cohesion 30, Débordement 3: Turn 1=3 to all, Turn 2=6, Turn 3=9.

Second Bande spawn: first Bande’s Cohesion increases.
[v2] Rogue in Ghost still takes Débordement.
[v2] No Bande attack actions exist. No hit rolls. Débordement only.


---

## SPEC-14: Boss Phase Controller
### Purpose
Monitors Patron PS thresholds, triggers phase transitions. In the demo, only the **Ours Corrompu** has phases. Full stat blocks and capacities are defined in SPEC-21.5 ([08_ContentData.md](./08_ContentData.md)) — that file is canonical for stat values. This file (SPEC-14) is canonical for the transition procedure and timing.

### Phase Transition Procedure
The Ours Corrompu has 2 phases. Phase transition is triggered when Phase 1 PS reaches 0.

**Transition timing:** Phase transition is immediate when Phase 1 PS reaches 0. The current attacker's turn (the knight who dealt the killing blow) resolves completely — all remaining actions, effects, and damage on that turn finish normally. The boss does NOT enter Agony — phase transition overrides death. The boss's next turn (at its normal initiative slot in the initiative queue) uses Phase 2 stats. If the boss already acted this round before being reduced to 0 PS, Phase 2 does NOT grant an additional turn — the boss must wait for the next round. The boss retains its original initiative value (unchanged by phase transition).

**Phase 1 → Phase 2 Transition (step-by-step):**
1. **Trigger:** Phase 1 PS reaches 0. The boss does NOT enter Agony — phase transition overrides death.
2. **Stat changes:** Apply Phase 2 stat overrides from [SPEC-21.5](./08_ContentData.md) (canonical for all stat values). See that file for the full Phase 2 stat block. Summary of key changes: PS resets, Bouclier increases, Défense increases, Actions Multiples increases. Do NOT duplicate stat values here — SPEC-21.5 is the single source of truth.
3. **New weapon unlocked:** Ombre dévorante — see [SPEC-21.5](./08_ContentData.md) for complete weapon stats. As an Anathème attack, damage after CdF bypasses PA and hits PEs directly (SPEC-06 §Knight Defense vs Anathème). Added to weapon pool alongside existing weapons.
4. **New capacity unlocked:** Régénération — if Nocte bande is present, boss consumes 50 Cohésion to heal 25 PS (once per turn, at start of boss turn). Requires Cohésion > 50 (not just ≥ 50 — safeguard prevents killing bande). Does not partially consume. [v8.9] **Safeguard:** Régénération does NOT fire if consuming 50 Cohésion would reduce the bande's Cohésion to exactly 0 (i.e., requires Cohésion > 50, not just ≥ 50). The boss will not sacrifice the last of its bande — at Cohésion = 50, the bande is too weak to drain. This prevents the boss from accidentally destroying its own support. At Cohésion 51+: Régénération fires normally (leaving at least 1 Cohésion). **Timing:** Régénération fires at the start of the boss's turn (initiative 6). The Nocte bande acts at initiative 1 (last). This means Cohésion is consumed by Régénération BEFORE the bande's Débordement turn — the boss effectively weakens the bande before it deals damage.
5. **Bande reinforcement:** Per SPEC-22 ([10_MissionAndHub.md](./10_MissionAndHub.md)):
- If Nocte bande is still alive: `AddCohesion(200)` — the bande receives reinforcements.
- If Nocte bande was destroyed: Fresh spawn (200 Cohésion, débordementScore = 4 per SPEC-21.1, débordementTurn = 1).
6. **AI change:** Phase 2 AI shifts to LOWEST_PEs / LOWEST_PS, ADVANCE, Aggression 3 (up from HIGHEST_BETE / HIGHEST_COMBAT, ADVANCE, Aggression 2). The boss now hunts the most psychologically fragile knight to trigger Despair cascades via Ombre dévorante's Anathème damage.
7. **Narrative event:** "Darkness erupts. Arena goes dark." Fire `OnPhaseTransition(boss, 2)` event per SPEC-27. Lumière weapons become critical.
8. **PEs effect:** All knights lose 1D6 PEs (dread from darkness erupting). Fire `OnEspoirLost(knight, roll, "phase_transition")` for each knight.

### Acceptance Criteria
Phase 1 PS=0: Phase 2 triggers. PS resets to 120. Bouclier 12, Défense 10.
Phase 2: Boss gets 3 combat actions per turn (Actions Multiples 2). Single initiative slot.
Phase 2 AI: shifts to LOWEST_PEs / LOWEST_PS, ADVANCE, Aggression 3. Boss targets the knight with lowest PEs to trigger Despair via Ombre dévorante.
Phase 2 Ombre dévorante: Anathème attack. Attack Aspect: Bête (14). On hit: CdF reduces damage, remainder hits PEs directly (PA bypassed per SPEC-06 §Knight Defense vs Anathème).
Phase 2 Bande alive: AddCohesion(200). Bande destroyed: fresh spawn (200 Cohésion, Déb 4).
Phase 2 Régénération: Boss at start of turn consumes 50 Cohésion → heals 25 PS. Requires Cohésion > 50 (not ≥ 50 — safeguard prevents killing bande). At Cohésion 50: does not fire. At Cohésion 51: fires, bande left at 1 Cohésion. Blocked if no bande exists.
Phase 2 Darkness erupts: all knights −1D6 PEs.
Phase 2 PS=0: Boss dies. Encounter ends. Fire OnBossDefeated event.
