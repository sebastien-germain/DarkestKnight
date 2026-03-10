# Constants & Enumerations Reference
> **Purpose:** Consolidated reference for all enums, constants, and magic values used across the specification. Each entry lists the canonical source SPEC. Agents should reference this file when building the data layer to avoid hunting across 13+ files.
> **Dependencies:** None (read-only reference)
> **Depended on by:** ALL systems
---

## 1. Identity & Aspect Enums

### AspectId
**Source:** SPEC-01 ([01_KnightDataModel.md](./01_KnightDataModel.md))
```csharp
enum AspectId { CHAIR, BETE, MACHINE, DAME, MASQUE }
```

### CharacteristicId
**Source:** SPEC-01 ([01_KnightDataModel.md](./01_KnightDataModel.md))
```csharp
enum CharacteristicId {
  // Chair (Body)
  DEPLACEMENT, FORCE, ENDURANCE,
  // Bête (Beast)
  COMBAT, INSTINCT, HARGNE,
  // Machine
  TIR, SAVOIR, TECHNIQUE,
  // Dame (Lady)
  AURA, PAROLE, SANG_FROID,
  // Masque (Mask)
  DISCRETION, DEXTERITE, PERCEPTION
}
```
**Mapping:** Each Aspect contains exactly 3 Characteristics. Total: 15.

| Aspect | Characteristics |
|--------|----------------|
| Chair | Déplacement, Force, Endurance |
| Bête | Combat, Instinct, Hargne |
| Machine | Tir, Savoir, Technique |
| Dame | Aura, Parole, Sang-Froid |
| Masque | Discrétion, Dextérité, Perception |

### ArmorClass
**Source:** SPEC-02.7 ([01_KnightDataModel.md](./01_KnightDataModel.md))
```csharp
enum ArmorClass { WARRIOR, PALADIN, PRIEST, ROGUE }
// FUTURE: RANGER (post-demo, weights pre-defined in SPEC-02 table)
```

---

## 2. Combat Enums

### CombatStyle
**Source:** SPEC-04 ([03_CombatRolls.md](./03_CombatRolls.md))
```csharp
enum CombatStyle {
  STANDARD,       // No modifier
  AGRESSIF,       // +3 attack dice, −2 Def, −2 React
  DEFENSIF,       // −3 attack dice, +2 Defense only
  MISE_A_COUVERT, // −3 attack dice, +2 Reaction only (requires cover at rank)
  PUISSANT,       // −N attack dice (1–6), −2 Def/React. Requires Lourd weapon.
  PILONNAGE,      // −2 attack dice, accumulating bonus. Requires Deux Mains ranged.
  PRECIS,         // +3rd Characteristic. Costs Combat + Movement Action.
  AMBIDEXTRE,     // Dual-wield: 2 separate attacks, −3 dice (or −1 with Jumelé)
  AKIMBO          // Dual-wield: 1 combined attack, −3 dice (or −1 with Jumelé)
}
```

### NodType
**Source:** SPEC-33 ([13_NodSystem.md](./13_NodSystem.md))
```csharp
enum NodType { SOIN, ARMURE, ENERGIE }
// SOIN = restores PS (3D6). ARMURE = restores PA (3D6). ENERGIE = restores PE (3D6).
```

### MotivationType
**Source:** SPEC-12.1 ([08_ContentData.md](./08_ContentData.md))
```csharp
enum MotivationType { MINOR, MAJOR, VOEU }
// MINOR: repeatable, +1D6 PEs per trigger. 2 drawn from template pool + 1 Blason Voeu (+ optional 4th from Tarot).
// MAJOR: once per mission, +25 PEs current and max on fulfillment. 1 drawn from major pool.
// VOEU: Blason-derived minor motivation. Mechanically identical to MINOR but sourced from Blason, not template pool.
```

---

## 3. Weapon Enums

### WeaponCategory
**Source:** SPEC-19 ([07_WeaponsAndEffects.md](./07_WeaponsAndEffects.md))
```csharp
enum WeaponCategory { STANDARD, ADVANCED, RARE }
```

### WeaponSlot
**Source:** SPEC-19 ([07_WeaponsAndEffects.md](./07_WeaponsAndEffects.md))
```csharp
enum WeaponSlot { CONTACT, RANGED, DUAL_PROFILE }
```

### ProfileType
**Source:** SPEC-19 ([07_WeaponsAndEffects.md](./07_WeaponsAndEffects.md))
```csharp
enum ProfileType { CONTACT, RANGED }
```

### ProfileSwitchCost
**Source:** SPEC-04/19 ([03_CombatRolls.md](./03_CombatRolls.md), [07_WeaponsAndEffects.md](./07_WeaponsAndEffects.md))
```csharp
enum ProfileSwitchCost { FREE, MOVEMENT_ACTION }
// FREE: weapon adapts to action type (e.g., Marteau-Épieu, Pistolet de Service, Grenades Intelligentes)
// MOVEMENT_ACTION: manual manipulation required (e.g., Lance-Grenade ammo swap, future Arbalète Magnétique bolt swap, future Épée Bâtarde 1H/2H swap)
```

### WeaponRange
**Source:** SPEC-19 ([07_WeaponsAndEffects.md](./07_WeaponsAndEffects.md))
```csharp
enum WeaponRange { CONTACT, COURTE, MOYENNE, LONGUE, LOINTAINE }
```

**Range to Max Gap mapping (SPEC-03):**

| Range | Max Gap | Distance |
|-------|---------|----------|
| CONTACT | 1 | 0–2m |
| COURTE | 2 | 2–15m |
| MOYENNE | 4 | 15–50m |
| LONGUE | 5 | 50–300m |
| LOINTAINE | 6 | 300m+ (effectively unlimited on 4-rank track — max possible gap is 6) |

### ForceMode
**Source:** SPEC-19 ([07_WeaponsAndEffects.md](./07_WeaponsAndEffects.md))
```csharp
enum ForceMode { NONE, NORMAL, LESTE }
// NONE: no Force added. NORMAL: +Force×1. LESTE: +Force×2.
```

### WeaponEffectId
**Source:** SPEC-20 ([07_WeaponsAndEffects.md](./07_WeaponsAndEffects.md))
```csharp
enum WeaponEffectId {
  // Damage Modifiers (§20.1)
  ASSISTANCE_ATTAQUE, DEGATS_CONTINUS, DESTRUCTEUR, EN_CHAINE,
  FUREUR, LESTE, MEURTRIER, ORFEVRERIE, PRECISION, ULTRAVIOLENCE,
  // Defense Bypass (§20.2)
  ANTI_ANATHEME, ANTI_VEHICULE, IGNORE_ARMURE, IGNORE_CDF, PENETRANT, PERCE_ARMURE,
  // Crowd Control (§20.3)
  BARRAGE, CHOC, DEMORALISANT, DISPERSION, LUMIERE, PARASITAGE,
  // Targeting & Utility (§20.4)
  ARTILLERIE, ASSASSIN, CADENCE, CHARGEUR, DESIGNATION,
  DEUX_MAINS, JUMELE_AKIMBO, JUMELE_AMBIDEXTRIE, LOURD, SILENCIEUX, TIR_EN_SECURITE
}
```
Parameterized effects use `_X` suffix at runtime (e.g., `BARRAGE_2`, `DISPERSION_5`, `PERCE_ARMURE_40`). The X value is stored in the DamageProfile, not the enum.

[v8.9] **Note:** Weapon effect tags used in ArmourLayerResolver pseudocode (e.g., `weapon.hasTag(IGNORE_CDF)`, `weapon.hasTag(PENETRANT_X)`) map directly to `WeaponEffectId` values. No separate `DamageTag` enum is needed — use `WeaponEffectId` for both weapon data and damage resolution checks.

---

## 4. Enemy Enums

### SeigneurId
**Source:** SPEC-18 ([06_EnemySystem.md](./06_EnemySystem.md))
```csharp
enum SeigneurId { LA_BETE, LA_MACHINE, LA_CHAIR, LA_DAME, LE_MASQUE }
// Each enemy belongs to one Seigneur faction. Determines thematic behavior and resistances.
// Maps to EnemyBase.seigneur field.
```

### EnemyTier
**Source:** SPEC-18 ([06_EnemySystem.md](./06_EnemySystem.md))
```csharp
enum EnemyTier {
  T1_BANDE,     // Swarm/mob. Cohesion pool. Off-track.
  T2_HOSTILE,   // Named grunt. PS only.
  T3_SALOPARD,  // Elite fighter. PS + PA optional.
  T4_COLOSSE,   // Huge tank. PS + PA + 10:1 damage rule.
  T5_PATRON     // Boss. PS + PA + Bouclier + Point Faible + Phases.
}
```

### TargetPriority
**Source:** SPEC-18.7 ([06_EnemySystem.md](./06_EnemySystem.md))
```csharp
enum TargetPriority {
  HIGHEST_COMBAT, HIGHEST_BETE, HIGHEST_DAME, HIGHEST_AURA,
  LOWEST_PS, LOWEST_PA, LOWEST_PEs, NEAREST_RANK, RANDOM
}
```

### RankBehavior
**Source:** SPEC-18.7 / 18.12 ([06_EnemySystem.md](./06_EnemySystem.md))
```csharp
enum RankBehavior {
  HOLD,              // Stay in current rank
  ADVANCE,           // Move toward Rank 1 (front)
  RETREAT,           // Move toward Rank 4 (back)
  FLANK,             // Target rear-rank knights
  HYBRID_MELEE_75,   // 75% melee spawn preference (Bestian)
  HYBRID_RANGED_75   // 75% ranged spawn preference
  // Pattern: HYBRID_MELEE_XX / HYBRID_RANGED_XX where XX = preference %
}
```

---

## 5. Architecture Enums

### LifeState / ControlState / ActionState / ArmorState
**Source:** SPEC-30 ([12_ArchitectureGuardrails.md](./12_ArchitectureGuardrails.md))
```csharp
enum LifeState { Alive, Dead }
enum ControlState { PlayerControlled, EnemyControlled }
enum ActionState { CanAct, Incapacitated }
enum ArmorState { Normal, Folded }
```

### RNGPolicy
**Source:** SPEC-28 ([12_ArchitectureGuardrails.md](./12_ArchitectureGuardrails.md))
```csharp
enum RNGPolicy { DeterministicNodeSeed, FullRNGOnRetry }
```

### RngStreamId
**Source:** SPEC-28 ([12_ArchitectureGuardrails.md](./12_ArchitectureGuardrails.md))
```csharp
enum RngStreamId { Combat, Loot, Cosmetic }
```

### EncounterType
**Source:** SPEC-29 ([12_ArchitectureGuardrails.md](./12_ArchitectureGuardrails.md))
```csharp
enum EncounterType { Combat, Boss, Event, Rest }
// Demo uses: Combat + Boss only.
```

### TriggerEvent / TriggerActionType
**Source:** SPEC-29 ([12_ArchitectureGuardrails.md](./12_ArchitectureGuardrails.md))
```csharp
enum TriggerEvent { OnCombatStart, OnTurnStart, OnEnemyDeath, OnBossPhaseChanged }
enum TriggerActionType { SpawnReinforcement, ApplyEffect, ChangeCover, SwapAiProfile }
```

---

## 6. Generation Enums

### ConditionType
**Source:** SPEC-02.8 ([01_KnightDataModel.md](./01_KnightDataModel.md))
```csharp
enum ConditionType { ASPECT, EITHER_CHAR }
// ASPECT: check Aspect score itself (e.g., knight.chair >= 4)
// EITHER_CHAR: check either condChar1 or condChar2 >= condMinScore
```

---

## 7. Global Constants

| Constant | Value | Source | Notes |
|----------|-------|--------|-------|
| Max Aspects | 9 (normal), 7 (Vétéran) | SPEC-01, SPEC-24 | Warrior Type can push OD to 6 |
| Max Characteristics | ≤ parent Aspect | SPEC-01 | Hard cap |
| Max OD | 5 (normal), 6 (Warrior Type only) | SPEC-01, SPEC-17.1 | |
| Base Aspects | 2 | SPEC-02 | At generation start |
| Base Characteristics | 1 | SPEC-02 | At generation start |
| Knights generated | 8 | SPEC-02 | 2 per armor class |
| Knights per mission | 4 | SPEC-02/03 | Selected from roster |
| Ranks per side | 4 | SPEC-03 | Max 4 combatants per side |
| Max on-track enemies | 4 | SPEC-03/29 | Reinforcements queued until slot free |
| Heroism max | 6 | SPEC-09 | Chevalier Véritable triggers at 4 |
| Base PEs | 50 | SPEC-01/07 | + Tarot mods, − implants×3 |
| PEs penalty threshold | 10 | SPEC-07 | `penalty = max(0, 10 − currentPEs)` |
| Despair threshold | 0 PEs | SPEC-07 | |
| PS passthrough ratio | 1 per 5 PA dmg | SPEC-06 | `floor(paAbsorbed / 5)` |
| Colosse damage tranche | 10:1 | SPEC-18.5 | 10 raw = 1 effective |
| Grenades per mission | 5 | SPEC-19.4 | Per knight. Auto-refill at Camelot |
| Nods per mission | 3 of each type | SPEC-33 | Per knight. Auto-refill at Camelot |
| Tarot cards drawn | 5 from 22 | SPEC-24 | Select 2 advantages + 1 disadvantage |
| Max assistants | 3 | SPEC-04 | Per Combo Roll |
| Implant max | 6 | SPEC-01 | Each reduces maxPEs by 3 |
| Hémorragie countdown | 3 → 0 | SPEC-08 | Ticks at start of bleeding knight's turn |
| Bande initiative | 1 (always last) | SPEC-11 | |
| Math rounding | Floor (round down) | SPEC-03/INDEX | All divisions |
| Math operation order | Division/Multiplication before Addition/Subtraction | SPEC-03/INDEX | Knight tabletop rule |
| Defense/Reaction floor | 0 (minimum) | INDEX | After all modifiers. Negative → clamped to 0 |
| Hit threshold | Strictly exceed | SPEC-04/INDEX | Equal = miss |
| Duration "1 turn" | Until start of activator's next turn | SPEC-03/INDEX | |
| Successive doubling | ×3 (not ×4) | SPEC-03/INDEX | |

---

*End of constants reference. All values are derived from the canonical SPECs listed. If a value changes in its source SPEC, update this file accordingly.*

