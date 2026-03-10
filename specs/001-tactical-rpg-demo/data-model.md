# Data Model: Knight Into the Darkness

> **Version:** 1.0 -- Maps TechSpec SPEC files to Unity ScriptableObject schemas and runtime C# classes.
> **Source:** `/TechSpec/specifications/` (SPEC-01 through SPEC-33)
> **Target:** `Scripts/Data`, `Scripts/Combat`, `Scripts/Core`, `Assets/Data`

---

## Table of Contents

1. [Enumerations and Constants](#1-enumerations-and-constants)
2. [ScriptableObject Schemas (Authoring Data)](#2-scriptableobject-schemas-authoring-data)
   - [KnightBaseSO](#21-knightbaseso-spec-01)
   - [ArchetypeDataSO](#22-archetypedataso-spec-029)
   - [HautFaitDataSO](#23-hautfaitdataso-spec-028)
   - [BlasonDataSO](#24-blasondataso-spec-025)
   - [TarotCardSO](#25-tarotcardso-spec-24)
   - [WeaponProfileSO](#26-weaponprofileso-spec-19)
   - [EnemyBaseSO](#27-enemybaseso-spec-18)
   - [EncounterSO](#28-encounterso-spec-29)
   - [InjuryTableSO](#29-injurytableso-spec-23)
   - [MotivationTemplateSO](#210-motivationtemplateso-spec-121)
   - [MissionDefinitionSO](#211-missiondefinitionso-spec-22)
3. [Runtime State Classes](#3-runtime-state-classes-c)
   - [RunState](#31-runstate-spec-32)
   - [CombatSession](#32-combatsession-spec-32)
   - [CombatantState](#33-combatantstate-spec-30)
   - [NodInventory](#34-nodinventory-spec-33)
4. [Derived Value Formulas](#4-derived-value-formulas)
5. [Validation Rules](#5-validation-rules)
6. [State Transitions](#6-state-transitions)
7. [Entity Relationship Map](#7-entity-relationship-map)

---

## 1. Enumerations and Constants

Source: SPEC-14 (`14_Constants.md`)

### Identity and Aspect Enums

```csharp
enum AspectId { CHAIR, BETE, MACHINE, DAME, MASQUE }

enum CharacteristicId {
    // Chair (Body)
    DEPLACEMENT, FORCE, ENDURANCE,
    // Bete (Beast)
    COMBAT, INSTINCT, HARGNE,
    // Machine
    TIR, SAVOIR, TECHNIQUE,
    // Dame (Lady)
    AURA, PAROLE, SANG_FROID,
    // Masque (Mask)
    DISCRETION, DEXTERITE, PERCEPTION
}

enum ArmorClass { WARRIOR, PALADIN, PRIEST, ROGUE }
// FUTURE: RANGER (post-demo, weights pre-defined in SPEC-02 table)
```

### Aspect-to-Characteristic Mapping

| Aspect  | Characteristics                        |
|---------|----------------------------------------|
| Chair   | Deplacement, Force, Endurance          |
| Bete    | Combat, Instinct, Hargne              |
| Machine | Tir, Savoir, Technique                |
| Dame    | Aura, Parole, Sang-Froid              |
| Masque  | Discretion, Dexterite, Perception     |

### Combat Enums

```csharp
enum CombatStyle {
    STANDARD,        // No modifier
    AGRESSIF,        // +3 attack dice, -2 Def, -2 React
    DEFENSIF,        // -3 attack dice, +2 Defense only
    MISE_A_COUVERT,  // -3 attack dice, +2 Reaction only (requires cover)
    PUISSANT,        // -N attack dice (1-6), -2 Def/React. Requires Lourd.
    PILONNAGE,       // -2 attack dice, accumulating bonus. Requires Deux Mains ranged.
    PRECIS,          // +3rd Characteristic. Costs Combat + Movement Action.
    AMBIDEXTRE,      // Dual-wield: 2 separate attacks, -3 dice (or -1 with Jumele)
    AKIMBO           // Dual-wield: 1 combined attack, -3 dice (or -1 with Jumele)
}

enum NodType { SOIN, ARMURE, ENERGIE }

enum MotivationType { MINOR, MAJOR, VOEU }
```

### Weapon Enums

```csharp
enum WeaponCategory { STANDARD, ADVANCED, RARE }
enum WeaponSlot     { CONTACT, RANGED, DUAL_PROFILE }
enum ProfileType    { CONTACT, RANGED }
enum ProfileSwitchCost { FREE, MOVEMENT_ACTION }
enum WeaponRange    { CONTACT, COURTE, MOYENNE, LONGUE, LOINTAINE }
enum ForceMode      { NONE, NORMAL, LESTE }

enum WeaponEffectId {
    // Damage Modifiers
    ASSISTANCE_ATTAQUE, DEGATS_CONTINUS, DESTRUCTEUR, EN_CHAINE,
    FUREUR, LESTE, MEURTRIER, ORFEVRERIE, PRECISION, ULTRAVIOLENCE,
    // Defense Bypass
    ANTI_ANATHEME, ANTI_VEHICULE, IGNORE_ARMURE, IGNORE_CDF,
    PENETRANT, PERCE_ARMURE,
    // Crowd Control
    BARRAGE, CHOC, DEMORALISANT, DISPERSION, LUMIERE, PARASITAGE,
    // Targeting and Utility
    ARTILLERIE, ASSASSIN, CADENCE, CHARGEUR, DESIGNATION,
    DEUX_MAINS, JUMELE_AKIMBO, JUMELE_AMBIDEXTRIE, LOURD,
    SILENCIEUX, TIR_EN_SECURITE
}
```

### Enemy Enums

```csharp
enum SeigneurId  { LA_BETE, LA_MACHINE, LA_CHAIR, LA_DAME, LE_MASQUE }

enum EnemyTier {
    T1_BANDE,      // Swarm/mob. Cohesion pool. Off-track.
    T2_HOSTILE,    // Named grunt. PS only.
    T3_SALOPARD,   // Elite fighter. PS + PA optional.
    T4_COLOSSE,    // Huge tank. PS + PA + 10:1 damage rule.
    T5_PATRON      // Boss. PS + PA + Bouclier + Point Faible + Phases.
}

enum TargetPriority {
    HIGHEST_COMBAT, HIGHEST_BETE, HIGHEST_DAME, HIGHEST_AURA,
    LOWEST_PS, LOWEST_PA, LOWEST_PEs, NEAREST_RANK, RANDOM
}

enum RankBehavior {
    HOLD, ADVANCE, RETREAT, FLANK,
    HYBRID_MELEE_75, HYBRID_RANGED_75
}
```

### Architecture Enums

```csharp
enum LifeState    { Alive, Dead }
enum ControlState { PlayerControlled, EnemyControlled }
enum ActionState  { CanAct, Incapacitated }
enum ArmorState   { Normal, Folded }

enum RNGPolicy    { DeterministicNodeSeed, FullRNGOnRetry }
enum RngStreamId  { Combat, Loot, Cosmetic }

enum EncounterType     { Combat, Boss, Event, Rest }
enum TriggerEvent      { OnCombatStart, OnTurnStart, OnEnemyDeath, OnBossPhaseChanged }
enum TriggerActionType { SpawnReinforcement, ApplyEffect, ChangeCover, SwapAiProfile }

enum ConditionType { ASPECT, EITHER_CHAR }
```

### Global Constants

| Constant                   | Value              | Source        |
|----------------------------|--------------------|---------------|
| Max Aspects                | 9 (normal), 7 (Veteran) | SPEC-01/24 |
| Max OD                     | 5 (normal), 6 (Warrior Type only) | SPEC-01/17 |
| Base Aspects at generation | 2                  | SPEC-02       |
| Base Characteristics       | 1                  | SPEC-02       |
| Knights generated          | 8 (2 per class)   | SPEC-02       |
| Knights per mission        | 4                  | SPEC-03       |
| Ranks per side             | 4                  | SPEC-03       |
| Max on-track enemies       | 4                  | SPEC-03/29    |
| Heroism max                | 6                  | SPEC-09       |
| Base PEs                   | 50                 | SPEC-01/07    |
| PEs penalty threshold      | 10                 | SPEC-07       |
| Despair threshold          | 0 PEs              | SPEC-07       |
| PS passthrough ratio       | 1 per 5 PA damage  | SPEC-06       |
| Colosse damage tranche     | 10 raw = 1 effective | SPEC-18.5   |
| Grenades per mission       | 5 per knight       | SPEC-19.4     |
| Nods per mission           | 3 of each type     | SPEC-33       |
| Tarot cards drawn          | 5 from 22          | SPEC-24       |
| Max assistants per roll    | 3                  | SPEC-04       |
| Implant max per knight     | 6                  | SPEC-01       |
| Hemorragie countdown       | 3 to 0             | SPEC-08       |
| Bande initiative           | 1 (always last)    | SPEC-11       |
| Hit threshold              | Strictly exceed     | SPEC-04       |
| Math rounding              | Floor (round down)  | Global        |

### Range-to-Gap Mapping

| Range     | Max Gap | Distance  |
|-----------|---------|-----------|
| CONTACT   | 1       | 0--2m     |
| COURTE    | 2       | 2--15m    |
| MOYENNE   | 4       | 15--50m   |
| LONGUE    | 5       | 50--300m  |
| LOINTAINE | 6       | 300m+     |

---

## 2. ScriptableObject Schemas (Authoring Data)

All authoring data is stored as Unity ScriptableObjects under `Scripts/Data` (definitions) and `Assets/Data` (instances). These are immutable at runtime -- combat systems read but never write to SOs.

---

### 2.1 KnightBaseSO (SPEC-01)

**Path:** `Scripts/Data/KnightBaseSO.cs`
**Instances:** 8 generated at campaign start (2 per armor class)
**Purpose:** Template and runtime data structure for a knight. All other systems read/write this model.

```csharp
class KnightBaseSO : ScriptableObject {

    // --- Identity (immutable after generation) ---
    string knightId;             // Unique, generated
    string displayName;          // Display name
    string callsign;             // From HautFait surnoms list

    // --- Generation Data (immutable) ---
    ArchetypeDataSO archetype;   // Reference to archetype SO
    TarotCardSO[] drawnCards;    // 5 cards drawn at generation
    TarotCardSO[] activeAdvantages;  // 2 selected from drawn
    TarotCardSO activeDisadvantage;  // 1 selected from drawn
    HautFaitDataSO hautFait;     // Reference to haut fait SO
    BlasonDataSO blason;         // Reference to blason SO

    // --- Motivations ---
    Motivation[] minorMotivations;  // 3-4 entries: 2 drawn + 1 Voeu + optional Tarot
    Motivation majorMotivation;     // 1 drawn, nullable (null if Sacrifice Total)

    // --- Aspects (mutable only at Camelot) ---
    int chair;    // 0-9 (0-7 if Veteran)
    int bete;     // 0-9
    int machine;  // 0-9
    int dame;     // 0-9
    int masque;   // 0-9

    // --- Characteristics (mutable by injuries, training) ---
    int[15] characteristics;     // Indexed by CharacteristicId enum
    // Access: characteristics[(int)CharacteristicId.COMBAT]

    // --- Meta-Armor (immutable after assignment) ---
    ArmorClass armorClass;       // Warrior/Paladin/Priest/Rogue
    int maxPA;                   // From SPEC-02.7 table
    int maxPE;                   // From SPEC-02.7 table
    int baseCdF;                 // From SPEC-02.7 table
    int[15] odLevels;           // OD per characteristic. Cap: 5 (6 with Warrior Type)
    SlotCounts slotCounts;       // head/torso/armL/armR/legL/legR

    // --- Runtime Resources (mutable) ---
    int currentPS;
    int maxPS;                   // Derived: 10 + 6 * Max(Force, Endurance, Deplacement)
    int currentPA;
    int currentPE;
    int currentPEs;
    int maxPEs;                  // 50 + tarotMods - (implantCount * 3)
    int heroisme;                // 0-6
    int currentCdF;

    // --- Combat State (CombatSession-only in practice) ---
    int position;                // 1-4 (rank)
    CombatStyle activeStyle;
    bool isInAgony;
    bool isInFoldState;
    bool isDespair;
    bool isGhostActive;         // Rogue Ghost state
    bool hasHemorragie;
    int hemorragieCountdown;     // 3->2->1->0 (dead)
    int activeWarriorTypeIndex;  // Nullable, Warrior only
    List<InjuryResult> activeInjuries;
    List<StatusEffect> activeStatusEffects;

    // --- Equipment ---
    int implantCount;            // 0-6, each reduces maxPEs by 3
    WeaponProfileSO[] equippedWeapons;
    int activeWeaponIndex;
    int activeProfileIndex;
    NodInventory nods;           // 3 each of SOIN, ARMURE, ENERGIE per mission
    bool isDead;
}
```

**Supporting structs:**

```csharp
struct SlotCounts {
    int head, torso, armL, armR, legL, legR;
}

struct Motivation {
    string motivationId;
    MotivationType type;         // MINOR, MAJOR, VOEU
    string displayText;          // French description
    string eventTag;             // Matched by SPEC-12 event system
}

struct InjuryResult {
    int severityDie;             // Column 1-4
    int rowDie;                  // Row 1-6
    string injuryId;
    bool isPermanent;            // Hemorragie: isPermanent = false
}
```

---

### 2.2 ArchetypeDataSO (SPEC-02.9)

**Path:** `Scripts/Data/ArchetypeDataSO.cs`
**Instances:** 17
**Purpose:** Personality/background type providing +1 to exactly two characteristics.

```csharp
class ArchetypeDataSO : ScriptableObject {
    string archetypeId;          // e.g., "ARCHETYPE_REBUT"
    string displayName;          // French display name
    CharacteristicId bonus1;     // Fixed, OR null if choiceBonusSlot == 1
    CharacteristicId bonus2;     // Fixed, OR null if choiceBonusSlot == 2
    CharacteristicId[] choicePool; // Nullable. For choice archetypes (#6,#7,#8)
    int choiceBonusSlot;         // 0=no choice, 1=bonus1 from pool, 2=bonus2 from pool
}
```

**Instance table (17 entries):**

| # | Archetype | Bonus 1 | Bonus 2 | choicePool | choiceBonusSlot |
|---|-----------|---------|---------|------------|-----------------|
| 1 | Rebut | Deplacement | Endurance | -- | 0 |
| 2 | Citoyen | Technique | Discretion | -- | 0 |
| 3 | Habitant des Territoires Libres | Savoir | Parole | -- | 0 |
| 4 | Celebrite | Aura | Parole | -- | 0 |
| 5 | Survivant | Endurance | Hargne | -- | 0 |
| 6 | Membre d'une Societe Secrete | Savoir | *(pool)* | [Combat, Tir] | 2 |
| 7 | Agent du Nodachi | *(pool)* | Dexterite | [Combat, Tir] | 1 |
| 8 | Membre d'un Service Secret | *(pool)* | Discretion | [Combat, Tir] | 1 |
| 9 | Genie | Savoir | Technique | -- | 0 |
| 10 | Artiste | Hargne | Aura | -- | 0 |
| 11 | Independant | Perception | Instinct | -- | 0 |
| 12 | Religieux | Sang-Froid | Hargne | -- | 0 |
| 13 | Leader | Aura | Instinct | -- | 0 |
| 14 | Hors-la-loi | Sang-Froid | Discretion | -- | 0 |
| 15 | Voyageur | Deplacement | Perception | -- | 0 |
| 16 | Combattant | Combat | Tir | -- | 0 |
| 17 | Force de la Nature | Force | Endurance | -- | 0 |

**Validation:** Choice archetypes resolve randomly at generation (50/50). Characteristic bonus wasted if already at parent Aspect cap.

---

### 2.3 HautFaitDataSO (SPEC-02.8)

**Path:** `Scripts/Data/HautFaitDataSO.cs`
**Instances:** 14
**Purpose:** Heroic deed providing +1 Aspect, +2 characteristic points, and a callsign.

```csharp
class HautFaitDataSO : ScriptableObject {
    string hautFaitId;
    string displayName;
    string[] surnoms;            // Callsign options (3 per entry)
    AspectId bonusAspect;        // Single or first of dual
    AspectId bonusAspectAlt;     // Nullable. For dual-aspect entries (#4, #11, #12)
    int bonusCharPoints;         // Always 2
    ConditionType conditionType; // ASPECT or EITHER_CHAR
    CharacteristicId condChar1;  // Nullable
    CharacteristicId condChar2;  // Nullable
    int condMinScore;            // Default 4
}
```

**Instance table (14 entries):**

| # | Haut Fait | Bonus Aspect | Condition | Surnoms |
|---|-----------|-------------|-----------|---------|
| 1 | Combat de Titans | Chair | Chair >= 4 | Le Poing, le Force ne, le Colosse |
| 2 | En Territoire Ennemi | Bete | Bete >= 4 | Le Discret, l'Ombre, le Fantome |
| 3 | Le Renouveau de l'Espoir | Bete | Hargne >= 4 OR Sang-Froid >= 4 | L'Espere, le Lumineux, l'Innocent |
| 4 | Heros de Guerre | **Bete OR Machine** | Combat >= 4 OR Tir >= 4 | Le Champion, le Herault, le Guerrier |
| 5 | Construction Premiere Arche | Machine | Technique >= 4 OR Savoir >= 4 | L'Architecte, le Createur, le Constructeur |
| 6 | Conception Meta-Armure | Machine | Technique >= 4 OR Savoir >= 4 | L'Armurier, le Forgeron, le Fabricant |
| 7 | Decouverte Tache Dissimulee | Masque | Perception >= 4 OR Instinct >= 4 | Le Limier, le Traqueur, l'Observateur |
| 8 | Tueur de Tenebres | Masque | Discretion >= 4 OR Perception >= 4 | Le Nettoyeur, l'Assassin, le Bourreau |
| 9 | Guide | Dame | Parole >= 4 OR Perception >= 4 | Le Guide, le Voyageur, le Capitaine |
| 10 | Creation du Knight | Dame | Aura >= 4 OR Combat >= 4 | Le Fondateur, le Senechal, l'Allie |
| 11 | Defenseur de l'Art | **Bete OR Dame** | Hargne >= 4 OR Sang-Froid >= 4 | Le Mecene, le Gardien, le Conservateur |
| 12 | Sauveur | **Masque OR Dame** | Perception >= 4 OR Sang-Froid >= 4 | Le Protecteur, le Sauveur, le Garde du corps |
| 13 | Survivant Peste Rouge | Chair | Endurance >= 4 OR Hargne >= 4 | Le Survivant, l'Abime, le Devaste |
| 14 | Torture dans les Tenebres | Chair | Endurance >= 4 OR Force >= 4 | Le Torture, le Tenebreux, le Sombre |

**Dual-Aspect Resolution** (entries #4, #11, #12): Weighted by armor class primary aspects.

| # | Aspect A / Aspect B | Warrior | Paladin | Priest | Rogue |
|---|---------------------|---------|---------|--------|-------|
| 4 | Bete / Machine | 75/25 | 25/75 | 25/75 | 75/25 |
| 11 | Bete / Dame | 75/25 | 50/50 | 25/75 | 75/25 |
| 12 | Masque / Dame | 50/50 | 50/50 | 25/75 | 75/25 |

**Validation:** Condition check after Tarot bonuses applied. Max 10 retries on failure, then waive condition.

---

### 2.4 BlasonDataSO (SPEC-02.5)

**Path:** `Scripts/Data/BlasonDataSO.cs`
**Instances:** 9 active (Le Dragon #4 deferred)
**Purpose:** Animal emblem defining a Voeu (vow) that becomes the 3rd minor motivation.

```csharp
class BlasonDataSO : ScriptableObject {
    string blasonId;
    string displayName;
    string animalIcon;           // Animal emblem identifier
    string voeuText;             // French description of the vow
    string voeuEventTag;         // e.g., "VOEU_AIGLE"
}
```

**Instance table (9 active):**

| # | Blason | Tag | Voeu Trigger |
|---|--------|-----|-------------|
| 1 | L'Aigle | VOEU_AIGLE | Stabilize a Despaired ally |
| 2 | L'Ours | VOEU_OURS | Heal an ally from Agony (PS=0 to PS>=1) |
| 3 | Le Cerf | VOEU_CERF | Never change rank during an entire combat node |
| 5 | Le Faucon | VOEU_FAUCON | Solo kill a T3+ enemy (no Assist used) |
| 6 | Le Lion | VOEU_LION | Deploy at Rank 1, never retreat to higher rank |
| 7 | Le Loup | VOEU_LOUP | Use Nod d'Armure on ally at <=25% maxPA |
| 8 | Le Sanglier | VOEU_SANGLIER | Survive mission with injury/Fold/low PEs |
| 9 | Le Serpent | VOEU_SERPENT | First to attack a new enemy type |
| 10 | Le Taureau | VOEU_TAUREAU | Complete mission: no Despair, no ally deaths |

**Validation:** Voeu event tag must not duplicate the knight's 2 drawn minor motivation tags. Max 10 retries on collision.

---

### 2.5 TarotCardSO (SPEC-24)

**Path:** `Scripts/Data/TarotCardSO.cs`
**Instances:** 22 (Major Arcana 0--XXI)
**Purpose:** Each card grants +1 Aspect, +3 characteristic points, 1 advantage, and 1 disadvantage.

```csharp
class TarotCardSO : ScriptableObject {
    int cardId;                  // 0-21 (Roman numeral mapping)
    string displayName;          // e.g., "Le Bateleur"
    AspectId linkedAspect;       // Nullable for Le Fou (0) and La Maison-Dieu (XVI)
    int charPoints;              // 3 per card (6 for Le Fou, 0+2 Aspect pts for Maison-Dieu)
    TarotAdvantage advantage;
    TarotDisadvantage disadvantage;
}

struct TarotAdvantage {
    string advantageId;
    string description;          // French text
    string mechanicalEffect;     // Implementation key
}

struct TarotDisadvantage {
    string disadvantageId;
    string description;          // French text
    string mechanicalEffect;     // Implementation key
}
```

**Aspect bonus table:**

| Card | Aspect +1 | Char Points | Special |
|------|-----------|-------------|---------|
| 0. Le Fou | (none) | +6 free (all 15 chars weighted) | No Aspect bonus |
| I. Le Bateleur | Dame | +3 Dame chars | -- |
| II. La Papesse | Masque | +3 Masque chars | -- |
| III. L'Imperatrice | Machine | +3 Machine chars | -- |
| IV. L'Empereur | Dame | +3 Dame chars | -- |
| V. Le Pape | Machine | +3 Machine chars | -- |
| VI. L'Amoureux | Bete | +3 Bete chars | -- |
| VII. Le Chariot | Bete | +3 Bete chars | -- |
| VIII. La Justice | Machine | +3 Machine chars | -- |
| IX. L'Ermite | Machine | +3 Machine chars | -- |
| X. La Roue | Bete | +3 Bete chars | -- |
| XI. La Force | Chair | +3 Chair chars | -- |
| XII. Le Pendu | Chair | +3 Chair chars | Grants 4th minor motivation |
| XIII. L'Arcane sans Nom | Masque | +3 Masque chars | Incompatible with XVI |
| XIV. La Temperance | Chair | +3 Chair chars | -- |
| XV. Le Diable | Bete | +3 Bete chars | -- |
| XVI. La Maison-Dieu | (none) | +2 Aspect pts (random from primary) | No char pts; incompatible with XIII |
| XVII. L'Etoile | Masque | +3 Masque chars | -- |
| XVIII. La Lune | Masque | +3 Masque chars | -- |
| XIX. Le Soleil | Dame | +3 Dame chars | -- |
| XX. Le Jugement | Dame | +3 Dame chars | -- |
| XXI. Le Monde | Chair | +3 Chair chars | Grants 4th minor motivation (dormant in demo) |

**Key advantages (combat-relevant):**

| Card | Advantage | Mechanic |
|------|-----------|----------|
| I | Infatigable | No PS loss from PA passthrough (1 per 5 PA normally) |
| V | Forteresse Spirituelle | +5 maxPEs at creation |
| IX | Esprit d'Acier | All PEs losses reduced by 1 (min 0) |
| X | Chanceux | Once per mission: reroll one failed test |
| XI | Dur a Cuire | +5 maxPS at creation |
| XIII | Trompe la Mort | Reroll Mort injury result (once per mission) |
| XIV | Guerison Rapide | Nod de Soin: +3 flat bonus (3D6+3) |
| 0 | Chevalier Veritable | Heroic Mode triggers at 4 Heroisme (not 6) |

**Key disadvantages (combat-relevant):**

| Card | Disadvantage | Mechanic |
|------|-------------|----------|
| VIII | Trop Prudent | Initiative: 2D6 instead of 3D6 |
| IX | Solitaire | +1 difficulty when acting as assistant |
| XIII | Veteran | Aspect cap reduced to 7 (not 9) |
| XV | Brute | Machine aspect cap at 5 |
| XII | Sacrifice Total | No Major Motivation; majorMotivation = null |
| XX | Prisonnier | All PEs recovery blocked until freed (future) |

**Validation:** Cards XIII and XVI are incompatible -- if both drawn, redraw one. Amnesique (XVI disadvantage) replaces one advantage post-selection.

---

### 2.6 WeaponProfileSO (SPEC-19)

**Path:** `Scripts/Data/WeaponProfileSO.cs`
**Instances:** 8 demo weapons + 5 grenade types
**Purpose:** Weapon definition with one or more damage profiles.

```csharp
class WeaponProfileSO : ScriptableObject {
    string weaponId;
    string displayName;
    WeaponCategory category;     // STANDARD, ADVANCED, RARE
    WeaponSlot slot;             // CONTACT, RANGED, DUAL_PROFILE
    int pgCost;                  // Glory point cost
    DamageProfile[] profiles;    // One or more profiles
    ProfileSwitchCost profileSwitchCost; // FREE or MOVEMENT_ACTION
}

struct DamageProfile {
    string profileName;          // e.g., "Contact", "Tir", "Grenade Explosive"
    ProfileType type;            // CONTACT or RANGED
    int damageDice;              // Number of D6 for damage
    int damageFlat;              // Flat bonus to damage roll
    bool addForce;               // If true, add Force score
    ForceMode forceMode;         // NONE, NORMAL, LESTE
    int violenceDice;            // D6 for violence (vs Bandes)
    int violenceFlat;            // Flat bonus to violence
    WeaponRange range;           // CONTACT through LOINTAINE
    int energyCost;              // PE per use (0 = free)
    WeaponEffectId[] effects;    // References to SPEC-20 effect definitions
}
```

**Demo weapon roster:**

| Weapon | Slot | Profiles | Key Effects | PG |
|--------|------|----------|-------------|-----|
| Pistolet de Service | DUAL_PROFILE | Contact (knife) + Tir (pistol) | Silencieux | 15 |
| Marteau-Epieu | DUAL_PROFILE | Contact (hammer) + Tir (charge) | Perce Armure 40, Degats Continus 3 | 10 |
| Couteau de Combat | CONTACT | Contact (knife) | Orfevrerie, Silencieux, Jumele Ambidextrie | 15 |
| Fusil de Precision | RANGED | Tir (sniper) | Tir en Securite, Precision, Deux Mains | 40 |
| Fusil d'Assaut | RANGED | Tir (assault) | Ultraviolence, Barrage 2, Deux Mains | 30 |
| Pistolet Mitrailleur | RANGED | Tir (SMG) | Meurtrier, Ultraviolence, Jumele Akimbo | 25 |
| Shotgun Escamotable | RANGED | Tir (shotgun) | Meurtrier, Choc 1, Barrage 2, Deux Mains | 20 |
| Lance-Grenade Leger | RANGED | Explosive / Antiblindage / Incendiaire | Dispersion 3, Destructeur | 40 |

**Default loadouts by armor class:**

| Armor Class | Weapons |
|-------------|---------|
| Warrior | Marteau-Epieu + Fusil d'Assaut + Pistolet de Service |
| Paladin | Marteau-Epieu + Shotgun Escamotable + Pistolet de Service |
| Priest | Marteau-Epieu + Fusil de Precision + Pistolet de Service |
| Rogue | 2x Couteau de Combat + Pistolet de Service |

All knights also carry 5 Grenades Intelligentes and 3 Nods of each type.

**Grenade types:**

| Type | Damage | Violence | Key Effects |
|------|--------|----------|-------------|
| Shrapnel | 3D6 | 3D6 | Ultraviolence, Meurtrier, Dispersion 5 |
| Flashbang | 0 | 0 | Choc 1 (auto), Barrage 2, Lumiere 2, Dispersion 5 |
| Anti-Blindage | 3D6 | 3D6 | Destructeur, Perce Armure 20, Penetrant 6, Dispersion 5 |
| IEM | 0 | 0 | Parasitage 2, Dispersion 5 |
| Explosive | 3D6 | 3D6 | Anti-Vehicule, Choc 1, Dispersion 3 |

---

### 2.7 EnemyBaseSO (SPEC-18)

**Path:** `Scripts/Data/EnemyBaseSO.cs`
**Instances:** 5 demo enemy types + boss phases
**Purpose:** Runtime data structure for all enemy combatants. Mirrors KnightBase symmetry.

```csharp
class EnemyBaseSO : ScriptableObject {
    // --- Identity ---
    string enemyId;
    string displayName;
    EnemyTier tier;              // T1_BANDE through T5_PATRON
    SeigneurId seigneur;

    // --- Aspects (all 5, always present, range 0-20) ---
    int chair, bete, machine, dame, masque;

    // --- Aspects Exceptionnels (nullable per aspect) ---
    AspectExceptionnel chairExc, beteExc, machineExc, dameExc, masqueExc;

    // --- Derived Values (hand-authored, NOT computed) ---
    int defense;                 // Opposition for melee attacks
    int reaction;                // Opposition for ranged attacks
    int initiative;

    // --- Tier-dependent health ---
    // T1 Bande only:
    int cohesion;                // 100/200/300/450
    int debordement;             // Cumulative damage per turn

    // T2+ Individual:
    int maxPS, currentPS;
    int maxPA, currentPA;        // Default 0 if not specified
    int bouclier;                // Shield, default 0. T3+ only.

    // --- Special flags ---
    bool isColosse;              // T4: 10 raw damage = 1 effective
    bool immuneToForcedDisplacement; // Default false. Behemot = true.
    string pointFaible;          // Characteristic name, nullable

    // --- T5 Boss ---
    BossPhase[] phases;          // Phase transitions (T5 only)

    // --- Combat capabilities ---
    WeaponProfileSO[] weapons;
    EnemyCapacity[] capacities;
    EnemyAI aiProfile;

    // --- Runtime ---
    int combatActionsRemaining;  // Base 1 + N from Actions Multiples
    int movementActionsRemaining; // Always 1
    bool isDead;
}

struct AspectExceptionnel {
    bool isMajeur;               // Majeur includes Mineur
    int score;                   // 1-10: +N auto-successes
}

struct EnemyAI {
    TargetPriority primary;
    TargetPriority secondary;
    bool preferHighest;
    RankBehavior rankBehavior;
    int aggressionLevel;         // 1-3
}

struct BossPhase {
    int phaseIndex;
    int psThreshold;             // PS value that triggers transition
    int newMaxPS;
    int newBouclier;
    int newDefense;
    int actionsMultiples;        // Extra combat actions
    WeaponProfileSO[] newWeapons;
    EnemyCapacity[] newCapacities;
    EnemyAI newAiProfile;
}

struct EnemyCapacity {
    string capacityId;
    string description;
    int cooldownPerCombat;       // -1 = unlimited
    bool resetsOnPhaseTransition;
}
```

**Demo enemy roster:**

| Enemy | Tier | Aspects (C/B/M/D/Mq) | Def/React/Init | HP | Special |
|-------|------|----------------------|----------------|-----|---------|
| Nocte | T1 Bande | 4/9/1/0/4 | 5/1/1 | Cohesion 200 | Debordement 4, IGNORE_CDF, Hypersensibilite Lumineuse |
| Bestian | T2 Hostile | 4/8/2/1/5 | 4/3/2 | 20 PS | Bete Mineur(2), Saut nv1, Anatheme ranged attack |
| Faune | T3 Salopard | 8/10/4/4/7 | 6/2/4 | 80 PS | Bete Majeur(6), Masque Mineur(1), Charge Brutale, Actions Multiples(1) |
| Behemot | T4 Colosse | 16/16/6/2/2 | 8/3/5 | 200 PS, 60 PA | Chair Majeur(5), Bete Majeur(6), Charge Brutale, Peur(1), immune displacement |
| Ours Corrompu P1 | T5 Patron | 12/14/6/4/4 | 8/4/6 | 150 PS, 30 PA, Bouclier 8 | Chair Mineur(3), Bete Majeur(6), Charge Brutale, Peur(2) |
| Ours Corrompu P2 | T5 Patron | 12/14/6/4/4 | 10/4/6 | 120 PS, 30 PA, Bouclier 12 | + Ombre devorante (Anatheme), Regeneration, Actions Multiples(2) |

---

### 2.8 EncounterSO (SPEC-29)

**Path:** `Scripts/Data/EncounterSO.cs`
**Instances:** 5 (one per mission node with combat/boss)
**Purpose:** Canonical encounter format for content/UI/combat agents.

```csharp
[CreateAssetMenu]
class EncounterSO : ScriptableObject {
    string encounterId;          // Stable identifier
    string missionId;
    int nodeIndex;
    EncounterType type;          // Combat, Boss, Event, Rest
    CoverLayout cover;           // Per-rank cover flags
    PlayerStartLayout playerStart;
    List<EnemySpawnEntry> initialEnemies; // Max 4 on-track
    ReinforcementQueue reinforcement;
    BandeConfig bande;           // Off-track bande configuration
    List<EncounterTrigger> triggers;
}

struct CoverLayout {
    bool r1, r2, r3, r4;
}

struct EnemySpawnEntry {
    string enemyId;
    int startRank;               // 1-4
    bool startsInCover;
    bool isElite;                // Informational tag
}

struct ReinforcementQueue {
    List<string> enemyIdsInOrder;
    ReinforcementRule rule;      // WhenSlotFree, OnTurnN, OnTrigger
    int turnN;
}

struct BandeConfig {
    bool enabled;
    string bandeId;
    int cohesionMax;
    int cohesionStart;
    int debordementScore;
}

struct EncounterTrigger {
    TriggerCondition cond;
    TriggerAction action;
}

struct TriggerCondition {
    TriggerEvent triggerEvent;
    string filterEnemyId;        // Null = match all
    int filterTurnIndex;         // -1 = any turn
    int filterPhaseIndex;        // -1 = any phase
}

struct TriggerAction {
    TriggerActionType actionType;
    string targetEnemyId;
    int targetRank;              // 0 = auto
    string effectId;
    int value;                   // Generic payload (cohesion, etc.)
    string aiProfileOverride;
}
```

**Validation rules:**
- `initialEnemies.Count <= 4`
- Ranks must be 1--4
- `startsInCover` requires cover at that rank
- Reinforcement spawns must never exceed 4 on-track enemies
- `encounterId` must be stable (seed derivation relies on it)

---

### 2.9 InjuryTableSO (SPEC-23)

**Path:** `Scripts/Data/InjuryTableSO.cs`
**Instances:** 1 (singleton)
**Purpose:** 4x6 injury lookup matrix (severity column x d6 row).

```csharp
class InjuryTableSO : ScriptableObject {
    InjuryEntry[,] entries;      // [4, 6] — column=severity 1-4, row=1-6
}

struct InjuryEntry {
    string injuryId;
    InjuryEffect[] effects;     // Mechanical effects (stat reductions, etc.)
    bool isPermanent;            // Permanent until implant/therapy
    string description;          // French flavor text
}
```

**Lookup:** Roll 1D6 for row. Severity determined by how far below 0 PS dropped. Column 4 at maximum severity includes `Mort` (death) results.

---

### 2.10 MotivationTemplateSO (SPEC-12.1)

**Path:** `Scripts/Data/MotivationTemplateSO.cs`
**Instances:** 10 minor + 5 major = 15
**Purpose:** Templates for procedural motivation assignment at knight generation.

```csharp
class MotivationTemplateSO : ScriptableObject {
    string motivationId;         // e.g., "MN-01", "MJ-01"
    MotivationType type;         // MINOR, MAJOR, VOEU
    string displayText;          // French flavor text
    string eventTag;             // Matched by SPEC-12 event bus
}
```

**Minor motivation templates (10):**

| ID | Tag | Trigger |
|----|-----|---------|
| MN-01 | ALLY_SAVED_FROM_DEATH | Nearby ally takes 0 PS from hit at <=10% PS |
| MN-02 | MULTI_KILL_3 | Kill 3+ enemies in one turn |
| MN-03 | EXPLOIT_LANDED | Land an Exploit that hits |
| MN-04 | ONE_HIT_KILL | One attack: >=50% PS to 0 |
| MN-05 | SURVIVED_LOW_PA | Survive node at <=25% maxPA |
| MN-06 | DESPAIR_STABILIZED | Stabilize a Despaired ally |
| MN-07 | LONG_RANGE_KILL | Kill at weapon's max range |
| MN-08 | CDF_FULL_ABSORB | CdF absorbs 100% of an attack |
| MN-09 | GHOST_KILL | Kill while Ghost mode active |
| MN-10 | COMBAT_REPAIR | Priest restores PA via NanoC/Mechanic in combat |

**Major motivation templates (5):**

| ID | Tag | Trigger | Reward |
|----|-----|---------|--------|
| MJ-01 | MISSION_NO_DEATHS | Complete mission with 0 deaths | +25 PEs current+max |
| MJ-02 | BOSS_KILLED | Kill assigned boss | +25 PEs current+max |
| MJ-03 | MISSION_HEALTHY_FINISH | All 4 knights >50% maxPA at end | +25 PEs current+max |
| MJ-04 | HIGH_DAMAGE_DEALER | Deal 100+ effective PS damage in mission | +25 PEs current+max |
| MJ-05 | FOLD_SURVIVOR | Enter Fold during mission and survive | +25 PEs current+max |

---

### 2.11 MissionDefinitionSO (SPEC-22)

**Path:** `Scripts/Data/MissionDefinitionSO.cs`
**Instances:** 1 (demo mission: "The Corrupted Den")
**Purpose:** Complete node graph with encounters, narrative events, and branch points.

```csharp
class MissionDefinitionSO : ScriptableObject {
    string missionId;
    string displayName;
    MissionNode[] nodes;         // 7 nodes (including branches)
    BranchPoint[] branchPoints;
}

struct MissionNode {
    int nodeIndex;
    string nodeId;               // Stable for seed derivation
    string displayName;
    EncounterType type;
    string encounterId;          // Reference to EncounterSO
    NarrativeEvent[] events;
}

struct BranchPoint {
    int fromNodeIndex;
    int[] branchNodeIndices;     // e.g., [2A, 2B]
    int convergeNodeIndex;       // Node 3
}

struct NarrativeEvent {
    string eventId;
    int nodeIndex;
    string triggerType;          // AUTO, PLAYER_CHOICE, PHASE_TRANSITION, ROLL_CHECK
    string description;
    float rollThreshold;         // -1 if N/A
    NarrativeEffect[] effects;
}

struct NarrativeEffect {
    string targetScope;          // ALL_KNIGHTS, SINGLE_KNIGHT, SPECIFIC_KNIGHT_ID
    string selectorRule;         // HIGHEST_PERCEPTION, RANDOM, etc.
    string effectType;           // PES_GAIN, PES_LOSS, REVEAL_POINT_FAIBLE, GRANT_HEROISME
    string diceFormula;          // "1D6", "+3", "-1D6"
    string metadata;             // Extra data (e.g., Point Faible characteristic)
}
```

**Demo mission node graph:**

| Node | Type | Encounter | Description |
|------|------|-----------|-------------|
| 1: Perimeter | Combat | 1 Nocte bande (200) | Teaches bande/violence mechanics |
| 2A: Nest | Combat | 2 Bestians + 1 Nocte bande (100) | Individual enemy + Anatheme |
| 2B: Ruins | Combat+Event | 1 Faune + survivor event | Narrative: +1D6 PEs to all |
| 3: Breach | Combat | 1 Faune + 2 Bestians | Hardest non-boss fight |
| 4: Antechamber | Narrative | No combat | Point Faible discovery (1/6), prayer option |
| 5: The Den | Boss | Ours Corrompu + Nocte bande (200) | Two-phase boss fight |

---

## 3. Runtime State Classes (C#)

Runtime state uses plain C# classes/structs (serializable where needed). No gameplay logic in MonoBehaviour Update().

---

### 3.1 RunState (SPEC-32)

**Lifetime:** Persistent across combats and mission nodes. Serialized for save system.
**Boundary:** The ONLY state that crosses Hub<->Combat boundary.

```csharp
class RunState {
    KnightRuntimeState[] knights; // 8 knights
    int pgTotal;                  // Points de Gloire (meta-currency)
    MissionProgress missionProgress;
    uint runSeed;                 // Generated once at run start (unsigned 32-bit)
    string currentMissionId;
    int currentNodeIndex;
}

struct KnightRuntimeState {
    // Snapshot of all RunState-persistent KnightBase fields
    // (see SPEC-32 field boundary table for full list)
    string knightId;
    int currentPS, maxPS, currentPA, currentPE;
    int currentPEs, maxPEs;
    int heroisme;
    int currentCdF;
    List<InjuryResult> activeInjuries;
    int implantCount;
    NodInventory nods;
    bool isDead;
    bool hasHemorragie;
    int hemorragieCountdown;
    // ... all other RunState-persistent fields per SPEC-32
}
```

**Fields written back atomically on combat exit (`ApplyCombatResults`):**
- `currentPS`, `currentPA`, `currentPE`, `currentPEs`, `currentCdF`
- `heroisme`, `activeInjuries`, `isDead`, `nods`
- `hasHemorragie`, `hemorragieCountdown`
- `pgTotal += earnedPG`

**Fields NEVER modified by combat:**
- Aspects, Characteristics, OD levels, maxPA, maxPE, baseCdF, equipment, generation data
- These are only modified at Camelot Hub

---

### 3.2 CombatSession (SPEC-32)

**Lifetime:** Created at deployment, destroyed on combat exit. Ephemeral.
**Isolation:** No reference to CombatSession may persist after combat exit.

```csharp
class CombatSession {
    CombatantState[] deployedKnights; // 4 max
    CombatantState[] enemies;
    TurnQueue turnQueue;
    BandeRuntimeState activeBande;
    IRng rngCombat;              // SPEC-28: Combat stream
    IRng rngLoot;                // SPEC-28: Loot stream
    EncounterSO encounterData;   // Reference to encounter definition
    int earnedPG;
    List<StatusEffect> activeEffects;
}

struct BandeRuntimeState {
    int cohesion;
    int debordementScore;
    int debordementTurn;         // Starts at 1, increments after damage
    bool isActive;
}
```

**CombatSession-only fields** (reset at combat start, NOT written back):

| Field | Reset Value | Notes |
|-------|-------------|-------|
| position | Set by deployment | Rank 1-4 |
| activeStyle | Standard | Reset each combat |
| isInAgony | false | -- |
| isInFoldState | false | -- |
| isDespair | false | -- |
| isGhostActive | false | -- |
| activeWarriorTypeIndex | null | -- |
| activeStatusEffects | empty | All cleared between encounters |
| Active Barrage debuffs | empty | -- |
| Choc/Parasitage counters | 0 | -- |
| Shrine active state | false | Priest must re-place |

---

### 3.3 CombatantState (SPEC-30)

**Purpose:** Canonical state model preventing inconsistent incapacitation handling. Single authoritative source for "can act / is ally / is targetable" queries.

```csharp
struct CombatantState {
    LifeState life;              // Alive, Dead
    ControlState control;        // PlayerControlled, EnemyControlled
    ActionState action;          // CanAct, Incapacitated
    ArmorState armor;            // Normal, Folded

    bool isDespair;
    bool isDespairPermanent;
    int despairDuration;         // -1 = inactive, 1D6 initial, 0 = permanent

    int hemorragieCountdown;     // -1 = inactive, 3..1 = turns remaining

    int position;                // 1-4 (rank)
    CombatStyle activeStyle;

    // All runtime combat fields from SPEC-32 field boundary table
}
```

**Canonical defeat evaluation** (the ONLY defeat check, used everywhere):

```csharp
bool IsSquadDefeated(List<CombatantState> knights) {
    var living = knights.Where(k => k.life == LifeState.Alive);
    if (!living.Any()) return true;
    return living.All(k =>
        k.action == ActionState.Incapacitated ||
        (k.isDespair && k.isDespairPermanent &&
         k.control == ControlState.EnemyControlled));
}
```

**State rules:**
- Dead: `life == Dead`
- Agony: sets `action = Incapacitated`
- Despair: sets `control = EnemyControlled` (temporary or permanent)
- Fold: sets `armor = Folded`, affects available actions
- Temporary Despair (`isDespairPermanent == false`) does NOT count as defeated

---

### 3.4 NodInventory (SPEC-33)

**Purpose:** Per-knight consumable recovery items. 3 of each type per mission.

```csharp
struct NodInventory {
    int soin;    // 0-3, starts at 3. Restores PS (3D6).
    int armure;  // 0-3, starts at 3. Restores PA (3D6).
    int energie; // 0-3, starts at 3. Restores PE (3D6).
}
```

**Usage rules:**
- In combat: costs 1 Movement Action. Self or ally (Courte range, Gap <= 2).
- Between nodes: free, no range restriction.
- Ally in Agony: Nod de Soin permitted (exits Agony). Self in Agony: not permitted.
- Guerison Rapide advantage: Nod de Soin becomes 3D6+3.
- Refilled to 3/3/3 at Camelot between missions (free).

---

## 4. Derived Value Formulas

These values are computed at runtime, never stored directly. Recalculate on dependency change.

| Derived Value | Formula | Recalculate When |
|---------------|---------|------------------|
| Defense | `highestCharIn(BETE) + odOf(that char)` | Bete char/OD change, Type change |
| Reaction | `highestCharIn(MACHINE) + odOf(that char)` | Machine char/OD change, Type change |
| Initiative | `highestCharIn(MASQUE) + odOf(that char)` | Masque char/OD change, Type change |
| maxPS | `10 + 6 * Max(Force, Endurance, Deplacement)` [OD excluded] | Chair char changes |
| maxPEs | `50 + tarotMods - (implantCount * 3)` | Creation; implant/therapy at Camelot |

**Pattern for Defense/Reaction/Initiative:** Pick the single highest-scoring Characteristic within the relevant Aspect, then add only that Characteristic's OD.

```csharp
// Pseudocode
int Defense  => highestCharIn(BETE)    + odOf(highestCharIn(BETE));
int Reaction => highestCharIn(MACHINE) + odOf(highestCharIn(MACHINE));
int Initiative => highestCharIn(MASQUE) + odOf(highestCharIn(MASQUE));
int maxPS    => 10 + 6 * highestCharIn(CHAIR);  // OD excluded
```

**Worked example (SPEC-01-AC1):**
Knight with Bete chars Combat 5 (OD 2), Instinct 3 (OD 0), Hargne 4 (OD 1):
Defense = 5 + 2 = 7 (Combat is highest).

**Major Motivation bonus:** +25 maxPEs and +25 currentPEs independently on fulfillment.

**Enemy derived values:** Defense, Reaction, Initiative are hand-authored in stat blocks (SPEC-21). Do NOT derive from formulas for enemies.

---

## 5. Validation Rules

### Knight Generation Validation

| Rule | Constraint | Source |
|------|-----------|--------|
| Characteristic cap | Each Characteristic <= parent Aspect score | SPEC-01 |
| Aspect cap (normal) | All Aspects <= 9 | SPEC-01 |
| Aspect cap (Veteran) | All Aspects <= 7 (if Tarot XIII disadvantage) | SPEC-24 |
| Aspect cap (Brute) | Machine <= 5 (if Tarot XV disadvantage) | SPEC-24 |
| OD cap (normal) | OD <= 5 from all sources | SPEC-01 |
| OD cap (Warrior Type) | OD <= 6 (Warrior Type +1 is the ONLY exception) | SPEC-17.1 |
| Tarot incompatibility | Cards XIII and XVI cannot both appear in same draw | SPEC-24 |
| Blason uniqueness | Voeu event tag must not duplicate drawn MN tags | SPEC-02.5 |
| Motivation uniqueness | No duplicate motivations within a single knight | SPEC-02 |
| Generation determinism | Same seed = identical results | SPEC-02/28 |

### Encounter Validation

| Rule | Constraint | Source |
|------|-----------|--------|
| Enemy count | initialEnemies.Count <= 4 | SPEC-29 |
| Rank validity | All ranks 1-4 | SPEC-29 |
| Cover consistency | startsInCover requires cover[rank] == true | SPEC-29 |
| Reinforcement cap | Never exceed 4 on-track enemies | SPEC-29 |
| ID stability | encounterId never regenerated after creation | SPEC-29 |

### Runtime Validation

| Rule | Constraint | Source |
|------|-----------|--------|
| Characteristics floor | Characteristics >= 0 after injury | SPEC-01 |
| Defense/Reaction floor | Minimum 0 after all modifiers | Global |
| Heroism range | 0-6 | SPEC-09 |
| Implant limit | 0-6, maxPEs floor 10 | SPEC-01/25 |
| PA passthrough | floor(paAbsorbed / 5) PS lost per hit | SPEC-06 |
| Hit threshold | Successes must strictly exceed Defense/Reaction | Global |

---

## 6. State Transitions

### 6.1 Knight Lifecycle

```
                    +-------+
                    | Normal|
                    +---+---+
                   /    |    \
                  v     v     v
            +------+ +------+ +--------+
            | Fold | | Agony| | Despair|
            +--+---+ +--+---+ +---+----+
               |         |         |
               v         v         v
           +------+  +------+  +----------+
           |Normal|  |Normal|  | Normal   |
           |(PA>5)|  |(heal)|  |(stabilize)|
           +------+  +------+  +----------+
                      |              |
                      v              v
                   +------+    +-----------+
                   | Dead |    | Permanent |
                   +------+    | Despair   |
                               +-----------+
```

**Fold State** (PA = 0):
- Guardian mode: PA=5, CdF=5, no PE, no OD, no modules
- Exit: PA > 5 via Nod d'Armure or Priest Mechanic

**Agony** (PS = 0):
- Injury roll on SPEC-23 table
- Hemorragie countdown: 3 -> 2 -> 1 -> 0 (dead)
- Exit: healed by Nod de Soin (PS >= 1)

**Despair** (PEs = 0):
- Hostile control for 1D6 turns
- Duration 0 = permanent Despair
- Exit: stabilized by ally (SPEC-07.3)

### 6.2 Combat Flow

```
Deployment -> Initiative Roll -> Turn Loop -> Victory/Defeat
     |                                              |
     v                                              v
  Set positions                            ApplyCombatResults
  Reset combat state                       (atomic commit to RunState)
  Create CombatSession                     Destroy CombatSession
```

### 6.3 Mission Flow

```
Camelot Hub
  -> Squad Select (4 of 8)
    -> Deploy
      -> Node 1 (Perimeter: Nocte bande)
        -> Node 2 (Branch: 2A Nest OR 2B Ruins)
          -> Node 3 (Breach: Faune + 2 Bestians)
            -> Node 4 (Antechamber: Narrative)
              -> Node 5 (The Den: Boss fight)
                -> Mission End
                  -> Camelot Hub
```

**Between nodes:** No automatic recovery. Nods usable (free). All state persists.
**At Camelot:** PS/PA/PE fully restored (free). PEs unchanged. Injuries persist. Nods/grenades/ammo auto-refilled.

### 6.4 Boss Phase Transition (Ours Corrompu)

```
Phase 1 (PS=150, PA=30, Bouclier 8)
  -> PS reaches 0 (does NOT enter Agony)
    -> Phase 2 immediately:
       PS resets to 120, PA 30, Bouclier 12
       Defense 8 -> 10
       Actions Multiples 1 -> 2
       New weapon: Ombre devorante (Anatheme)
       New capacity: Regeneration (50 Cohesion -> 25 PS)
       AI shifts: LOWEST_PEs, Aggression 3
       Nocte bande: AddCohesion(200) or fresh spawn
       All knights: -1D6 PEs
```

---

## 7. Entity Relationship Map

```
KnightBaseSO
  |-- ArchetypeDataSO         (1:1, assigned at generation)
  |-- HautFaitDataSO           (1:1, assigned at generation)
  |-- BlasonDataSO             (1:1, assigned at generation)
  |-- TarotCardSO[5]           (1:5 drawn, 2 adv + 1 disadv active)
  |-- WeaponProfileSO[]        (1:N equipped weapons)
  |-- Motivation[]             (1:3-4 minor + 0-1 major)
  |-- NodInventory             (1:1 embedded struct)
  |-- InjuryResult[]           (1:N active injuries)
  '-- CombatantState           (1:1 during combat only)

EnemyBaseSO
  |-- WeaponProfileSO[]        (1:N inline weapons)
  |-- EnemyCapacity[]          (1:N special abilities)
  |-- EnemyAI                  (1:1 targeting profile)
  |-- BossPhase[]              (1:N for T5 only)
  '-- CombatantState           (1:1 during combat only)

EncounterSO
  |-- EnemySpawnEntry[]        (max 4 on-track)
  |-- ReinforcementQueue       (1:1 optional)
  |-- BandeConfig              (1:1 optional off-track)
  '-- EncounterTrigger[]       (1:N scripted events)

MissionDefinitionSO
  |-- MissionNode[]            (7 nodes including branches)
  |-- BranchPoint[]            (1:N branch definitions)
  '-- NarrativeEvent[]         (per-node events)

RunState (persistent)
  |-- KnightRuntimeState[8]    (full roster)
  |-- MissionProgress          (current node, branch path)
  '-- uint runSeed             (RNG root)

CombatSession (ephemeral)
  |-- CombatantState[4]        (deployed knights)
  |-- CombatantState[]         (enemies)
  |-- TurnQueue                (initiative order)
  |-- BandeRuntimeState        (off-track bande)
  |-- IRng (Combat, Loot)      (deterministic streams)
  '-- EncounterSO              (reference to encounter data)
```

### Cross-System Data Flow

```
Generation Pipeline (SPEC-02)
  reads: ArchetypeDataSO, HautFaitDataSO, BlasonDataSO, TarotCardSO
  writes: KnightBaseSO[8]
  RNG: RunSeed (SPEC-28)

Combat Start
  reads: KnightBaseSO, EncounterSO, EnemyBaseSO
  creates: CombatSession (snapshot of RunState)

Combat Loop
  reads/writes: CombatSession (ephemeral state)
  references: WeaponProfileSO, InjuryTableSO, MotivationTemplateSO
  RNG: nodeSeed-derived streams (SPEC-28)

Combat Exit
  reads: CombatSession final state
  writes: RunState (atomic ApplyCombatResults commit)

Camelot Hub
  reads/writes: RunState (roster, PG, injuries)
  reads: WeaponProfileSO (for shop)
```

---

### Meta-Armor Base Stats (SPEC-02.7)

| Armor | PA | PE | CdF | Slots (H/T/AL/AR/LL/LR) | Base OD Characteristics |
|-------|----|----|-----|--------------------------|------------------------|
| Warrior | 100 | 40 | 8 | 7/10/10/12/7/7 | Deplacement, Combat, Tir, Dexterite |
| Paladin | 120 | 20 | 8 | 7/7/7/10/7/7 | Force, Endurance, Tir, Perception |
| Priest | 70 | 60 | 10 | 5/5/5/8/5/5 | Force, Endurance, Savoir, Technique |
| Rogue | 50 | 70 | 12 | 5/5/5/8/5/5 | Deplacement, Combat, Discretion, Dexterite |

---

### PG (Points de Gloire) Economy (SPEC-25)

**Earning:**

| Source | PG |
|--------|----|
| Mission success | 30 |
| All knights alive bonus | 10 |
| Kill Patron (boss) | 15 |
| Kill Colosse | 5 |
| Mode Heroique activation | 5 |
| Exploit roll | 1 |
| Minor Motivation fulfilled | 3 |
| Major Motivation fulfilled | 10 |

**Spending:**

| Purchase | PG Cost |
|----------|---------|
| Cybernetic Implant | 20 (removes 1 injury, maxPEs -3) |
| Reconstruction Therapy | 100 (removes ALL injuries, resets implants) |
| Standard Weapon | Per weapon PG value (SPEC-19) |
