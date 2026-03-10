# KNIGHT: INTO THE DARKNESS

### TECHNICAL SPECIFICATION DOCUMENT

**Version 8.8** — Pass 8 deep-dive validation (1H+5M+4L resolved: complete injury digital effects for all 24 entries, PEs penalty canonical formula, Pilonnage/Puissant style gate corrections, implant PEs penalty, Mise à couvert prose, HautFaitData standalone struct). L-4 (Phase 2 AI change) verified present in SDT — false positive. Builds on v8.7 (Pass 7 remediation).

**February 2026**

# PART 1 — MDD REVIEW FINDINGS
Line-by-line review of MDD v1.3. All 20 findings (5C + 5H + 6M + 4L) have been resolved. Every finding shows a green RESOLVED box. This document is complete.

## 1.1 Severity Legend

| Severity | Definition | Impact |
| --- | --- | --- |
| CRITICAL | Blocks implementation. Agent cannot produce correct code. | Build fails or wrong behavior. |
| HIGH | Ambiguity across systems. Agent may implement inconsistently. | Subtle bugs, cross-system mismatches. |
| MEDIUM | Requires agent assumptions that may differ from intent. | Non-fatal but potentially wrong. |
| LOW | Polish, completeness, or edge cases. | Minor, no functional impact. |


## 1.2 Critical Findings (ALL RESOLVED)


| C1+C2 | Damage = SUM all weapon dice face values + flat bonuses (Force, OD×3, effects). This is separate from the attack roll (count evens). Excess successes do nothing by default. Only Assistance à l'Attaque converts excess successes to +1 flat damage each. Dégâts Maximum (1 Héroïsme) maximizes all weapon dice to 6. See SPEC-05 for full chain. |
| --- | --- |



| C3 | Before first turn: player places each knight on P1–P4. Enemy positions from encounter data (EnemySpawn SO). Default suggestion: melee-oriented (Warrior/Barbarian) at P1–P2, ranged-oriented (Paladin/Priest/Rogue) at P3–P4. See SPEC-03. |
| --- | --- |



| C4 | PA=0: Guardian Suit (PA 5, CdF 5). OFFLINE: ODs, modules, signature abilities, PE spending. ACTIVE: weapons, Nods, free actions. PE pool frozen (not lost). Exit: any PA restoration (Nod d'Armure, Priest Mechanic) restores armor and reactivates all systems. See SPEC-10. |
| --- | --- |



| C5 | 2 per class × 4 classes = 8 knights. Warrior ×2, Paladin ×2, Priest ×2, Rogue ×2. Remove ambiguous 'remaining' language from §3.1.1. See SPEC-02. |
| --- | --- |


## 1.3 High Findings (ALL RESOLVED)


| H1 | Violence damages Bande Cohesion only. Never reduces knight PEs. Remove all MDD references to 'Violence damages PEs for individual knights.' |
| --- | --- |



| H2 | Added to §5.6 Step 2b: floor(paAbsorbed/5) = PS lost. Disabled by Infatigable. Mirrored in §10.3. |
| --- | --- |



| H3 | Bandes do not attack and never roll to hit. Débordement hits ALL enemies automatically each turn. No position restriction. Knights target Bande from any position with Courte+ range. |
| --- | --- |



| H4 | Added to §5.1: Combo chosen per-roll, may change each action, must differ from Base Characteristic. |
| --- | --- |



| H5 | Warrior Types, Paladin Shrine/Watchtower, Priest NanoC/Mechanic, Rogue Ghost — all with full stat blocks. |
| --- | --- |


## 1.4 Medium Findings (ALL RESOLVED)


| M1 | Tabletop confirms: 'Les résultats sont conservés durant toute la phase de conflit.' Initiative = 3D6 + Initiative score, rolled once at combat start. Results persist until combat ends. Trop prudent: 2D6 instead. Bandes always initiative 1. See SPEC-15. |
| --- | --- |



| M2 |  |
| --- | --- |



| M3 | Tabletop entraide has no positioning constraint. For the demo: assistance works from any position. Each assisting knight spends 1 Movement Action and rolls 1 Characteristic (not used by others). Successes added to the aided knight's roll. Max 3 assistants. |
| --- | --- |



| M4 | [v4] No automatic healing between combat nodes. PS, PA, PE, and PEs carry over exactly as-is. Knights may use Nods during node transitions (costs no action outside combat). This creates maximum attrition pressure and makes resource management critical. Matches roguelite design philosophy. |
| --- | --- |



| M5 |  |
| --- | --- |



| M6 | [v4] MAJOR CHANGE: Exactly 1 combatant per position. 4 positions per side = max 4 knights, max 4 individual enemies. Movement = swap with adjacent ally or shift into empty slot. Forced displacement chain-swaps. Dead combatant leaves empty slot. Squad: 4 knights selected from 8-knight roster at mission start, no changes mid-mission. Reinforcements only spawn into empty enemy slots. See SPEC-03 rewrite. |
| --- | --- |


## 1.5 Low Findings (ALL RESOLVED)


| L1 | Tabletop scale: Facile(1), Normale(2), Ardue(3), Difficile(4), Éprouvante(5), Délicate(6), Herculéen(9). Gap at 7–8 is by design. Add tooltip/note in UI confirming. [v7] NOTE ON NUMBERING: Numbered lists throughout this document (acceptance criteria, algorithm steps) may show non-sequential indices due to DOCX conversion artifacts. Agents should treat list items by ordinal position, not by displayed number. E.g., if steps show 5-14, read as steps 1-10. [v7] Difficulty-to-Threshold Mapping (for agent implementation):   Facile       = 1 success required   Normale      = 2 successes required   Ardue        = 3 successes required   Difficile    = 4 successes required   Éprouvante   = 5 successes required   Délicate     = 6 successes required   (7-8)        = UNUSED in demo — reserved for future content   Herculéen    = 9 successes required These thresholds apply to all test types: Peur resistance, Savoir checks, social tests. When a mechanic says 'difficulty reduced by 1 level', subtract 1 from the required successes (minimum 1). |
| --- | --- |



| L2 | Push past Rank 4 (back): clamped to Rank 4. Push past Rank 1 (front): clamped to Rank 1. Cannot push across center line. Already reflected in SPEC-03 v4. |
| --- | --- |



| L3 |  |
| --- | --- |



| L4 | Chargeur X: X uses per mission. No mid-mission reload. Auto-refill at Camelot. Grenades: 5/mission, same rule. Added to SPEC-04 notes. |
| --- | --- |


# PART 2 — SYSTEM SPECIFICATIONS
Each section defines one implementable system, ordered by dependency. Changes marked: [v2] H-level, [v3] C-level, [v4] M-level (DD ranks), [v5] Expert review: enemy/weapon/effects (18–20), enemy roster/mission (21–22), injury/tarot/hub (23–25). [v8] Acceptance Criteria numbering convention: use SPEC-NN-ACn prefix (e.g. SPEC-02-AC1) for unambiguous cross-referencing. Current document uses sequential numbering within sections due to DOCX conversion — agent teams should apply the prefix scheme during implementation.

## SPEC-01: Knight Data Model (KnightBase)
### Purpose
Runtime data structure for a knight. All other systems read/write this model.
### Data Schema

```csharp
class KnightBase : ScriptableObject {
  string knightId; string displayName; string callsign;
  ArchetypeData archetype; TarotCard[] drawnCards;
  TarotCard[] activeAdvantages; // 2 selected
  TarotCard activeDisadvantage; // 1 selected
  HautFaitData hautFait; BlasonData blason;
  Motivation[] minorMotivations; // 3 entries
  Motivation majorMotivation;   // nullable

  // Aspects (mutable)  int chair,bete,machine,dame,masque; // 0–9
  // 15 Characteristics indexed by CharacteristicId enum
  int[15] characteristics;

  // Meta-Armor (immutable after assignment)
  ArmorClass armorClass; // Warrior/Paladin/Priest/Rogue/...
  int maxPA, maxPE, baseCdF;
  int[15] odLevels; // OD per characteristic, max 5 (Types can exceed)
  SlotCounts slotCounts;

  // Runtime Resources (mutable)
  int currentPS, maxPS; int currentPA; int currentPE;
  int currentPEs, maxPEs; int heroisme; // 0–6
  int currentCdF;

  // Combat State
  int position; // 1–4
  CombatStyle activeStyle;
  bool isInAgony, isInFoldState, isDespair;
  bool isGhostActive; // [v2] Rogue Ghost state
  int activeWarriorTypeIndex; // [v2] nullable, Warrior only
  List<InjuryResult> activeInjuries;
  List<StatusEffect> activeStatusEffects;

  // Equipment
  Weapon[] equippedWeapons; int activeWeaponIndex;
  int activeProfileIndex;
  Module[] equippedModules; // [v8] Modules OUT OF DEMO SCOPE
  NodInventory nods;
  bool isDead;
}
```

### Derived Value Formulas

| Derived Value | Formula | Recalculate When |
| --- | --- | --- |
| Defense | Max(hargne,combat,instinct) + that char’s OD | BÊTE char/OD change or Type change |
| Reaction | Max(tir,savoir,technique) + that char’s OD | MACHINE char/OD change or Type change |
| Initiative | Max(discretion,dexterite,perception) + that char’s OD | MASQUE char/OD change or Type change |
| maxPS | 10 + 6 × Max(force,endurance,deplacement) [OD excluded] | CHAIR char changes |
| maxPEs | 50 ± Tarot + Major Motivation | Creation; Major Motivation runtime |
| Points de Contact | Max(aura,parole,sangFroid) [OD excluded]. [v8.2 M-5 NOTE] Computed and stored but NOT mechanically used in demo. Future full-game design: Points de Contact will grant pre-mission bonuses at Camelot (e.g. experimental weapon = bonus damage for 1 mission, reinforced force field, tactical reinforcements). Requires dedicated design pass for full game. | DAME char changes |


[v7] RESOLVED: [v7] [v8.5 C-1 MERGED] Defense = highest Bête Characteristic score + that Characteristic's OD level. Reaction = highest Machine Characteristic score + that Characteristic's OD level. Initiative = highest Masque Characteristic score + that Characteristic's OD level. Pattern: pick the single highest-scoring Characteristic within the relevant Aspect, then add only that Characteristic's OD. Do NOT sum all Aspect ODs. Pseudocode: int Defense => highestCharIn(BETE) + odOf(highestCharIn(BETE)); int Reaction => highestCharIn(MACHINE) + odOf(highestCharIn(MACHINE)); int Initiative => highestCharIn(MASQUE) + odOf(highestCharIn(MASQUE)); [v8.7 C-1 FIX] maxPS = 10 + 6 × Max(Force, Endurance, Déplacement) [OD excluded]. Recalculate whenever any Chair Characteristic changes (injury, buff, debuff). Pseudocode: int maxPS => 10 + 6 * highestCharIn(CHAIR); maxPA = armor base PA (SPEC-02.7). maxPE = armor base PE (SPEC-02.7). baseCdF = armor base CdF (SPEC-02.7). maxPEs = 50 (base, modified by Tarot advantages: Forteresse Spirituelle +5, implants −3 each).
### Acceptance Criteria
KnightBase with all chars at 1, aspects at 2: correct derived values.
[v8.7 M-1 FIX] Injury reducing a Chair Characteristic recalculates maxPS immediately. Injury reducing a Bête Characteristic recalculates Defense immediately. Injury reducing a Machine Characteristic recalculates Reaction immediately. Injury reducing a Masque Characteristic recalculates Initiative immediately.
Characteristics floor at 0.
Major Motivation: maxPEs +25, currentPEs +25 independently.
SPEC-01-AC1: Knight with Bête chars Combat 5 (OD 2), Instinct 3 (OD 0), Hargne 4 (OD 1): Defense = 5 + 2 = 7 (Combat is highest).
SPEC-01-AC2: Same knight, Machine chars Tir 4 (OD 1), Savoir 2 (OD 0), Technique 3 (OD 0): Reaction = 4 + 1 = 5 (Tir is highest).
SPEC-01-AC3: Warrior Type Hunter active (+1 OD to Combat, Instinct, Hargne): Combat OD becomes 3, Defense recalculates to 5 + 3 = 8.
SPEC-01-AC4: Characteristic at 6, parent Aspect at 5: validation error. Characteristic capped at Aspect score.
[v8.5] Aspect/Characteristic Reference: Chair (Body): Déplacement, Force, Endurance. Bête (Beast): Combat, Instinct, Hargne. Machine: Tir, Savoir, Technique. Dame (Lady): Aura, Parole, Sang-Froid. Masque (Mask): Discrétion, Dextérité, Perception. Constraint: Each Characteristic score ≤ its parent Aspect score. Base: Aspects = 2, Characteristics = 1 (before generation bonuses).

## SPEC-02: Knight Generation Pipeline
### Purpose
Procedurally generates 8 demo knights at campaign start. [v4] Player selects 4 for each mission.
### Slot Allocation
[v3] RESOLVED (C5): 2 of each demo class. 8 knights total. [v4] 4 deployed per mission (DD rank system).

| Slot | Armor | Primary Aspects |
| --- | --- | --- |
| 1–2 | Warrior | BÊTE + CHAIR |
| 3–4 | Paladin | CHAIR + MACHINE |
| 5–6 | Priest | MACHINE + DAME |
| 7–8 | Rogue | MASQUE + BÊTE |


### [v4] Mission Squad Selection
Before each mission, the player selects 4 knights from the 8-knight roster to deploy. This selection is final for the entire mission run — no swaps mid-mission. Remaining 4 stay at Camelot, retaining their state (injuries, resources) for future missions.
Roster view: Show all 8 knights with current PS/PA/PE, active injuries, armor class, abilities.
Injured/dead knights: Injured knights are selectable (with warning). Dead knights are greyed out permanently.
Balance suggestion: UI hints at balanced composition (e.g., at least 1 melee, 1 support).
Lock: Player confirms 4. Proceed to first encounter deployment (SPEC-03).

### Algorithm (10 Steps)
Base: All Aspects = 2, all Characteristics = 1.
Archetype: Random from 16. Apply +1 to two Characteristics.
Tarot: [v6] Draw 5 from full 22-card Major Arcana (3× weight for primary aspects). Per card: +1 Aspect, distribute 3 Characteristic pts among the card’s linked Aspect characteristics ([v8] using the Armor Class Weighted Distribution Table below — each point rolled independently against class weights, capped at parent Aspect score). From 5 drawn: select 2 advantages, 1 disadvantage. All 5 cards apply stat bonuses. [v8] Incompatibility check: if both L’Arcane sans Nom (XIII) and La Maison-Dieu (XVI) are drawn, redraw the second one. See SPEC-24.1.
Haut Fait:
[v8.8 L-2 FIX] SPEC-02.8 Data Model (standalone for agent extraction): struct HautFaitData { string hautFaitId; string displayName; string callsign; AspectId bonusAspect; int bonusCharPoints = 2; ConditionType conditionType; // ASPECT = check Aspect score, EITHER_CHAR = check either condChar1 or condChar2. CharacteristicId condChar1; CharacteristicId condChar2; // nullable (single-condition entries). int condMinScore = 4; } enum ConditionType { ASPECT, EITHER_CHAR }
Blason: [v8.3 H-1 FIX] [v8.4 L-1 FIX] Random from 9 demo Blasons (see SPEC-02.5 Blason Table below). Voeu = 3rd minor Motivation. Voeu event tag must not duplicate the knight's 2 drawn MN templates (if collision, re-draw Blason). Le Cheval and Le Corbeau are deferred to full game.
Motivations: [v8.2 M-3 FIX] 1 major + 2 minor randomly drawn from template pools (see SPEC-12.1 Motivation Templates). 3rd minor = Blason Voeu. No duplicates within a single knight. Algorithm: draw 2 unique minor templates from pool of 10, draw 1 major template from pool of 5. Each template carries an eventTag string used by SPEC-12 event matching.
Meta-Armor: [v8.2 CR-2 FIX] Apply armor base stats from table below. Also assign starting Types/abilities per SPEC-17.
[v8.2] SPEC-02.7 — Meta-Armor Base Stats Table (Source: Knight Livre de Base, knight-jdr-systeme.fr)

| Armor | PA | PE | CdF | Gen. | Slots (6 zones) | Base Overdrives |
| --- | --- | --- | --- | --- | --- | --- |
| Warrior | 100 | 40 | 8 | 1st | 7/10/10/12/7/7 | Déplacement, Combat, Tir, Dextérité |
| Paladin | 120 | 20 | 8 | 1st | 7/7/7/10/7/7 | Force, Endurance, Tir, Perception |
| Priest | 70 | 60 | 10 | 1st | 5/5/5/8/5/5 | Force, Endurance, Savoir, Technique |
| Rogue | 50 | 70 | 12 | 2nd | 5/5/5/8/5/5 | Déplacement, Combat, Discrétion, Dextérité |

Slots = Head/Torso/ArmL/ArmR/LegL/LegR. Guardian Suit (Fold state): PA 5, CdF 5, no PE, no OD, no modules. Set knight.maxPA = armor PA, knight.maxPE = armor PE, knight.baseCdF = armor CdF. Initialize currentPA = maxPA, currentPE = maxPE, currentCdF = baseCdF.
Sections: Skip (null).
Derived Values: Compute per SPEC-01.
Equipment: Standard kit + default weapons/modules.

### [v8] Armor Class Weighted Distribution Table (M-1)
When distributing Tarot/Haut Fait Characteristic points during procedural generation, each point is assigned independently using a weighted random roll. Only the 3 characteristics linked to the drawn card’s Aspect participate. Their weights are normalized into probabilities, then one is selected per point. Weight tiers: Specialized (★7), Primary (5), Secondary (3), Tertiary (1). All classes sum to 43 for balance. Characteristic scores are capped at their parent Aspect score — if a characteristic is already at cap, it is excluded from the eligible pool and its weight is redistributed among remaining candidates. If all 3 are capped, remaining points are discarded.

| Characteristic | Aspect | Warrior | Rogue | Paladin | Priest | Ranger | Tier Key |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Déplacement | Chair | 3 | 5 | 1 | 1 | 1 |  |
| Force | Chair | 5 | 1 | 3 | 1 | 1 |  |
| Endurance | Chair | ★7 | 1 | 5 | 1 | 1 |  |
| Combat | Bête | 5 | 5 | 3 | 1 | 1 |  |
| Instinct | Bête | 3 | 3 | 1 | 1 | 5 |  |
| Hargne | Bête | 5 | 1 | 1 | 1 | 1 |  |
| Tir | Machine | 3 | 5 | 5 | 5 | ★7 |  |
| Savoir | Machine | 1 | 1 | 1 | 5 | 5 |  |
| Technique | Machine | 1 | 1 | 3 | ★7 | 3 |  |
| Aura | Dame | 1 | 1 | 5 | 5 | 1 |  |
| Parole | Dame | 1 | 1 | 3 | 3 | 1 |  |
| Sang-Froid | Dame | 5 | 3 | 5 | 5 | 5 |  |
| Discrétion | Masque | 1 | ★7 | 1 | 1 | 3 |  |
| Dextérité | Masque | 1 | 5 | 1 | 3 | 3 |  |
| Perception | Masque | 1 | 3 | 5 | 3 | 5 |  |
| Total |  | 43 | 43 | 43 | 43 | 43 |  |


Tier key: ★7 = Specialized, bold = Primary (5), plain = Secondary (3), 1 = Tertiary. Paladin has no Specialized tier but compensates with 5 Primaries and 4 Secondaries. Priest specializes in Technique (★7) for NanoC construction, with strong Dame support (Aura/Sang-Froid at 5). [v8.2 L-3 NOTE] Ranger column is intentionally preserved as the NEXT PLANNED EXPANSION CLASS (post-demo). Stats are canonical (PA 50, PE 70, CdF 12, Source: Livre de Base). Generation weights are pre-defined to accelerate implementation when the class is added. Ranger is excluded from demo generation pool — do not select during SPEC-02 step 1.

[v8] Distribution Algorithm (pseudocode):
```python
function distributeCardPoints(armorClass, card, numPoints, knight):
  aspect = card.linkedAspect
  linkedChars = getCharacteristicsForAspect(aspect) // always 3
  weights = [WEIGHT_TABLE[armorClass][c] for c in linkedChars]
  for i in range(numPoints):
    // Filter out chars already at Aspect cap
    eligible = [(c, w) for c, w in zip(linkedChars, weights)
                if knight.chars[c] < knight.aspects[aspect]]
    if len(eligible) == 0: break  // all capped, discard point
    totalW = sum(w for _, w in eligible)
    roll = random(0, totalW)
    selected = weightedSelect(eligible, roll)
    knight.chars[selected] += 1
```

[v8] Worked Example:


### Acceptance Criteria
8 distinct knights. No Characteristic exceeds parent Aspect.
Each has 2 advantages, 1 disadvantage, 3 minor + 1 major Motivation.
Same seed = identical results.

### [v8.3 H-1] SPEC-02.5 — Blason System (Source: Knight Livre de Base, knight-jdr-systeme.fr/fr/crest/)
Each knight chooses a Blason at creation — an animal emblem displayed on the meta-armor (shoulder or torso). The Blason defines the knight’s moral identity and provides a Vœu (vow): an additional minor Motivation (the 3rd) with its own event tag. In the tabletop, Blasons are purely roleplay. For the video game, each Vœu is mapped to a concrete event trigger following the same SPEC-12 event bus system as MN templates. Vœu motivations are minor (+1D6 PEs per trigger, repeatable). Le Cheval and Le Corbeau are deferred to full game (relationship system and investigation system respectively).
Generation: [v8.4 L-1 FIX] Random from 9 demo Blasons. Uniqueness rule: the Vœu event tag must NOT duplicate either of the knight’s 2 drawn MN minor motivation tags. If collision, re-draw Blason (max 10 retries, then pick first non-colliding).
Demo Blason Table (10 Blasons):
[1] L’Aigle — «Les Justes» — Tag: VOEU_AIGLE — Vœu: Faire respecter le code d’honneur du Knight lorsqu’il semble bafoue. — Trigger: knight stabilizes a Despaired ally (SPEC-07 Stabilization). The Aigle knight enforces honor when a comrade falters.
[2] L’Ours — «Les Protecteurs» — Tag: VOEU_OURS — Vœu: Empêcher la mort d’un être humain. — Trigger: knight heals an ally who is in Agony (PS = 0) back to PS ≥ 1 via Nod Soin or Priest Mechanic. The Ours knight prevents death directly.
[3] Le Cerf — «Les Déférents» — Tag: VOEU_CERF — Vœu: Obéir à la lettre aux ordres des chevaliers et d’Arthur. — Trigger: knight does not change rank position during an entire combat node (Track: set neverMovedFlag = true at deployment. Set to false if knight ever occupies a rank ≠ deploymentRank during the node. Checked at node completion: triggers only if neverMovedFlag is still true). Represents disciplined obedience — staying at your assigned post. Triggers once per qualifying combat node.
[4] Le Dragon — «Les Droits» — [v8.4 L-1 FIX] DEFERRED TO FULL GAME. Removed from demo Blason rolling table. The Dragon Vœu ("no Failure Critique in combat") requires clearer scoping of which roll types count. Will be redesigned when the relationship system and broader roll tracking are implemented.
[5] Le Faucon — «Les Guerriers» — Tag: VOEU_FAUCON — Vœu: Pourfendre, sans aide, un salopard ou un patron de l’Anathème. — Trigger: knight deals the killing blow to a T3+ enemy (Salopard, Colosse, or Patron) AND no Assist was used on the killing attack roll. Solo elite kill. Triggers once per qualifying kill.
[6] Le Lion — «Les Courageux» — Tag: VOEU_LION — Vœu: Ne jamais fuir devant un ennemi s’il met des innocents en danger. — Trigger: knight deployed at Rank 1 (front) survives an entire combat node without ever moving to a higher rank (Rank 2, 3, or 4). Holding the line. Track: flag set to false if knight ever occupies Rank 2+ during combat. Checked at node completion. Triggers once per qualifying node.
[7] Le Loup — «Les Solidaires» — Tag: VOEU_LOUP — Vœu: Protéger un chevalier alors qu’il est en danger. — Trigger: knight uses Nod d’Armure or Priest Mechanic on an ally who is at ≤25% maxPA at the time of healing. Pack loyalty — supporting the wounded. Triggers once per qualifying heal action.
[8] Le Sanglier — «Les Persévérants» — Tag: VOEU_SANGLIER — Vœu: Accomplir les objectifs coûte que coûte. — Trigger: knight is alive at mission completion AND has at least one of: ≥1 active injury, entered Fold state during mission, or current PEs ≤ 10. Perseverance through adversity — only triggers if the knight was actually tested. Checked at OnMissionComplete. If the knight is unscathed (no injury, never folded, PEs > 10), the Vœu does NOT trigger — there was no adversity to overcome.
[9] Le Serpent — «Les Savants» — Tag: VOEU_SERPENT — Vœu: Découvrir les secrets de l’Anathème. — Trigger: knight is the first party member to attack a previously un-attacked enemy type during the mission. Track per-mission per enemy type (e.g., first knight to hit a Bestian, first to hit a Faune, etc.). Scouting the unknown. Triggers once per new enemy type encountered. In the demo mission with 4 enemy types, up to 4 possible triggers per mission.
[10] Le Taureau — «Les Loyaux» — Tag: VOEU_TAUREAU — Vœu: Respecter, à chaque mission, le code d’honneur du Knight. — Trigger: knight completes a mission without entering Despair AND without any allied knight dying during the mission. Full squad discipline upheld. Strictest Vœu — requires zero deaths and personal morale control. Checked at OnMissionComplete. Triggers once per qualifying mission.
Deferred Blasons (full game only): Le Cheval (loyalty/relationship system required), Le Corbeau (investigation/discovery system required), Le Dragon (roll scope clarification required — see v8.4 L-1). These are canonical tabletop Blasons and will be implemented when their dependent systems are designed.
Data Model: struct BlasonData { string blasonId; string displayName; string animalIcon; string voeuText; string voeuEventTag; } — Stored on KnightBase.blason. The Vœu motivation is stored as the 3rd entry in KnightBase.minorMotivations[] with type = VOEU and the Blason’s event tag.

## SPEC-03: Position System (Darkest Dungeon Ranks)
[v8] Global Duration Definition: Throughout this document, “Duration: 1 turn” means the effect persists from the moment of activation until the start of the activating combatant’s next turn. The effect is active during all other combatants’ turns in between (including enemy turns). This applies globally to all duration references: Warrior Type OD bonus, Ghost stealth, Shrine, NanoC effects, Barrage Def/React reduction, and any other “1 turn” duration.
[v8] Global Math Rules (Tabletop Canon):
### Purpose
[v4] MAJOR REWRITE. Manages P1–P4 linear rank track with Darkest Dungeon-style single-occupancy positioning. Each rank holds exactly one combatant.
### Track Layout
KNIGHT SIDE                    ENEMY SIDE
[Rank 4] [Rank 3] [Rank 2] [Rank 1]  |  [Rank 1] [Rank 2] [Rank 3] [Rank 4]
  Back     Mid-B    Mid-F    Front   |   Front    Mid-F    Mid-B     Back

### [v4] Core Rank Rules
1 combatant per rank. No exceptions. 4 ranks per side = max 4 combatants per side.
Knight squad: 4 knights selected from 8-knight roster at mission start. No changes mid-mission. Remaining 4 stay at Camelot.
Enemy cap: Max 4 individual enemies on-track. Reinforcements only spawn into empty ranks.
[v2] Bandes are OFF-TRACK. They have no rank. Débordement hits all knights regardless.
Gap formula:


### [v4] Range Band to Max Gap

| Range Band | Max Gap | Weapon Examples |
| --- | --- | --- |
| Contact | 1 | All melee weapons |
| Courte | 2 | Short-range firearms, Nods, modules |
| Moyenne | 4 | Standard firearms |
| Longue | 5 | Rifles |
| Lointaine | 6 | Sniper rifles |


### Rank Modifiers

| Rank | Defense | Reaction | Notes |
| --- | --- | --- | --- |
| Rank 1 (Front) | +1 | — | Frontline. Melee access. |
| Rank 2 (Mid-F) | — | — | Neutral. |
| Rank 3 (Mid-B) | — | — | Neutral. Ranged focus. |
| Rank 4 (Back) |  |  | Exposed. Backline. |


### [v6] Cover System
[v6] NEW — Critical C3 resolution. Cover represents terrain features (walls, barricades, debris) that provide protection against ranged attacks.

Cover Flag:
Each rank on either side can have a hasCover: boolean flag, set by encounter data (EnemySpawn ScriptableObject) or generated by NanoC Cover Wall ability (SPEC-17).
Encounter designers specify which ranks start with cover via the encounter definition (SPEC-22).


Cover Bonuses & Penalties:
A combatant in a Cover rank gains +3 Reaction against all incoming ranged attacks.
A combatant attacking with a ranged weapon from a Cover rank suffers  on their attack roll (restricted firing angles).
Cover has no effect on melee (contact) attacks — Defense is unaffected.
Cover bonuses stack with Rank Modifiers (e.g., Rank 1 Cover = +3 Reaction from cover, +0 from rank = +3 total Reaction bonus).

Weapon Effect Interactions:
Tir en Sécurité: Knight firing from a Cover rank does NOT
Artillerie:
Dispersion: Cover does not protect against Dispersion damage if the combatant is hit as a secondary target.

Enemy Cover:

Enemy AI with RankBehavior HOLD will prefer staying in cover. ADVANCE may leave cover to close distance.

### [v4] Movement — DD Rank Swap System
Movement in the rank system works like Darkest Dungeon: knights swap positions with adjacent allies or shift into empty slots.

Move into empty adjacent rank: 1 Movement Action. Knight shifts one rank forward or back into empty slot.
Swap with adjacent ally: 1 Movement Action. Knight and ally exchange ranks simultaneously. No action cost for the swapped ally.
Move 2 ranks: Forfeit Combat Action for Movement. Can chain two single-rank moves or swaps.
Cannot pass through occupied enemy ranks. Only ally swaps and empty slots.

### [v4] Forced Displacement
Forced displacement pushes target 1+ ranks toward their back line.
Chain-swap rule: If target is pushed into an occupied ally rank, those two swap. The displaced ally is NOT further pushed.
Push into empty rank: Target simply moves. No chain.
Push past Rank 4: Clamped to Rank 4. No further effect.
Colonne Vertébrale Brisée: [v8] Spine broken — character loses use of legs, armor handles movement. Movement action costs 2 actions (any combination of movement + combat actions) instead of 1. In the demo (1 movement + 1 combat per turn), this means a position shift costs the entire turn. Future content may grant additional actions via effects/capacities, so implement as doubled action cost, not “whole turn.” Cannot Sprint. Forced displacement still applies normally (armor absorbs the push).
Jambe Mutilée: 1 rank of movement costs 2 Movement Actions.

### [v6] Death and Empty Ranks
When a combatant dies, their rank becomes empty.
Adjacent combatants may shift into the empty rank on their turn (1 Movement Action).
Enemy reinforcements spawn into empty enemy ranks only. If all 4 enemy ranks occupied, reinforcements are queued. [v8] Queue check: triggers at the start of the enemy phase, before any enemy acts. Queued enemies spawn into ranks vacated by deaths in the preceding knight phase. Freshly spawned reinforcements CAN act on the turn they spawn (they have initiative in the current round). Spawn priority: highest-tier enemy in queue spawns first. If tied, random.
Knight death leaves rank empty permanently for that encounter (no mid-mission roster swap).

### [v4] Weapon Rank Targeting
Weapons have rank restrictions for both the user and the target, derived from range bands:

| Weapon Type | Usable From (Knight Ranks) | Can Target (Enemy Ranks) |
| --- | --- | --- |
|  | Rank 1 only | Rank 1 only |
|  | Rank 1–2 | Rank 1–2 |
|  | Rank 1–4 (all) | Rank 1–4 (all) |
|  | Rank 1–4 (all) | Rank 1–4 (all) |
|  | Rank 2–4 | Rank 1–4 (all) |

Note: Contact weapons at Rank 1 can only hit enemy Rank 1. A Warrior at Rank 2 cannot melee — must swap to Rank 1 first.

### [v4] Squad Selection (Mission Start)
Before each mission, the player selects 4 knights from the 8-knight roster. Selection is final for the entire mission run. Remaining 4 knights are unavailable but retain their state (injuries, resources) for future missions.
Selection screen: Show all 8 knights with current PS, PA, PE, injuries, abilities.
Composition guidance: UI suggests balanced team (1 melee, 1 tank, 1 support, 1 flex).
Lock: Player confirms. Selected 4 proceed to deployment phase.

### [v3] Deployment Phase (C3 Resolved)
Before the first turn of each encounter, the player places the 4 selected knights on Ranks 1–4.
Enemy placement: Ranks loaded from encounter data (EnemySpawn ScriptableObject).
Knight placement: Player assigns each knight to a rank. 1 knight per rank. All 4 must be placed.
Default suggestion: UI pre-places Warrior at Rank 1, Paladin at Rank 2, Priest at Rank 3, Rogue at Rank 4.
Confirm: Player presses ‘Deploy.’ Initiative rolls. Combat begins.

### Acceptance Criteria
[v4] Rank 1 vs Rank 1: Gap=0. Contact weapon succeeds.
[v4] Rank 3 vs Rank 1: Gap=2+0=2. Contact (maxGap 1) rejected. Courte (maxGap 2) succeeds.
[v4] Knight at Rank 2 moves to empty Rank 1: 1 Movement Action. Knight now at Rank 1.
[v4] Knight at Rank 1 swaps with ally at Rank 2: both exchange. 1 Movement Action for initiator only.
[v4] Enemy pushes knight from Rank 1 to Rank 2 (occupied by ally): knight and ally swap. Ally now at Rank 1.
[v4] Knight dies at Rank 2: rank becomes empty. No replacement available mid-mission.
[v4] 4 enemy ranks full, reinforcement queued: spawns when next enemy dies.
Colonne Vertébrale Brisée: movement costs 2 actions, forced displacement still applies.


[v6] Artillerie weapon vs enemy in Cover rank: ignore +3 Reaction bonus. Enemy Reaction = base value.

## SPEC-04: Combo Roll System
### Purpose
Resolves all attacks, abilities, and contested actions.
### Roll Procedure
Base Characteristic: Set by action type (Combat for melee, Tir for ranged, etc.).
[v2] Choose Combo Characteristic: Player selects from any Characteristic EXCEPT the Base. Choice is per-roll — may change each action. Must differ from Base.
Pool size: Base Characteristic score (dice) + Combo Characteristic score (dice) + style mods + PEs penalty + injury penalties.
PEs penalty:
Roll D6s equal to pool size.
Count successes: Even results (2, 4, 6) = 1 success each.
Add OD auto-successes: Base Characteristic’s OD level + Combo Characteristic’s OD level. These are flat successes added after the roll, not extra dice.
Special outcomes:
Compare: [v7] Successes > Defense (melee) or Reaction (ranged) = hit. Must strictly exceed, not equal.

### Style Modifiers

| Style | Attack Pool | Defense/Reaction Mod |
| --- | --- | --- |
| Standard | — | — |
| Agressif | +3 dice |  |
| Défensif |  | +2 Def, +2 React |
| Mise à couvert |  | +2 Reaction (ranged) |
| Puissant |  |  |
| Pilonnage |  | — |
| Précis | +3rd Characteristic | Costs Combat + Movement Action |


### Special Outcomes

| Outcome | Condition | Effect |
| --- | --- | --- |
| Exploit |  | Re-roll pool, add new successes. +1 Héroïsme, +1 PEs. |
| Failure Critique |  |  |
| Assistance | Up to 3 allies Combo Roll | [v4] From any rank (no proximity required). Each spends 1 Movement Action. Successes added. Characteristic used must not duplicate Base, Combo, or another assistant's choice. |


### Edge Cases
Pool = 0: no dice, no Exploit/Failure Critique. OD auto-successes still apply.
Précis: 3rd Characteristic’s OD also adds auto-successes.
Exploit on attack: re-roll successes count for threshold AND damage. [v8] Clarification: on Exploit, re-roll the entire dice pool. New successes from the re-roll are ADDED to the original successes (not replacing). The combined total is used for: (a) hit threshold comparison, and (b) excess successes beyond threshold become bonus damage dice (same as Mode Héroïque excess). OD auto-successes are added once (not re-applied on re-roll).
[v8] Exploit/Failure Critique is determined solely by dice faces, not OD auto-successes. OD is added AFTER checking for special outcomes.
[v8] Exploit that still misses after re-roll + OD: no damage dealt, but Héroïsme and PEs bonuses are still awarded.

[v8] Pool = 1 die: yes, 50% of single-die rolls trigger special outcomes. This is intended — small pools are inherently volatile.

### [v4] Dual-Wielding Rules (L3 Resolved)
Attacking with two weapons simultaneously:  to attack pool.
Jumelé (Akimbo) weapon effect: reduces penalty to 
Ambidextre module: removes penalty entirely (0 penalty).
Both weapons resolve against the same target. Damage rolled separately for each weapon.
Incompatible with Paladin Watchtower (which grants extra ranged shots, not dual-wield).

### [v4] Chargeur & Ammo Rules (L4 Resolved)
Weapons with Chargeur X: X uses per mission. Each attack consumes 1 charge.
No mid-mission reload. When charges reach 0, weapon cannot fire until Camelot return.
Auto-refill at Camelot between missions. No PG cost.
Grenades: Fixed 5 per mission per knight. Same auto-refill rule. Choose type before each throw.
Nods: Quantity set per mission (default 3 of each type per knight). Auto-refill at Camelot.
### Acceptance Criteria
Combat 4 + Perception 3, Standard, PEs 50: rolls 7D6.
Same with PEs 7 (penalty 3): rolls 4D6.

All even: Exploit. Zero even: Failure Critique.

### [v4] Action Economy Reference (M2, M5, M6 Resolved)
Consolidated action cost table for common combat activities. [v4] Updated for DD rank system.

| Action | Cost | Notes |
| --- | --- | --- |
| Attack (melee or ranged) | 1 Combat Action | Combo Roll required. Rank targeting applies (see SPEC-03). |
| [v4] Move into adjacent empty rank | 1 Movement Action | Shift 1 rank forward/back into empty slot. |
| [v4] Swap with adjacent ally | 1 Movement Action | Both knights exchange ranks. No cost for swapped ally. |
| [v4] Move 2 ranks | Forfeit Combat Action | Chain two single-rank moves/swaps. |
| Use a Nod (self) | 1 Movement Action | Restores 3D6 of PS/PA/PE. |
| [v4] Use a Nod (on ally) | 1 Movement Action |  |
| Assist an ally | 1 Movement Action | From any rank. Roll 1 unique Characteristic. Max 3 assistants. |
| Switch equipped weapon | Free Action | Holster current, draw another. |
| Switch weapon profile | 1 Movement Action |  |
| Throw grenade | 1 Combat Action | Base Tir. Courte range (+Force OD extends). 5/mission. |
| Activate Warrior Type | 1 Movement Action | 1 PE. Max 1 switch/turn. |
| Activate Paladin Shrine | Free Action | 1–2 PE depending on range. |
| Activate Watchtower | 1 Movement Action | 2 PE. |
| Activate Rogue Ghost | Free Action | 2 PE/turn. |
| Use NanoC (Simple) | 1 Movement Action | 3 PE. |
| Use NanoC (Detailed) | 1 Combat Action | 6 PE. |
| Use NanoC (Mechanical) | Full Turn (both actions) | 9 PE. |
| Use Priest Mechanic | 1 Movement Action | 4 PE. Contact or Long range. |
| Delay initiative | Free (start of turn) | Announce lower initiative value. |
| Deactivate Watchtower | Free Action | Restores mobility. |
| Switch combat style | Free Action | Once per turn. |


## SPEC-05: Damage Resolver
### Purpose
Computes final damage from a hit and routes through armor layers. [v2] Violence pathway to PEs removed. [v3] Canonical damage formula established (C1/C2 resolved).

### [v3] CRITICAL DISTINCTION: Attack Roll vs Damage Roll
These are two completely separate systems. The MDD previously conflated them. The tabletop is unambiguous:


|  | Attack Roll (To-Hit) | Damage Roll (On Hit) |
| --- | --- | --- |
| Purpose | Determine if attack connects | Determine how much damage |
| Dice pool | Base + Combo + mods | Weapon damage dice (from profile) |
| Counting method | Count EVENS = successes | SUM ALL FACE VALUES |
| Threshold | [v7] Successes > Defense (melee) or Reaction (ranged) | N/A — no threshold |
| Excess successes | Do nothing by default | N/A |
| OD contribution | Auto-successes added to hit | Flat damage bonus (+3/OD for Force melee, +1/OD for Tir Précision etc.) |


### [v3] Canonical Damage Chain (8 Steps)
Hit determination: [v7] Attack Roll successes > threshold (strictly greater). Miss = no damage. Stop.
Roll weapon damage dice: Per weapon profile (e.g. 3D6, 5D6+10). SUM all face values. This is the base damage.
Add weapon flat bonus: If profile includes +N (e.g. Pistolet 2D6+6), add it.
Add Force bonus (melee only): +Force score. +3 per Force OD level. Lesté weapons: Force ×2 instead of ×1.
Add effect bonuses: Précision: +Tir + 1/OD. Orfèvrerie: +Dextérité + 1/OD. Silencieux (from stealth): +Discrétion + OD. Ghost bonus (Rogue): +Discrétion + OD.
Puissant style bonus: [v8.7 M-5 FIX] Knight sacrifices N dice (1-6) from attack pool → gains N dice added to damage/violence roll. −2 Defense, −2 Reaction. Restricted to contact weapons with Lourd property. Roll sacrificed dice, SUM faces, add to damage total. [v8.8 M-3 NOTE] Demo weapon availability: Puissant requires a Lourd Contact weapon — no demo-scope weapon has the Lourd property; Puissant is scaffolded for future weapons. Pilonnage requires a Deux Mains Ranged weapon — available with Fusil de Précision, Fusil d'Assaut, and Shotgun Escamotable.
[v8.8 L-3 FIX] Mise à couvert style: Knight takes cover behind terrain features. −3 attack dice, +2 Reaction against ranged attacks only. Defense (melee) unaffected. Interactions: +2 Reaction bonus applies to Dispersion splash if the incoming attack is ranged. Does NOT apply to Débordement (automatic damage, no attack roll). Stacks with SPEC-03 Cover System bonuses (+3 Reaction from cover rank + +2 from Mise à couvert = +5 total Reaction vs ranged). Does not stack with Défensif (+2 Def, +2 React) — knight must choose one style per turn.
Dégâts Maximum (1 Héroïsme): Replace step 2 — all weapon dice = 6 (maximum). Then add all bonuses normally.
Route total raw damage through ArmourLayerResolver (SPEC-06).

### [v3] Excess Successes Rule
By default, excess successes on the attack roll have NO damage effect. They are discarded after hit confirmation.
Exceptions:
Assistance à l'Attaque weapon effect: +1 flat damage per excess success above threshold.
Mode Héroïque: Excess successes = extra D6 damage dice (roll and SUM).
[v8] Exploit (all dice even): Re-roll entire pool, ADD new successes to original. Combined excess above threshold = extra D6 damage dice (roll and SUM). See SPEC-04 Edge Cases for full Exploit rules.
No other source converts excess successes to damage.

### [v3] Worked Example — Melee Attack
[v7] Knight with Combat 4, Instinct 3, Force 5, Force OD 1 uses Marteau-Épieu (3D6, Perce Armure 40) in Standard style vs enemy Defense 4. NOTE: This example also demonstrates Lesté mechanics for reference — Marteau-Épieu itself is NOT Lesté. Force bonus: Force×1 + 3/OD = 5 + 3 = 8.
Attack pool: 4+3 = 7D6. Rolls: [1,2,3,4,5,6,2]. Evens: 2,4,6,2 = 4 successes + 1 OD auto = 5. 5 > 4 = HIT.
Damage dice: 3D6. Rolls: [3,5,4]. SUM = 12.
[v7] Force bonus (NOT Lesté — Force×1): [v7] Force×1 = 5. Force OD×3 = 3. Total bonus = 8. (If Lesté: Force×2 = 10 + 3 = 13.)
Total raw damage: 12 + 8 = 20. Route through ArmourLayerResolver.

### [v3] Worked Example — Ranged Attack with Précision
Knight with Tir 4, Perception 3, Tir OD 2 uses Fusil de Précision (4D6+6, Précision) vs enemy Reaction 5.
Attack pool: 4+3 = 7D6. Rolls: [2,2,4,1,6,3,5]. Evens: 2,2,4,6 = 4 successes + 2 OD auto = 6. 6 > 5 = HIT.
Damage dice: 4D6. Rolls: [3,6,2,5]. SUM = 16. +6 flat = 22.
Précision bonus: +Tir 4 + OD 2×1 = +6.
Total raw damage: 22 + 6 = 28. Route through ArmourLayerResolver.

### [v2] Violence Damage Chain
Determine Violence dice from weapon profile.
Roll Violence dice. SUM all face values (same as damage dice).
[v2] Against Bandes ONLY: Each Violence point = 1 Cohesion lost.
[v2] Against individuals: Violence has NO effect. Ignore completely.

### Force Bonus Summary

| Source | Value | Applies To |
| --- | --- | --- |
| Force score | +Force flat damage | Melee only |
| Force OD | +3 per OD level | Melee only |
| Lesté | Force×2 instead of ×1 |  |
| Tir (Précision) | +Tir score + 1/OD | Ranged with Précision effect |
| Dextérité (Orfèvrerie) | +Dext score + 1/OD | Weapons with Orfèvrerie effect |
| Discrétion (Ghost/Silencieux) | +Discrétion + OD | Stealth attacks (see SPEC-17) |

### Acceptance Criteria
[v7] Marteau-Épieu (3D6), rolls [3,5,4], Force 5 OD 1 (NOT Lesté): 12 + 5 + 3 = 20 raw. (If weapon were Lesté: 12 + 10 + 3 = 25 raw.)
[v3] Fusil Précision (4D6+6), rolls [3,6,2,5], Tir 4 OD 2: 16 + 6 + 4 + 2 = 28 raw.
[v3] 6 successes vs Defense 4 with no Assistance: 0 extra damage from excess.

[v3] Dégâts Maximum on 3D6: dice = [6,6,6] = 18. Then add flat bonuses.
[v2] 4D6 Violence vs knight: ignored. Same vs Bande: SUM = Cohesion lost.

## SPEC-06: Armour Layer Resolver
### Purpose
Routes raw damage through defensive layers. [v2] PA passthrough now explicit Step 2b.
### Knight / Human NPC Chain (3-Step + Passthrough)
Step 1 — CdF:
Step 2a — PA absorption: Remaining reduces PA.
[v2] Step 2b — PA Passthrough: For every complete 5 PA lost in this hit, knight also loses 1 PS. Formula: passthroughPS = floor(paAbsorbed / 5). Disabled by Infatigable Tarot advantage. If PA reaches 0: trigger fold (SPEC-10).
Step 3 — PS damage: Remaining after PA depletion hits PS. PS = 0: trigger Agony (SPEC-08).

### Anathème Creature Chain (4-Step)
CdF reduction.
Bouclier reduction (resets each hit).
PA absorption.
PS damage.

### [v8] Knight Defense vs Anathème Attacks (H-1)
Some enemies (Bestian, Ours Corrompu) have Anathème-typed attacks that target knights. [v8] The to-hit procedure is unchanged: the enemy rolls a standard attack using the weapon’s range band (e.g. Moyenne for Flot de ténèbres). Knights oppose with Defense (if contact) or Reaction (if ranged), and standard gap/position rules apply. On miss: no effect. On hit: damage does NOT use the standard Knight Armour Layer chain. Instead, it follows a dedicated 2-step resolution:
Step 1 — CdF: Apply CdF reduction normally using the knight’s total CdF (base + Shrine bonus). This total is subject to IGNORE_CDF and Pénétrant X as usual.
Step 2 — Damage reduces PEs directly: Remaining damage after CdF is subtracted from PEs (not PS). PA is completely bypassed — no PA absorption, no PA Passthrough, no Fold trigger from this damage. If PEs reaches 0, trigger Despair (SPEC-07).
[v8] Pseudocode:
function resolveAnathemeVsKnight(rawDamage, target, tags):
  afterCdF = rawDamage
  if IGNORE_CDF not in tags:
    afterCdF = max(rawDamage - target.CdF, 0)
  // Skip PA entirely — no absorption, no passthrough
  target.PEs = max(target.PEs - afterCdF, 0)
  fire OnPEsChanged(target)
  if target.PEs == 0: triggerDespair(target)  // SPEC-07
[v8] PEs Recovery on Anathème Kill:
Per tabletop: when an enemy with the Anathème capacity is killed, each knight who lost PEs to that enemy’s Anathème attacks recovers 1D6 PEs per complete 6 PEs lost. Track cumulative Anathème PEs damage per knight per source enemy. On enemy death: recoveredPEs = floor(totalAnathemeLost / 6) × roll(1D6). Fire OnPEsChanged. This can exit Despair if PEs rises above 0.

### Tag Overrides

| Tag | Effect |
| --- | --- |
| Ignore CdF | Skip Step 1. |
| Pénétrant X | Reduce CdF by X for this hit. |
| Perce Armure X | Ignore up to X PA. |
| Destructeur | [v6] [v7] If remaining > 0 after CdF+Bouclier AND target has PA: roll +2D6, add to remaining BEFORE PA absorption. See canonical pseudocode below. |
| Meurtrier | [v6] [v7] If damage reaches PS (remaining > 0 after PA fully depleted): +2D6 bonus damage added to remaining before PS reduction. See canonical pseudocode below. |
| Dégâts Continus X | Bypass CdF+Bouclier. X dmg/turn for 1D6 turns. [v8] Tick timing: damage applies at the start of the afflicted combatant’s turn, before they act. DoT damage bypasses CdF and Bouclier but IS absorbed by PA normally. Applies to both knights and enemies. |
| Chair Mineur | Additional flat reduction after Bouclier, before PA. |


[v7] Canonical ArmourLayerResolver Pseudocode (8-Step)
```python
function resolveArmourLayers(rawDamage, weapon, target):
  Step 1: remaining = rawDamage
  Step 2: if weapon.hasTag(IGNORE_CDF):
              pass  // skip CdF entirely
          else:
              effectiveCdF = target.CdF
              if weapon.hasTag(PENETRANT_X): effectiveCdF = max(target.CdF - X, 0)
              remaining -= min(effectiveCdF, remaining)
  Step 3: if target.hasBouclier: remaining -= min(target.Bouclier, remaining)
  Step 4: if target.hasChairMineur: remaining -= min(target.ChairExc, remaining)
  Step 5 — DESTRUCTEUR CHECK:
          if remaining > 0 AND weapon.hasTag(DESTRUCTEUR) AND target.PA > 0:
            destructeurBonus = roll(2, D6)  // Roll 2D6
            remaining += destructeurBonus
  Step 6: if target.PA > 0 AND NOT weapon.hasTag(IGNORE_ARMURE):
            // [v8] Perce Armure: reduce effective PA before absorption
            if weapon.hasTag(PERCE_ARMURE_X):
                effectivePA = max(target.PA - X, 0)
                paAbsorbed = min(effectivePA, remaining)
            else:
                paAbsorbed = min(target.PA, remaining)
            target.PA -= paAbsorbed
            remaining -= paAbsorbed
            passthroughPS = floor(paAbsorbed / 5)  // unless Infatigable
            target.PS -= passthroughPS
            if target.PA == 0: triggerFold(target)  // SPEC-10
  Step 7 — MEURTRIER CHECK:
          if remaining > 0 AND weapon.hasTag(MEURTRIER):
            meurtrierBonus = roll(2, D6)  // Roll 2D6
            remaining += meurtrierBonus
  Step 8: target.PS -= remaining
          if target.PS <= 0: triggerAgony(target)  // SPEC-08
```
[v7] Worked Example — Both Destructeur + Meurtrier on same hit:
Weapon: Mitrailleuse lourde (5D6+10, Destructeur, Perce Armure 20). Target: Chevalier renégat (CdF 10, PA 25, PS 40).
Step 1: rawDamage = 26 (rolled 5D6=16, +10 flat).

Step 3: No Bouclier. Step 4: No Chair Mineur.


Step 7: MEURTRIER — remaining = 0, does NOT trigger (no damage reached PS).
Step 8: remaining = 0, no PS damage beyond passthrough. Final: PA=0 (Fold), PS=35.
### [v2] PA Passthrough Worked Example

### Acceptance Criteria
CdF 8, PA 100, PS 40. 15 raw: CdF absorbs 8, PA takes 7. Passthrough=1. PA=93, PS=39.
Same with Infatigable: PA=93, PS=40.


## SPEC-07: Espoir (PEs) System
### Purpose
Tracks morale. PEs < 10 = dice penalty. PEs = 0 = Despair. [v8.8 M-1 FIX] Canonical penalty formula: pesPenalty = max(0, 10 − currentPEs). Applied as negative modifier to all Combo Roll dice pools (SPEC-04). [v2] No Violence input.
### State Table

| PEs Range | State | Dice Penalty | Additional |
| --- | --- | --- | --- |
| 10–50+ | Résolu | 0 | Full effectiveness. |
| 7–9 | Ébranlé | 1–3 | — |
| 4–6 | Brisé | 4–6 | Armor portrait cracking. |
| 1–3 | Au bord du gouffre | 7–9 | Failure Critique threshold +1. |
| 0 | Désespoir | N/A | Acts against team 1D6 turns. Parole+Aura diff 3 to stabilize. |


### [v2] Loss Sources (Updated)



Narrative event choices (variable per event).
[v2] REMOVED: 'Enemy Violence = PEs loss.' Violence does not affect individual PEs.

### Recovery Sources
Minor Motivation fulfilled: +1D6 PEs (repeatable).
Major Motivation completed: +25 currentPEs and +25 maxPEs.
Exploit: +1 PEs.
No between-mission PEs recovery in demo.
### Acceptance Criteria
PEs 9: penalty=1. PEs 3: penalty=7.



### [v8] Despair Resolution (CR-3 — New Subsection)
When a knight’s PEs reaches 0, the Despair (Désespoir) state activates. This subsection provides the full mechanical specification for agent implementation.
[v8] Despair Trigger:
PEs reaches 0 from any source. Immediate — no action, no roll.

Fire OnDespairEntered(knight, despairDuration) event (SPEC-27).
Ignorer Désespoir (1 Héroïsme, SPEC-09): cancels this Despair trigger entirely. The PEs loss that would have triggered Despair is also prevented — PEs stays at 1 (not 0). Knight acts normally.
[v8] Despair Behavior (Hostile AI):
During Despair, the knight is player-uncontrollable and acts under hostile AI for despairDuration turns:
Target selection: Random allied knight within weapon range. If no ally in range, move toward nearest ally (1 Movement Action), then attack if now in range.
Attack resolution: Standard Combo Roll using the knight’s currently equipped weapon. System auto-selects highest available Base + Combo pair. Roll vs target ally’s Defense (contact) or Reaction (ranged). Damage resolved normally through ally’s ArmourLayerResolver (SPEC-06).
Action economy:
Turn countdown: Decrement despairDuration at end of the Despaired knight’s turn. Display remaining turns on HUD.
[v8] Stabilization (Ally Intervention):
Who:
Cost: 1 Combat Action from the stabilizing knight.
Roll: Combo Roll with Parole as Base + Aura as Combo (or any other valid Combo, but Parole must be Base). Difficulty: Ardue (3 successes required). OD auto-successes apply normally.
Success: Despair ends immediately. Despaired knight’s PEs resets to 1. Player regains control. Stabilizing knight earns +1 Héroïsme (SPEC-09).
Failure: No effect. Another ally may attempt on their turn. Multiple attempts allowed (1 per ally per turn).
[v8] Despair Exit Conditions:
Ally stabilization (above): PEs resets to 1. Immediate.
PEs restoration from any source: If any effect restores PEs above 0 (Motivation trigger, Exploit by another knight on behalf of the squad), Despair ends immediately. Knight regains control.
Duration expires (despairDuration = 0): Despair becomes permanent. Knight remains under hostile AI indefinitely — there is no automatic recovery. The Despaired knight is now treated as an enemy combatant: allies may attack and must defeat them (reduce to PS 0, triggering Agony/Injury per SPEC-08) or stabilize them. Stabilization remains available at any time (see above). A Despaired knight defeated by allies counts as a normal Agony trigger and may survive via Ignorer l’Agonie.
[v8] Edge Cases:
Despair + Agony: If both trigger simultaneously, Agony takes priority (knight is incapacitated). Despair is suppressed while in Agony. If healed out of Agony but PEs is still 0, Despair activates.
Despair + Fold: Both states can be active simultaneously. Fold restrictions apply on top of Despair behavior (no ODs, no abilities, forced Standard style overrides Despair’s forced Agressif — use Standard).
Last knight in Despair: If all surviving knights are simultaneously in Despair or incapacitated, treat as squad wipe (SPEC-22 defeat condition).
Despaired knight kills ally:
[v8] Despair Acceptance Criteria:
[v8] PEs reaches 0: knight enters Despair. 1D6 = 3 turns. Knight attacks random ally on its turn for 3 turns.
[v8] Ally at adjacent rank spends 1 Combat Action, rolls Parole 4 + Aura 3 = 7D6, needs 3 successes: success = Despair ends, PEs = 1.
[v8] Motivation triggers during Despair granting +1D6 PEs: if PEs rises above 0, Despair ends immediately.
[v8] Ignorer Désespoir (1 Héroïsme): Despair trigger cancelled. PEs loss prevented, PEs stays at 1. Knight acts normally.
[v8] Duration expires, not stabilized: Despair becomes permanent. Knight remains hostile AI. Allies must defeat (PS 0) or stabilize to end Despair.


## SPEC-08: Injury & Agony System
### Purpose
Injury Table, Agony state, Hémorragie countdown, permanent injuries.
### Agony Trigger Sequence
PS reaches 0.
Immediately: roll Injury Table.
Apply injury to KnightBase.
Set isInAgony = true.

Disable all action slots.

### Agony State Rules
No actions possible.


Ignorer l’Agonie (1 Héroïsme): stay at 1 PS, no Agony, no Injury. Once/combat.

### Special Injuries
1,1 MORT: Instant permadeath.
1,5 Hémorragie:
1,4 Colonne Brisée: [v8] Movement costs 2 actions (any type). Cannot Sprint.
6,2 Jambe Mutilée: 1 position = 2 Movement Actions.
### Acceptance Criteria
PS 1 takes 5: PS=0, Agony, Injury roll.

Ignorer l’Agonie: PS=1, no Agony. Can’t reuse this combat.

## SPEC-09: Heroism System
### Purpose
Héroïsme points (0–6 per knight per mission).
### Earning

| Source | Amount | Recipient |
| --- | --- | --- |
| Exploit | +1 | Rolling knight |
| Defeat Salopard/Patron | +1 | Killing knight |
| Stabilize Désespéré | +1 | Stabilizing knight |
| Combat with no knight <50% PS | +1 | All knights |


### Spending

| Spend | Cost | Effect | Limit |
| --- | --- | --- | --- |
| Ignorer Agonie | 1 | Stay at 1 PS. No Agony/Injury. | 1/combat/knight |
| Ignorer Désespoir | 1 | Avoid Despair this turn. | Per occurrence |
| Relancer test | 1 | Reroll failed Combo Roll. | Per roll |
| Dégâts Maximum | 1 | Maximize weapon dice. | Per attack |
| Mode Héroïque | 6 | All allies Adjuvants. No PE cost. Extra dmg dice. | Once (all 6) |


### Mode Héroïque
Declare objective. Spend all 6.
Allies auto-contribute successes.
PE cost = 0. Excess successes = extra damage dice.
Ends on: objective complete OR hero at 0 PS/PEs.
+5 PG on completion.
Le Fou: triggers at 4 instead of 6.
Le Soleil (Egoiste disadvantage, SPEC-24.3): cannot be Adjuvant. Additionally cannot use Assist action on allies.

## SPEC-10: Fold State
### Purpose
When PA reaches 0, the meta-armor folds into a minimal Guardian Suit. [v3] Fully specified (C4 resolved).

### [v3] Trigger
PA reaches 0 from any damage source. Immediate — no action, no roll.

### [v3] Guardian Suit Stats

| Stat | Value |
| --- | --- |
| PA | 5 (fixed, does not scale) |
| CdF | 5 (fixed, replaces armor CdF) |
| PS | Unchanged (retains current PS) |
| PE | Frozen at current value. Cannot spend or recover PE. |
| PEs | Unchanged. Espoir system continues normally. |


### [v3] Systems OFFLINE in Fold State
All ODs: Overdrive levels do not apply. Auto-successes lost. Tier bonuses lost.
All modules: Cannot activate any slotted modules.
Signature abilities: Warrior Types deactivate. Paladin Shrine/Watchtower deactivate. Priest NanoC/Mechanic unavailable. Rogue Ghost deactivates.
PE spending: All PE expenditure blocked. PE pool frozen (value preserved, not lost).
Combat styles: Forced to Standard. Cannot switch styles.

### [v3] Systems ACTIVE in Fold State
Weapons: All equipped weapons remain usable. Base damage only (no OD bonuses).
Nods: All Nod types usable (Soin, Armure, Énergie). Nod d'Armure is the primary exit path.
Free actions: Weapon swap, communication, style switch (but forced Standard, so no effect).
Movement: Normal movement rules. Position system unchanged.
Combo Rolls: Base + Combo (no OD auto-successes). PEs penalty still applies.
Heroism spending: All Héroïsme abilities remain available (Ignorer Agonie, Relancer, Dégâts Maximum, Mode Héroïque).

### [v3] Exit Conditions
Nod d'Armure: Restores PA. If restored PA > 5, armor reactivates. All systems come back online.
Priest Mechanic: Restores 3D6+6 or 2D6+6 PA. Same exit logic.
Any PA restoration: The moment currentPA exceeds Guardian Suit's 5, armor unfolds.
On exit: CdF returns to armor base. ODs reactivate. Modules/abilities available. PE spending unlocked.

### [v3] Edge Cases
Damage while folded: If Guardian Suit PA (5) and CdF (5) are overwhelmed, remaining damage hits PS. PS = 0 triggers Agony (SPEC-08).
PA passthrough in fold: Still applies. floor(paAbsorbed/5) = PS lost. Infatigable still overrides.
Multiple folds per combat: Possible. Each time PA reaches 0, fold triggers. Each restoration exits fold.
Fold + Agony: If knight is in Agony AND fold simultaneously (PS = 0, PA = 0), both states apply. Healing PS exits Agony; restoring PA exits fold. Order independent.

### [v3] Status Effect Entry
```csharp
StatusEffect 'Foldé' {
  trigger: currentPA == 0
  overridePA: 5; overrideCdF: 5;
  disableODs: true; disableModules: true;
  disableAbilities: true; freezePE: true;
  forceStyle: Standard;
  exitCondition: currentPA > 5 (after any PA restore)
}
```


### Acceptance Criteria
PA hits 0: fold triggers. CdF=5, PA=5. ODs gone.
Warrior in Fold: active Type deactivated. Cannot activate Types.
Rogue in Fold: Ghost deactivated. Cannot activate Ghost.
Knight in Fold uses Nod d'Armure (+10 PA): PA=15, exits fold. All systems online.
Knight in Fold attacked: 12 damage. CdF absorbs 5 = 7 remaining. PA absorbs 5 = 2 remaining hits PS. Passthrough: floor(5/5)=1 PS.
Knight in Fold + Agony: both states active. Heal PS = exit Agony. Restore PA = exit fold.

## SPEC-11: Bande Controller
### Purpose
Manages Bande entities. [v2] Completely rewritten. Bandes have NO attacks. Débordement only.
### [v2] Core Rules
Bande = single entity representing 3–12+ creatures.
Cohesion: Damaged by Violence only. Dégâts (weapon damage) are completely ignored against Bandes. Knights must use high-Violence weapons (Fusil d’Assaut, grenades) to reduce Cohesion effectively.
[v2] Débordement is the Bande’s ONLY offensive mechanic. No attacks, no hit rolls.
Débordement: Automatic damage to ALL enemies each turn. Cumulative: turn N = N × débordementScore. [v8] Débordement damage is resolved through the standard Knight Armour Layer chain (SPEC-06). CdF is the knight’s total (base + Shrine bonus); this total CdF is subject to all normal modifiers including IGNORE_CDF and Pénétrant X. Apply any Débordement Tags (e.g., IGNORE_CDF for Nocte) during resolution. PA absorbs normally. No attack roll — damage is automatic, bypassing hit determination. [v8.3 M-4 FIX] Clarification: Shrine (SPEC-17.2) adds +6 to the knight's CdF total. It is NOT a separate defensive layer — it is part of CdF. Therefore IGNORE_CDF bypasses both base CdF and Shrine bonus entirely. Pénétrant X reduces the combined total (base + Shrine). Agents must NOT implement Shrine as a separate shield; it modifies the CdF value that SPEC-06 Step 2 already handles. [v8.5 M-3 FIX] Worked Example — Nocte Débordement vs Shrine: Nocte bande, Turn 3, débordementScore 4. Damage = 3 × 4 = 12 per knight. Target: Paladin at Rank 2 with base CdF 8 + Shrine +6 = total CdF 14. Nocte Débordement carries IGNORE_CDF tag. Resolution: SPEC-06 Step 2 — IGNORE_CDF active, skip CdF step entirely. CdF 14 (including Shrine) provides zero protection. Full 12 damage → PA absorbs 12. PA: 120 → 108. Passthrough: floor(12/5) = 2 PS lost. Shrine is useless against Nocte. This is the intended threat model — Nocte bandes pressure through Shrine, forcing Violence-based weapons to reduce Cohesion rather than relying on CdF tanking.
Initiative: Bandes always act last (initiative 1).
One Bande rule:

### [v2] Targeting Rules
Bandes do not target. Débordement hits all enemies automatically. No position restriction.
Débordement ignores stealth. Ghost Mode does not protect against Débordement.
Knights target Bande from any position using Courte+ range. No gap check.
Smart Grenades deal Violence to Cohesion from any position.
### [v8] Knight vs Bande Attack Procedure
Knights attack the Bande using a standard Combo Roll (SPEC-04). Compare total successes vs the Bande’s Defense (contact attacks) or Reaction (ranged attacks). On hit: apply the weapon’s Violence dice to Cohesion. Weapon damage (Dégâts) is completely ignored — only Violence reduces Cohesion. On miss: no effect. Exploit and Failure Critique rules apply normally. Excess successes with Assistance à l’Attaque convert to +1 Violence (not damage). Range: Courte+ from any position, no gap check (Bandes are off-track, see core rules above).

### Data Model
```csharp
class BandeController {
  int cohesion;
  int debordementScore;
  int debordementTurn; // starts at 1, increments at turn end
  bool isActive;

  int GetDebordementDamage() => debordementTurn * debordementScore;

  void TakeDamage(int rawViolence) { // [v8] Only Violence reduces Cohesion. Dégâts are ignored.
    cohesion -= rawViolence;
    // [v8] Dégâts do not affect Bandes — only Violence reduces Cohesion
    if (cohesion <= 0) Destroy();
  }
  void AddCohesion(int amount) { cohesion += amount; }
  void OnTurnEnd() { debordementTurn++; }
}
```


### Acceptance Criteria
Cohesion 30, Débordement 3: Turn 1=3 to all, Turn 2=6, Turn 3=9.

Second Bande spawn: first Bande’s Cohesion increases.
[v2] Rogue in Ghost still takes Débordement.
[v2] No Bande attack actions exist. No hit rolls. Débordement only.

## SPEC-12: Motivation Event Detector
### Purpose
Listens to game events, checks Motivations for fulfillment. Primary PEs recovery.
On event: check all knights’ 3 minor Motivations against event tags.
Match: +1D6 PEs. Fire MotivationFulfilled UI event.
Major Motivation: +25 PEs current+max. Erased. New one at Camelot.
[v8.2 M-3] SPEC-12.1 — Motivation Template Pool
Each Motivation template has: id, displayText (French flavor), eventTag (matched by SPEC-12 event system), type (minor/major). At generation (SPEC-02 step 6): draw 2 unique minor templates + 1 major template. No duplicates within a knight. 3rd minor = Blason Voeu (always tag BLASON_VOEU, fulfilled by narrative mission events). At Camelot between missions: spent major is replaced by a new random draw from the pool (excluding the one just completed).
Minor Motivation Templates (10) — repeatable, +1D6 PEs per trigger:

[MN-02] «Abattre plusieurs ennemis» — Tag: MULTI_KILL_3 — Trigger: knight kills 3+ enemies during a single combat turn (across all actions).
[MN-03] «Exploit héroïque» — Tag: EXPLOIT_LANDED — Trigger: knight lands an Exploit (SPEC-04) that hits its target.


[MN-06] «Ramener un allié de l’abîme» — Tag: DESPAIR_STABILIZED — Trigger: knight stabilizes a Despaired ally (SPEC-07.3 Stabilization).
[MN-07] «Tir de précision» — Tag: LONG_RANGE_KILL — Trigger: knight kills an enemy with a ranged attack at the weapon’s maximum range band.
[MN-08] «Bouclier impénétrable» — Tag: CDF_FULL_ABSORB — Trigger: knight’s CdF absorbs 100% of an incoming attack (0 damage reaches PA).
[MN-09] «Assassinat furtif» — Tag: GHOST_KILL — Trigger: Rogue knight kills an enemy while Ghost mode is active (same attack that breaks Ghost).
[MN-10] «Réparer sous le feu» — Tag: COMBAT_REPAIR — Trigger: Priest knight restores PA to an ally via NanoC/Mechanic (SPEC-17) during a combat turn.
Major Motivation Templates (5) — one-time, +25 PEs current+max, replaced at Camelot:
[MJ-01] «Aucune perte» — Tag: MISSION_NO_DEATHS — Trigger: complete a full mission (all 5 nodes) with zero knight deaths.
[MJ-02] «Terrasser le monstre» — Tag: BOSS_KILLED — Trigger: party kills a Hostile-tier or higher enemy (tier 4+, e.g. Ours Corrompu, Behémot).
[MJ-03] «Indemnes» — Tag: MISSION_HEALTHY_FINISH — Trigger: complete a mission with all 4 deployed knights above 50% maxPA.
[MJ-04] «Machine de guerre» — Tag: HIGH_DAMAGE_DEALER — [v8.3 M-3 FIX] Trigger: a single knight deals 100+ effective PS damage to enemies across one complete mission. Track cumulative damage actually subtracted from enemy PS (after all armor layers: CdF, Bouclier, PA). Raw damage and PA damage do not count. Only PS reduction is tracked.
[MJ-05] «Revenant» — Tag: FOLD_SURVIVOR — Trigger: a knight who entered Fold state (PA=0, SPEC-10) during the mission survives to mission end.



## SPEC-13: Save System & Node Transitions
### Purpose
Auto-save at node transitions. No mid-combat saves. Resume from combat start on quit. [v4] M4 resolved: no rest between nodes.
Save at: node transitions, campaign creation, return to Camelot.
Quit mid-combat: resume from combat node start (full state revert).

### [v4] Between-Node Transition (M4 Resolved — No Rest)
No automatic recovery of any resource between nodes. Full attrition.
PS: Carries over exactly as-is. No auto-heal.
PA: Carries over exactly as-is.
PE: Carries over exactly as-is.
PEs: Carries over exactly as-is.
Nod use allowed: Knights may use Nods during node transitions at no action cost (free outside combat). This is the ONLY recovery mechanism between encounters.
Injuries persist: All active injuries carry forward. No healing between nodes.
Dead knights stay dead. Permadeath is permanent. In DD rank system, empty rank remains empty.
Design intent: Maximum attrition pressure. Resource management (Nods, PE, PEs) is the core strategic layer. Matches roguelite philosophy of short, high-stakes runs.

## SPEC-14: Boss Phase Controller
### Purpose
Monitors Patron PS thresholds, triggers phase transitions.
### Ours Corrompu

Phase 2: acts twice per turn.

### Acceptance Criteria
Phase 1 PS=0: Phase 2 stats, fresh PS 120.
Phase 2 with active Bande: Cohesion +4.
Phase 2: 2 turns per round.

## SPEC-15: Turn Queue & Initiative
### Purpose
Turn order management. FFX-style timeline. [v4] M1 resolved: initiative fully specified. DD rank movement integrated.

### [v4] Initiative Roll (M1 Resolved)
When: Once at the start of each encounter. Never re-rolled.
Formula: 3D6 + Initiative score. Results persist for entire combat.
Trop prudent disadvantage: 2D6 instead of 3D6.
Bandes: Always initiative 1 (no roll).
Masque Majeur enemies: Always initiative 30 (no roll).
Ties: Higher derived Initiative score first. If still tied, random.

### [v4] Delay Initiative
A knight may delay to a lower initiative value (announced at start of their turn).
Delay applies to entire turn (both actions). Cannot split.

NPCs: only Patron-type enemies may delay.

### [v4] Turn Structure (DD Ranks)
Each turn: 1 Combat Action + 1 Movement Action, in either order.
May forfeit Combat Action for 2nd Movement Action (2 rank moves/swaps). Cannot reverse.
[v4] Movement in DD ranks: Move into empty adjacent rank OR swap with adjacent ally = 1 Movement Action. See SPEC-03.
[v4] Nod on ally:
Free actions: 1 per type per turn (Ghost activation, style switch, weapon draw, etc.).

### Acceptance Criteria
[v4] Initiative rolled at encounter start. Same values used on turns 2, 3, etc.
[v4] Knight delays from 15 to 8: acts at position 8 in queue. Next turn can act at 15 or delay again.
Bande always last (init 1). Masque Majeur always first (init 30).

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


## [v2] SPEC-17: Armor Ability System (§4.5)
### Purpose
Manages all demo armor signature abilities. Each armor class has distinct abilities with PE costs, action requirements, and tactical effects.

### 17.1 Warrior — Les Types
Interchangeable OD boosts linked to Aspects.

| Field | Value |
| --- | --- |
| Types at creation | 3 chosen from 5 (Soldier/Chair, Hunter/BÊTE, Scholar/Machine, Herald/Dame, Scout/Masque) |
| Activation | 1 Movement Action |
| PE cost | 1 PE/turn combat, 6 PE/scene outside |
| Duration | 1 turn (until start of next turn). Spend PE to prolong, no action. |
| Effect | +1 OD to all 3 Characteristics of the linked Aspect |
| OD stacking | Stacks above normal cap of 5 (can reach 6+) |
| Constraint | 1 Type active at a time. Max 1 switch per turn. |
| Fold interaction | Type deactivates on fold. Cannot reactivate until armor restored. |



| Type | Aspect | Characteristics Boosted |
| --- | --- | --- |
| Soldier | CHAIR | Force, Endurance, Déplacement |
| Hunter | BÊTE | Hargne, Combat, Instinct |
| Scholar | MACHINE | Tir, Savoir, Technique |
| Herald | DAME | Aura, Parole, Sang-Froid |
| Scout | MASQUE | Discrétion, Dextérité, Perception |


Demo generation: Default 3 Types = Soldier + Hunter + 1 random. Prioritizes primary Aspects.
Evolutions (locked): 150 PG: no-action activation. 200 PG: all 5 Types. 250 PG: +2 OD per Type.

### Acceptance Criteria


Double switch in 1 turn: rejected.
Fold: Type deactivates.

### 17.2 Paladin — Shrine + Watchtower
Shrine — defensive force field dome.

| Field | Value |
| --- | --- |
| Activation | Free (no action) |
| PE cost | 1 PE (self) / 2 PE (Moyenne range on ally) |
| Duration | 1 turn |
| Effect |  |
| Entry restriction | Enemies: Force<5 or Chair<10 can’t enter. Bande Débordement CdF still applies. |
| [v8] Shrine anchors to the rank where it was placed and cannot be moved or repositioned. The Paladin is free to move away after placing the Shrine. CdF bonus applies to all allies at the Shrine’s rank ±1 regardless of Paladin position. Deactivates if Paladin enters Fold or is destroyed. [v8.2 H-2 FIX] Shrine deactivated by Fold does NOT auto-reactivate when Paladin exits Fold. Paladin must spend PE and a combat action to place a new Shrine after exiting Fold. | [v8] Shrine is fixed to placed rank. Paladin free to move away. |
| Limit | 1 per Paladin. No overlap. |
| vs Ignore CdF | Shrine CdF also bypassed. |


Watchtower — stationary fire platform.

| Field | Value |
| --- | --- |
| Activation | 1 Movement Action |
| PE cost | 2 PE to activate. +1 PE per extra shot. |
| Duration | Until deactivated (free action) |
| Effect | Cannot move. Reaction halved. +1 ranged Combat Action/turn (starting next turn). |
| Style restriction | Incompatible with Ambidextre. |


Lente et Lourde (passive) — Cannot use Course/Déplacement Silencieux/Saut modules. Discrétion and Déplacement tests +1 difficulty.
Evolutions (locked): 150 PG: slot ranged weapons on arms. 200 PG: Shrine +8 CdF. 250 PG: Lente et Lourde removed.

### Acceptance Criteria
[v7] Shrine (1 PE): +6 CdF to all allies at Shrine rank ±1. Shrine anchors to placed rank. Allies who move beyond rank gap > 1 lose the bonus.
Watchtower: immobile, Reaction halved, +1 ranged action next turn.
2 Shrines: second rejected.
Ignore CdF attack vs Shrine: CdF bonus bypassed.

### 17.3 Priest — NanoC + Mechanic
NanoC — creates temporary tactical constructions. Demo uses predefined menu.

| Field | Value |
| --- | --- |
| Activation | Varies by tier (see menu) |
| PE cost | 3 (simple) / 6 (detailed) / 9 (mechanical) |
| Duration | 10 turns combat / 1 min outside. +2 PE per extension. |
| Combo Roll | Base Technique (tabletop). Demo: auto-success for predefined items. |


Predefined NanoC Menu (Demo):

| Construction | Tier | PE | Action | Effect |
| --- | --- | --- | --- | --- |
| Cover Wall | Simple | 3 | Movement |  |
| Barricade | Simple | 3 | Movement | Blocks forced displacement through position. 10 turns. |
| Repair Platform | Detailed | 6 | Combat | Mechanic repairs here gain +1D6 PA. 10 turns. |
| Nano-Trap | Mechanical | 9 | Full Turn | Next enemy entering loses 1 action. Single use. |


Mechanic — repairs allied armor PA.

| Range | Activation | PE | Effect |
| --- | --- | --- | --- |
| Contact (same position) | 1 Movement Action | 4 | Restores 3D6+6 PA |
| Long (any position) | 1 Movement Action | 4 | Restores 2D6+6 PA |


Evolutions (locked): 150 PG: NanoC 1hr duration. 200 PG: Mechanic +1D6+6. 250 PG: create vehicles.

### Acceptance Criteria

Mechanic contact: 3D6+6 PA. Range: 2D6+6 PA.
NanoC expires at turn 10: effect removed.
Fold: NanoC and Mechanic unavailable.

### 17.4 Rogue — Mode Ghost
Stealth system with alpha-strike damage bonus.

| Field | Value |
| --- | --- |
| Activation | Free (no action) |
| PE cost | 2 PE/turn combat, 6 PE/min outside |
| Duration | 1 turn combat / 1 min outside |
| Stealth | Invisible. +3 auto-successes on stealth rolls. |
| Immunity | [v8] All enemies can attempt detection (see Detection roll). Machine Exc. Majeure: auto-detect all Ghosted knights without rolling — Ghost is completely ineffective against these enemies. |
| Alt-vision enemies | +3 effective Machine for detection. |
| Detection roll |  |


Attack from Ghost:

| Field | Value |
| --- | --- |
| Trigger | Any attack while Ghost active. |
| Ghost deactivation | Immediate after attack resolves. |
| Bonus dice | +Discrétion (with ODs) to attack roll. |
| Bonus damage | +Discrétion (with ODs) as flat damage. |
| Weapon restriction | [v8] Ranged weapons require the Silencieux effect tag. Contact weapons require NO Lumière effect tag. Ranged without Silencieux breaks Ghost before damage (no stealth bonus applied). Contact with Lumière breaks Ghost before damage. |
| Scope | First attack of turn only. |
| Re-stealth | [v8] Next turn: 2 PE to re-activate Ghost. New Discrétion combo roll establishes new stealth threshold for enemy detection. |


Débordement interaction: Bande Débordement hits Ghosted Rogues. Ghost does not protect.
Evolutions (locked): 150 PG: 6-turn duration. 200 PG: auto-success. 250 PG: 3 PE to stay invisible through attacks.

### Acceptance Criteria
[v8.7 L-4 FIX] Ghost activation: Free Action (no combat/movement action cost), 2 PE per turn maintenance. Invisible to enemies, can’t be targeted.
Discrétion 5 + OD 2 attacks: +7 dice, +7 flat damage. Ghost drops.
Lumière weapon: no Ghost bonus.
Débordement: Rogue in Ghost still takes damage.
0 PE at turn end: Ghost deactivates.


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
  // struct AspectExceptionnel { bool isMineur; int bonus; }
  // Mineur(N): +N automatic successes when using this aspect offensively
  // Majeur(N): +N automatic successes AND halve incoming damage using this aspect

  // Derived Values (all tiers)
  int defense; // Opposition threshold for melee
  int reaction; // Opposition threshold for ranged
  int initiative;

  // Tier-dependent health
  // T1 Bande:
  int cohesion; // 100/200/300/450 — reduced by Violence
  int debordement; // Cumulative damage per turn (see SPEC-11)

  // T2+ Individual:
  int maxPS; int currentPS; // Health (Points de Santé)
  int maxPA; int currentPA; // Armor (Points d’Armure) — [v8] default 0 if not specified
  int bouclier; // Shield — [v8] default 0 if not specified. T3+ only

  // T4 Colosse special:
  bool isColosse; // If true: 10 raw damage = 1 effective PS/PA loss

  // T5 Patron special:
  string pointFaible; // Characteristic name (e.g. 'Endurance')
  BossPhase[] phases; // Phase transitions (see SPEC-14)

  // Combat capabilities
  WeaponProfile[] weapons; // Contact and/or ranged
  EnemyCapacity[] capacities; // Special abilities
  EnemyAI aiProfile; // Targeting logic

  // Runtime state
  int ddRank; // 1–4 in DD rank system
  List<StatusEffect> activeStatusEffects;
  int actionsRemaining; // Base 1, +N from Actions Multiples
  bool isDead;
}
```


### 18.3 Aspects Exceptionnels Rules
Many T3+ enemies have exceptional aspects. These are critical for combat math:
Mineur (N): Enemy adds N automatic successes (flat bonus to success count, not extra dice) when rolling this aspect offensively (attack, resist). Functions identically to knight OD.
Majeur (N): Same as Mineur (N auto-successes) PLUS: incoming attacks that use this aspect for damage calculation have their damage halved (round down).
PNJ use Aspect/2 (round down) + Exceptionnel bonus as the opposition score (defense/reaction equivalent) when defending against knight actions.

### 18.4 Bouclier vs Champ de Force
Bouclier functions identically to knight CdF (subtracted from raw damage before PA/PS), but with one critical difference:
Pénétrant X ignores CdF but does NOT ignore Bouclier.
Anti-Anathème ignores Bouclier AND Chair Exceptionnelle.
Bouclier regenerates only if a specific capacity says so (it does not auto-regen).

### 18.5 Colosse Damage Rule (T4)
Colosses require complete tranches of 10 raw damage to lose 1 PS or PA. Remainder is discarded.


Anti-Véhicule weapon effect bypasses this rule: damage applies normally (1:1).

### 18.6 Point Faible (T5 Patrons)
Each patron has a declared weak point (a Characteristic). When a knight attacks using that Characteristic in their combo:
+1D6 bonus damage (additive, rolled and summed).
Ignore Bouclier for that attack.
[v8] UI must display the Point Faible once discovered. Discovery in demo: only random narrative events during mission nodes can reveal it. There is no in-combat discovery, no auto-reveal at any specific node, and no guaranteed discovery path. In-combat Perception-based discovery is NOT available in the demo — the tabletop allows this via the Warmaster class or specific modules, neither of which are in demo scope. Future implementation note: when Warmaster is added, discovery requires 1 Combat Action + Combo Roll (Perception Base), difficulty per tabletop rules.

### 18.7 Enemy AI Profile (EnemyAI)
Each enemy has a targeting profile that determines behavior in combat. This replaces a human GM's tactical decisions.
```csharp
struct EnemyAI {
  TargetPriority primary; // Which knight stat to sort by
  TargetPriority secondary; // Tiebreaker
  bool preferHighest; // true = attack highest stat, false = lowest
  RankBehavior rankBehavior; // HOLD, ADVANCE, RETREAT, FLANK
  int aggressionLevel; // 1–3 (1=cautious, 3=suicidal)
}

enum TargetPriority {
  HIGHEST_COMBAT, HIGHEST_BETE, HIGHEST_DAME, HIGHEST_AURA,
  LOWEST_PS, LOWEST_PA, LOWEST_PEs, NEAREST_RANK, RANDOM
}

enum RankBehavior {
  HOLD,    // Stay in current rank
  ADVANCE, // Move toward Rank 1 (front) if empty
  RETREAT, // Move toward Rank 4 (back) if empty
  FLANK   // Target rear-rank knights if possible
}
```


### 18.8 Actions Multiples
T3+ enemies may have Actions Multiples (N): N extra combat actions per turn beyond the standard 1 combat + 1 movement. These extra actions execute at the same initiative count.
Actions Multiples (1): 2 total combat actions.
Actions Multiples (3): 4 total combat actions.
Each combat action can target a different knight.

### 18.9 Enemy Attack Procedure
[v6] NEW — Critical C1 resolution. Complete step-by-step procedure for enemy attack execution. This is the enemy-side mirror of SPEC-04 (knight Combo Roll). Enemies use a fundamentally simpler system: single aspect, no combo, no OD equivalent.

IMPORTANT DISTINCTION:
ATTACKING (enemy's turn): Enemy uses full Aspect score as dice pool.
DEFENDING (opposition score): Enemy uses [v7] Aspect/2 (round down). Only specific Exceptionnel types modify derived values: Masque Exc Mineur adds to Defense, Machine Exc Mineur adds to Reaction. Other Exceptionnel types (Bête, Chair, Dame) do NOT affect Defense/Reaction.

Step 1 — Select Weapon:
AI profile determines contact vs ranged based on current rank, target availability, and weapon range bands (SPEC-03).
If enemy has multiple weapons, priority: (1) weapon whose range band covers the current target, (2) highest damage weapon if multiple qualify.

Step 2 — Select Aspect:
Each enemy weapon specifies its attack aspect in the stat block. If not explicitly specified, use defaults:
Contact weapons: highest of Chair or Bête
Ranged weapons: highest of Machine or Masque
The tabletop allows MJ to choose freely; for digital, the aspect is deterministic per weapon.

Step 3 — Build Dice Pool:
Pool = full Aspect score (NOT divided by 2).
No combo. PNJ never combo — this is explicitly a player privilege.
No OD equivalent. Aspect Exceptionnel adds auto-successes (Step 5), not dice.

Minimum pool: 0 dice (auto-fail if reduced to 0).

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
 Attack auto-misses. Enemy is exposed — next knight attack vs this enemy gets +2 auto-successes.

Step 7 — Damage Roll (on hit):
Roll weapon damage dice. SUM all face values + weapon flat bonus. [v8.2 H-1 FIX] PNJ weapon stat blocks (SPEC-21) include ALL flat bonuses pre-baked (Force/Tir bonus, weapon base bonus). Do NOT add Aspect-derived bonuses again at roll time. Agent implementation: use damageFormula string as-is (e.g. Faune “4D6+16” = roll 4D6, add faces, add 16). No further modifier lookup required for PNJ damage.
Apply weapon effects per SPEC-20 (Destructeur, Meurtrier, Dégâts Continus, etc.).
[v8] For weapon effects requiring a Characteristic score (Précision adds Tir, Orfèvrerie adds Dextérité): PNJ use full relevant Aspect score for all offensive calculations (attack pools, damage bonuses, capacity bonuses such as Charge Brutale). PNJ use Aspect/2 (round down) only for opposition scores when defending (Defense, Reaction).
Route total through target's ArmourLayerResolver (SPEC-06).

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
    for each combatAction (1 + Actions Multiples count):
      executeAttack(enemy, target)  // per SPEC-18.9
      // [v8.2 H-4 FIX] Cadence X handling (SPEC-20.4)
      if weapon.hasCadence(X):

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
### [v8.3 M-5] 18.12 Hybrid AI Pattern (RankBehavior Split)
[v8.3 M-5 FIX] Some enemies have a preferred combat style (melee or ranged) but can adapt situationally. This is modeled as a Hybrid RankBehavior that combines a spawn-time preference with per-turn situational adaptation. Generic pattern — reusable for any enemy with multiple weapon types.
Hybrid AI Schema: struct HybridRankBehavior { float meleeWeight; // 0.0–1.0. Bestian = 0.75. RankBehavior spawnPreference; // Rolled once at spawn: random(0,1) < meleeWeight ? ADVANCE : HOLD. Locked for this enemy instance. RankBehavior currentBehavior; // May be overridden per-turn by situational rules below. }
Per-turn situational override (applied in 18.11 Phase 2 MOVEMENT): (1) If spawnPreference = ADVANCE AND enemy is at Rank 1 AND contact target is in range: use ADVANCE (attack in melee). No override needed. (2) If spawnPreference = ADVANCE AND enemy is NOT at Rank 1: ADVANCE toward Rank 1 to close distance. (3) If spawnPreference = HOLD AND enemy has ranged weapon AND target is in ranged weapon range: HOLD (fire from current position). (4) OVERRIDE — If spawnPreference = HOLD BUT no target is within ranged weapon range AND enemy has contact weapon: temporarily switch to ADVANCE for this turn (close distance to attack). (5) OVERRIDE — If spawnPreference = ADVANCE BUT enemy is at Rank 3-4 with ranged weapon AND contact target is 2+ ranks away: temporarily switch to HOLD for this turn (fire ranged instead of wasting a turn moving). (6) If no weapon can reach any target from any behavior: skip attack (wasted turn).

Stat block notation: AI: RankBehavior field uses HYBRID_MELEE_XX or HYBRID_RANGED_XX where XX = melee/ranged weight as percentage (e.g. HYBRID_MELEE_75 = 75% melee preference). For non-hybrid enemies, use the existing HOLD/ADVANCE/RETREAT/FLANK enums as before.
### 18.10 Peur (Fear) Mechanic
Enemies with Peur (X) force a fear check at encounter start:
Knights test base Sang-Froid or Hargne vs enemy Aspect/2 (MJ choice, in digital: use highest enemy aspect/2).
Success:
Failure:
Échec Critique: [v7] Paralyzed for XD6 turns (no actions), where X = Peur level.
Each knight tests once per encounter per Peur source. Multiple Peur sources stack checks but not results (worst result applies).

### Acceptance Criteria
[v6] Enemy attack: Bestian (Bête 12, Bête Exc Mineur 3) attacks knight (Def 6). Rolls 12D6, gets 5 evens + 3 auto-successes = 8 total. 8 > 6: hit.
[v6] Enemy attack: Faune (Machine 10) fires ranged weapon at knight (React 4). Rolls 10D6, gets 3 evens. 3 < 4+1: miss (must exceed, not equal).
[v6] Enemy Failure Critique: all odd on 8D6. Attack auto-misses. Next knight attack vs this enemy: +2 auto-successes.
Bande with Cohésion 200, Débordement 6: turn 3 deals 18 damage. Violence reduces Cohésion.



Salopard with Actions Multiples (1): executes 2 combat actions at its initiative.


## [v5] SPEC-19: Weapon Data Model & Demo Arsenal
### Purpose
Defines the WeaponProfile schema and provides complete stat blocks for all weapons available in the demo. Links to SPEC-20 for effect definitions.

### 19.1 WeaponProfile Schema
```csharp
class WeaponProfile : ScriptableObject {
  string weaponId; string displayName;
  WeaponCategory category; // STANDARD, ADVANCED, RARE
  WeaponSlot slot; // CONTACT, RANGED, DUAL_PROFILE
  int pgCost; // Glory point cost (for future purchase system)

  // A weapon may have multiple profiles (e.g. Marteau-Épieu: contact + tir)
  DamageProfile[] profiles;
}

struct DamageProfile {
  string profileName; // e.g. 'Contact', 'Tir', 'Grenade Explosive'
  ProfileType type; // CONTACT, RANGED
  int damageDice; // Number of D6 for damage
  int damageFlat; // Flat bonus added to damage roll
  bool addForce; // If true, add Force score (contact weapons)
  ForceMode forceMode; // NORMAL (×1), LESTE (×2)
  int violenceDice; // Number of D6 for violence
  int violenceFlat; // Flat bonus to violence
  WeaponRange range; // CONTACT, COURTE, MOYENNE, LONGUE, LOINTAINE
  int energyCost; // PE per use (0 = free)
  WeaponEffectId[] effects; // References to SPEC-20 effect definitions
}

enum WeaponRange { CONTACT, COURTE, MOYENNE, LONGUE, LOINTAINE }
enum ForceMode { NONE, NORMAL, LESTE }

### 19.2 Range Rules (Tabletop Faithful)

| Range | Distance | Targeting in DD Ranks |
| --- | --- | --- |
| Contact | Melee (0–2m) |  |
| Courte | 2–15m |  |
| Moyenne | 15–50m | Any rank on either side |
| Longue | 50–300m | Any rank on either side |
| Lointaine | 300m+ | Any rank on either side |

Firing beyond range:

### 19.3 Demo Weapon Roster
Complete stat blocks for all weapons available in the demo. Every knight receives a Pistolet de Service + Marteau-Épieu as standard kit, plus class-specific weapons.

### Pistolet de Service (Standard Ranged — 15 PG)
Service sidearm. All knights carry one. Dual-profile: contact knife + ranged pistol.

| Profile | Damage | Violence | Range | Energy | Effects |
| --- | --- | --- | --- | --- | --- |
| Contact (knife) | 1D6 + Force | 1 | Contact | — | Silencieux |
| Tir (pistol) | 2D6 + 6 | 1D6 | Moyenne | — | Silencieux |


### Marteau-Épieu (Standard Contact — 10 PG)
Heavy warhammer with built-in single-shot incendiary charge. Default melee for all knights.

| Profile | Damage | Violence | Range | Energy | Effects |
| --- | --- | --- | --- | --- | --- |
| Contact | 3D6 + Force | 1D6 | Contact | — | Perce Armure 40 |
| Tir (charge) | 3D6 + 12 | 3D6 + 12 | Courte | — | Dégâts Continus 3, Dispersion 3, Lumière 2, Chargeur 1 |


### Couteau de Combat (Standard Contact — 15 PG)
Combat knife. Silent, precise. Favored by Rogues.

| Profile | Damage | Violence | Range | Energy | Effects |
| --- | --- | --- | --- | --- | --- |
| Contact | 3D6 + Force + Dextérité (Orfèvrerie) | 1D6 | Contact | — | Orfèvrerie, Silencieux, Jumelé (Ambidextrie) |


### Fusil de Précision (Standard Ranged — 40 PG)
Sniper rifle with camera-assisted targeting. Priest/Rogue favorite.

| Profile | Damage | Violence | Range | Energy | Effects |
| --- | --- | --- | --- | --- | --- |
| Tir | 4D6 + 6 + Tir (Précision) | 1D6 | Lointaine | — | Tir en Sécurité, Précision, Assistance à l'Attaque, Désignation, Deux Mains |


### Fusil d'Assaut (Standard Ranged — 30 PG)
Assault rifle. High violence for bande suppression.

| Profile | Damage | Violence | Range | Energy | Effects |
| --- | --- | --- | --- | --- | --- |
| Tir | 2D6 + 6 | 3D6 + 9 | Longue | — | Ultraviolence, Barrage 2, Deux Mains |


### Pistolet Mitrailleur (Standard Ranged — 25 PG)
Twin-barrel SMG. High rate of fire.

| Profile | Damage | Violence | Range | Energy | Effects |
| --- | --- | --- | --- | --- | --- |
| Tir | 3D6 | 4D6 | Moyenne | — | Meurtrier, Ultraviolence, Jumelé (Akimbo) |


### Shotgun Escamotable (Standard Ranged — 20 PG)
Foldable shotgun. Devastating at close range.

| Profile | Damage | Violence | Range | Energy | Effects |
| --- | --- | --- | --- | --- | --- |
| Tir | 2D6 + 10 | 2D6 | Moyenne | — | Meurtrier, Choc 1, Barrage 2, Deux Mains |


### Lance-Grenade Léger (Standard Ranged — 40 PG)
Grenade launcher. Three ammo types. Changing type costs 1 full turn.

| Profile | Damage | Violence | Range | Energy | Effects |
| --- | --- | --- | --- | --- | --- |
| Explosive | 4D6 | 3D6 | Moyenne | — | Dispersion 3 |
| Antiblindage | 3D6 + 10 | 1 | Moyenne | — | Destructeur |
| Incendiaire | 2D6 | 3D6 + 10 | Moyenne | — | Dégâts Continus 3 |


### 19.4 Grenades Intelligentes
All knights receive 5 grenades per mission. Base stats: 3D6 damage, 3D6 violence, Courte range (extendable with Force OD). Choose type before each throw. Grenade use = 1 Combat Action, base Tir. [v8] Hit resolution: Combo Roll with Tir as Base Characteristic. Compare total successes vs target’s Defense (if adjacent rank) or Reaction (if ranged). On hit: apply grenade effects to target; Dispersion X applies splash damage to enemies within X ranks of the target (see SPEC-20.5). On miss: no effect. Force OD level determines max range but does not consume OD. Grenades ignore weapon effects — they use only the grenade’s own effect tags.

| Type | Additional Effects | Notes |
| --- | --- | --- |
| Shrapnel | Ultraviolence, Meurtrier, Dispersion 6 | Anti-personnel. Max carnage. |
| Flashbang | Choc 1 (auto), Barrage 2, Lumière 2, Dispersion 6 | No damage/violence dealt. Crowd control. |
| Anti-Blindage | Destructeur, Perce Armure 20, Pénétrant 6, Dispersion 6 | Anti-armor. Bypasses defenses. |
| IEM | Parasitage 2, Dispersion 6 | No damage/violence dealt. Disables machines. |
| Explosive | Anti-Véhicule, +3D6 vs objects/vehicles, Choc 1, Dispersion 3 | Environmental destruction. |


### 19.5 Unarmed Strike
Any knight can punch/kick without a weapon:
In meta-armor: 1D6 + Force (+ OD bonuses) damage. 1 violence.
Without armor: Force score as damage. 1 violence.

### 19.6 Default Loadouts by Armor Class
Each armor class receives the following at mission start. Player may customize from available roster (weapons purchased with PG in future versions).

| Armor Class | Default Weapons | Rationale |
| --- | --- | --- |
| Warrior | Marteau-Épieu + Fusil d'Assaut + Pistolet de Service | Frontline melee + AoE suppression |
| Paladin | Marteau-Épieu + Shotgun Escamotable + Pistolet de Service | Tanking + close-range damage |
| Priest | Marteau-Épieu + Fusil de Précision + Pistolet de Service | Support + long-range precision |
| Rogue | 2× Couteau de Combat + Pistolet de Service | Dual-wield stealth + silent ranged |

All knights additionally carry 5 Grenades Intelligentes and 3 Nods of each type (Soin, Énergie, Armure).

### 19.7 Enemy Weapon Profiles
Enemy weapons use the same DamageProfile schema but are defined inline on EnemyBase (not purchased separately). See SPEC-21 (future) for demo enemy stat blocks with complete weapon data.

### Acceptance Criteria

Marteau-Épieu Lesté variant would be Force×2 + 3/OD. Default Marteau is NOT Lesté.

[v8] Grenade Flashbang: 0 damage, 0 violence. On hit (standard Combo Roll vs target Reaction): no damage or violence is dealt. Only weapon effects are applied: Choc 1 (auto-apply, no Chair threshold needed), Barrage 2, Lumière 2, Dispersion 6. The grenade exists purely for tactical effect application.

All weapons resolve through SPEC-05 damage chain. Effects resolve per SPEC-20 glossary.

## [v5] SPEC-20: Weapon Effects Glossary
### Purpose
Canonical mechanical definitions for every weapon effect referenced in the spec. Each effect is a discrete system behavior that modifies the damage chain (SPEC-05), armor resolver (SPEC-06), or combat state.

### 20.1 Damage Modifier Effects

| Effect | Mechanic | Implementation Notes |
| --- | --- | --- |
| Assistance à l'Attaque | Each excess success above Defense/Reaction threshold = +1 flat damage (or +1 violence vs bandes). | Applied after hit confirmation, before routing to SPEC-06. |
| Dégâts Continus X | Target takes X damage at start of each of their turns for 1D6 turns. Ignores CdF/Bouclier. Cannot stack with itself on same target. | Track: target, remaining turns, X value. Duration rolled once on initial hit. |
| Destructeur |  | Applied during SPEC-06 PA step. After confirming damage reaches PA, roll 2D6, add to running total before PA absorption. |
| En Chaîne | On kill of Hostile or Salopard, spend 3 PE to make one free attack on another target at same range, same turn. | Trigger: target.isDead after damage resolution. 3 PE check. One chain per kill. |
| Fureur | On kill, next attack this turn gets +2D6 damage. | Trigger: target.isDead. Buff expires at end of turn. |
| Lesté | Force is multiplied by ×2 (instead of ×1) for melee damage calculation. | Modify SPEC-05 step 4: forceMode = LESTE. |
| Meurtrier | [v6] When damage reaches PS (at least 1 point passes CdF, Bouclier, and PA): +2D6 bonus damage added to total. Does NOT skip or bypass any defensive layer. All layers resolve normally. | Applied during SPEC-06 PS step. After confirming damage reaches PS, roll 2D6, add to running total before PS reduction. |
| Orfèvrerie | Add Dextérité score (+1/OD) to damage, like Précision adds Tir. | SPEC-05 step 5. Stacks with Force for contact weapons. |
| Précision | Add Tir score (+1/OD) to damage for ranged weapons. | SPEC-05 step 5. For PNJ: add Machine/2 (round down) instead. |
| Ultraviolence | Violence dice that roll 1 or 2 are rerolled once. Keep new result. | Reroll mechanic for violence (anti-bande). NOT related to Meurtrier (+2D6 bonus). |


### 20.2 Defense Bypass Effects

| Effect | Mechanic | Implementation Notes |
| --- | --- | --- |
| Anti-Anathème | Ignores Bouclier AND Chair Exceptionnelle entirely. PA then PS are hit directly. | Used vs Anathème creatures. Bypasses their special defenses. |
| Anti-Véhicule | Ignores Colosse tranche rule (damage applies 1:1). Also instantly destroys common vehicles. |  |
| Ignore Armure | Damage bypasses PA entirely, hitting PS directly. CdF/Bouclier still applies. | Skip PA step in SPEC-06. [v6] Distinct from Destructeur: Ignore Armure skips PA entirely, Destructeur adds +2D6 when damage reaches PA but PA still absorbs. |
| Ignore CdF | Damage bypasses Champ de Force. PA then PS take full damage. | Skip CdF step in SPEC-06. Does NOT bypass Bouclier. |
| Pénétrant X |  | Reduce effective CdF by X (min 0). Does NOT affect Bouclier. |
| Perce Armure X |  | Reduce effective PA by X (min 0). Remaining damage after CdF goes straight to PS. |


### 20.3 Crowd Control Effects

| Effect | Mechanic | Implementation Notes |
| --- | --- | --- |
| Barrage X | [v8] Alternative fire mode. Player chooses ‘Attack’ or ‘Suppress (Barrage)’ before rolling. Suppress: costs 1 Combat Action. No attack roll, no damage dealt, no other weapon effects apply (Ultraviolence, Meurtrier, etc. are inactive). Target must be within the weapon’s range band and subject to standard gap/position validation (SPEC-09). Line of sight required. Target’s Defense AND Reaction reduced by X until start of their next turn. Multiple Barrage effects from different sources stack. | Costs 1 Combat Action. No attack roll needed. Applied to single target or bande. |
| Choc X | If attack roll exceeds target's Chair/2 (round down): target loses X next actions. Does not stack while already under Choc. Does not stack with Parasitage. | Check after hit: successes > Chair/2. For auto-Choc: skip the comparison, always apply. |
| Démoralisant | Reduces bande Débordement by 2 for the current turn only. Multiple applications add duration, not extra reduction. | Modify SPEC-11 débordement calculation for current turn. |
| Dispersion X | Attack hits X additional enemies within 2m of primary target. One attack roll, compare vs each target's Defense/Reaction separately. All hit targets take full damage. | In DD ranks: hits X targets on same or adjacent ranks. If ally on same rank as target: ally also hit. |
| Lumière X | Target Anathème creatures with Hypersensibilité Lumineuse: disables 1+ capacities for 1D6 turns AND doubles damage dealt by this weapon. X is intensity. | Check target.hasHypersensibilite. Double damage in SPEC-05 step 8 (after all bonuses). |
| Parasitage X | Target loses X next actions. Requires Machine-based targets or IEM sensitivity. Cannot stack while active. Cannot stack with Choc. | Similar to Choc but for machine targets. Mutual exclusion with Choc. |


### 20.4 Targeting & Utility Effects

| Effect | Mechanic | Implementation Notes |
| --- | --- | --- |
| Artillerie |  | Requires Désignation active on target. See SPEC-03 Cover System. |
| Assassin X | Adds XD6 to damage during surprise attacks only. | Trigger: isSurpriseAttack flag on combat init. |
| Cadence X |  |  |
| Chargeur X | Weapon has X uses per mission. At 0: weapon cannot fire. Refill at Camelot. | Track uses remaining. See SPEC-04 §4.4 L4 rules. |
| Désignation | Mark one target (not bande) per turn. Allies get +1 auto-success vs target. Ignore échecs critiques vs target. No action cost. | Set designated flag on target. Persists until new target designated. |
| Deux Mains | Requires both hands. Cannot dual-wield or use shield simultaneously. [v8.8 M-2 FIX] On Ranged weapons: also enables Style Pilonnage (SPEC-04). | Equipment validation: block incompatible combinations. |
| Jumelé (Akimbo) |  | Modify SPEC-04 dual-wield penalty. |
| Jumelé (Ambidextrie) |  | Identical mechanic. Different flavor label. |
| Lourd | Cannot move and attack in same turn (movement action forfeited). [v8.8 M-2 FIX] Enables Style Puissant only. (Style Pilonnage requires Deux Mains on a Ranged weapon — see Deux Mains effect.) | If weapon.hasLourd: attack = forfeit movement. Unlock Style Puissant. |
| Silencieux | User not detectable by sound when attacking. From stealth/Ghost/Changeling: add Discrétion (+OD) to damage. |  |
| Tir en Sécurité |  | See SPEC-03 Cover System. Knight retains full defensive benefit while attacking. |


### 20.5 Dispersion Adaptation for DD Ranks
[v5] NEW: The tabletop defines Dispersion as hitting enemies 'within 2 meters.' In the DD rank system, this translates to:
Dispersion X hits up to X additional targets on the same rank or adjacent ranks (±1 rank from the primary target).
Friendly fire rule: If an allied knight occupies the same rank as a Dispersion target, the ally takes full damage. UI must warn player before confirming attack.
Bande exception: Dispersion does not affect bandes (use Violence instead per SPEC-11).
Cross-side reach: An attacker can Disperse across the center line if ranks are adjacent.

### 20.6 Effect Resolution Order
When a weapon has multiple effects, resolve in this order:
Pre-hit effects: Désignation (free), Barrage (replaces damage).
Hit determination: Attack roll vs threshold.
Damage modification: Précision/Orfèvrerie/Silencieux (flat bonus), Lesté (Force multiplier), Assassin (surprise bonus), Fureur (kill chain bonus), Ultraviolence (reroll 1-2 on violence dice).
Defense bypass: Anti-Anathème, Ignore CdF, Pénétrant, Ignore Armure, Perce Armure, Anti-Véhicule.
Layer-triggered bonuses: [v6] Destructeur (+2D6 on reaching PA), Meurtrier (+2D6 on reaching PS). Applied during SPEC-06 layer resolution.
Post-hit effects: Choc/Parasitage (action loss), Dégâts Continus (DoT apply), En Chaîne (chain kill), Lumière (vs Anathème).
AoE resolution: Dispersion (additional targets), Cadence (multi-target).

### Acceptance Criteria


Perce Armure 40 vs PA 30: PA fully ignored, damage hits PS.
Perce Armure 40 vs PA 60: PA reduced by 40, effective PA = 20. Damage absorbed by remaining PA.
Pénétrant 10 vs CdF 8: CdF fully ignored. vs Bouclier 8: Bouclier still applies (Pénétrant doesn't bypass Bouclier).
Choc 2 auto: target loses 2 actions regardless of Chair score.
Dispersion 3 in DD: primary target Rank 2, 3 additional enemies on Ranks 1–3. Allied knight on Rank 2: takes damage (friendly fire).




End of SPEC-20. Continue to SPEC-21 for demo enemy roster. [v8.5 L-4 NOTE] The encounter compositions defined in SPEC-22 (Demo Mission Structure) must be authored as EncounterSO instances per SPEC-29 (Encounter Authoring Schema). Do not hardcode encounter data in scene logic.
## [v5] SPEC-21: Demo Enemy Roster
### Purpose
Complete stat blocks for all enemies in the demo, using the EnemyBase schema from SPEC-18. Covers 5 enemy types spanning all tiers. Includes AI targeting profiles, weapon data, and capacity definitions.

### 21.1 Nocte (T1 Bande — La Bête)
Shadow-born flying swarm. First enemies encountered. Weak individually, devastating in numbers.

| Field | Value |
| --- | --- |
| Tier | T1 Bande (recrue) |
| Seigneur | La Bête |
| Aspects | Chair 4 / Bête 9 / Machine 1 / Dame 0 / Masque 4 |
| Aspects Exceptionnels | None |
| Défense 5 / Réaction 1 / Initiative 1 | — |
| Cohésion | 100 (Easy) / 200 (Normal) / 300 (Hard) |
| Débordement | 4 |
| [v7] Débordement Tags | [v7] ANTI_VEHICULE (Débordement damage bypasses Colosse 10:1 armor scaling). [v8] NOTE: In the demo, no knight targets use Colosse rules. This tag is scaffolded for future content where knights may use vehicles or destructible cover objects subject to Colosse-type damage reduction. Bandes do not attack — Débordement only (SPEC-11). |
| AI: Primary | [v7] N/A — Bandes use Débordement only, no targeting AI (SPEC-11) |
| AI: RankBehavior | [v7] N/A — Bandes do not move (Initiative always 1, SPEC-11) |
| AI: Aggression | [v7] N/A — Bandes are automatic damage sources |


Capacities:
Vol: Can fly. Treated as having Long range movement per turn.
Anti-Véhicule: Inherent effect on all attacks.
Hypersensibilité Lumineuse: Any Lumière effect disables 1+ capacities for 1D6 turns. Lumière weapons deal double damage to Cohésion.
Ignore CdF: Inherent on all attacks.

### 21.2 Bestian (T2 Hostile — La Bête)
Corrupted predatory animals. Contact fighters with ranged darkness attack.

| Field | Value |
| --- | --- |
| Tier | T2 Hostile (recrue) |
| Seigneur | La Bête |
| Aspects | Chair 4 / Bête 8 / Machine 2 / Dame 1 / Masque 5 |
| Aspects Exceptionnels | Bête: Mineur (2) |
| Défense 4 / Réaction 3 / Initiative 2 | — |
| PS | 20 |
| PA / Bouclier | 0 / 0 |
| Weapon (Contact) | Griffes et crocs: 5D6 + 2, Contact, no effects |
| Weapon (Ranged) | Flot de ténèbres: 3D6, Moyenne, Choc 1 + Anathème |
| AI: Primary/Secondary | HIGHEST_DAME / HIGHEST_AURA |
| AI: RankBehavior | [v8.3 M-5 FIX] HYBRID_MELEE_75 (see SPEC-18.12 Hybrid AI Pattern) |
| AI: Aggression | 2 |


Capacities:
Module (Saut nv1): Can leap to any rank in one movement action.
Anathème (Flot de ténèbres): [v7] AI always targets PEs (maximizes despair pressure). CdF still applies, PA does not. If Bestian is killed, PJs who suffered this effect recover 1D6 PEs per 6 PEs lost.

### 21.3 Faune (T3 Salopard — La Bête)
Corrupted humans. Elite melee fighters with brutal charge capability.

| Field | Value |
| --- | --- |
| Tier | T3 Salopard (recrue) |
| Seigneur | La Bête |
| Aspects | Chair 8 / Bête 10 / Machine 4 / Dame 4 / Masque 7 |
| Aspects Exceptionnels | Bête: Majeur (6), Masque: Mineur (1) |
| Défense 6 / Réaction 2 / Initiative 3 | — |
| PS | 80 |
| PA / Bouclier | 0 / 0 |
| Weapon (Contact) | Arme lourde: 4D6 + 16, Contact, no effects |
| AI: Primary/Secondary | NEAREST_RANK / HIGHEST_DAME |
| AI: RankBehavior | ADVANCE |
| AI: Aggression | 3 |


Capacities:
Charge Brutale:
Module (Saut nv2): Enhanced leap. Can cross 2 ranks in one movement.
Note: [v8] Bête Majeur (6) means +6 automatic successes offensively (NOT dice — guaranteed, not rolled) AND Bête Mineur bonus applies (add Exceptionnel score to contact damage: +6) AND Bête Majeur adds full Bête Aspect score to contact damage (+10). Total flat contact damage bonus: +16 (already included in weapon stat 4D6+16). Bête Exceptionnelle is purely offensive — it has no defensive halving effect.

### 21.4 Behemot (T4 Colosse — La Bête)
Massive fused beast. Requires Anti-Véhicule or sustained focus fire. Demo mid-boss.

| Field | Value |
| --- | --- |
| Tier | T4 Colosse (initié) |
| Seigneur | La Bête |
| Aspects | Chair 15 / Bête 16 / Machine 2 / Dame 2 / Masque 2 |
| Aspects Exceptionnels | Chair: Majeur (5), Bête: Majeur (6) |
| Défense 8 / Réaction 1 / Initiative 1 | — |
| PS | 200 |
| PA / Bouclier | 0 / 5 |
| Colosse Rule | YES — 10 raw damage = 1 effective. Anti-Véhicule bypasses. |
| Point Faible | Endurance |
| Weapon (Contact) | Griffes et crocs: 3D6 + 22, Contact, Choc 2 + Dispersion 3 |
| Weapon (Ranged) | Langue épineuse: 3D6, Courte, Ignore Armure |
| AI: Primary/Secondary | NEAREST_RANK / HIGHEST_MACHINE |
| AI: RankBehavior | ADVANCE |
| AI: Aggression | 3 |


Capacities:
Peur (1):
Charge Brutale:
Actions Multiples (1): 2 combat actions per turn total.
DD Rank placement: Behemot always occupies Rank 1 (front). Too large to be pushed. Immune to forced displacement.

### 21.5 Ours Corrompu (T5 Patron — La Bête)
Demo final boss. Two-phase fight based on SPEC-14 phase controller. Corrupted bear incarnation of La Bête.

[v7] Phase 1 — Raging Beast (PS > 0)

| Field | Value |
| --- | --- |
| Tier | T5 Patron |
| Seigneur | La Bête |
| Aspects | Chair 12 / Bête 14 / Machine 4 / Dame 6 / Masque 8 |
| Aspects Exceptionnels | Chair: Mineur (3), Bête: Majeur (6) |
| Défense 8 / Réaction 4 / Initiative 6 | — |
| PS | 120 |
| PA / Bouclier | 0 / 8 |
| Point Faible | Sang-Froid |
| Weapon (Contact) | Griffes: 5D6 + 18, Contact, Choc 1 |
| Weapon (Ranged) | Souffle ténébreux: 3D6, Moyenne, Dispersion 2 + Anathème |
| AI: Primary/Secondary | HIGHEST_BETE / HIGHEST_COMBAT |
| AI: RankBehavior | ADVANCE |
| AI: Aggression | 2 |


Phase 1 Capacities:
Peur (2):
Charge Brutale: Once per phase. Rush + weapon + 2× Bête (= +28). [v8] Routes through ArmourLayerResolver (SPEC-06).
Actions Multiples (1): 2 combat actions per turn.



| Field | Change from Phase 1 |
| --- | --- |
| PS | Resets to 120 (total encounter PS = 240) |
| Bouclier | Increases to 12 (from 8) |
| Défense | Increases to 10 (from 8) |
| Actions Multiples | Increases to (2) — 3 combat actions per turn |
| New Weapon | Ombre dévorante: 6D6, Longue, Ignore CdF + Dégâts Continus 3 |
| New Capacity | Régénération: if Nocte bande present, consume 50 Cohésion to heal 25 PS |
| AI change |  |
| Narrative | Darkness erupts. Arena goes dark. Lumière weapons critical. |


### 21.6 Encounter Composition Reference
Recommended enemy configurations for demo encounters, balanced for 4 knights in DD rank system:

| Encounter Type | Enemies | DD Rank Placement | Estimated Difficulty |
| --- | --- | --- | --- |
| Easy skirmish | 1 Nocte bande (100) | Bande fills all 4 ranks | ★☆☆☆☆ |
| Standard fight | 2 Bestians | Rank 1, Rank 2 | ★★☆☆☆ |
| Tough fight | 1 Faune + 1 Nocte bande (100) | Faune R1, Bande all | ★★★☆☆ |
| Elite fight | 1 Faune + 2 Bestians | Faune R1, Bestians R2/R3 | ★★★★☆ |
| Sub-boss | 1 Behemot + 1 Nocte bande (200) | Behemot R1, Bande all | ★★★★☆ |
| Boss | 1 Ours Corrompu + 1 Nocte bande (200) | Boss R1, Bande all | ★★★★★ |


### Acceptance Criteria
Nocte bande with Cohésion 200: Fusil d'Assaut Violence 3D6+9 (avg 19.5) reduces Cohésion.
[v7] Bestian Anathème attack: AI always targets PEs. CdF applies, PA does not.
Faune Charge Brutale: 4D6+16 base + 20 (2× Bête) = 4D6+36 on charge hit.


AI: Faune targets nearest rank. Bestian targets highest Dame knight.

## [v5] SPEC-22: Demo Mission Structure
### Purpose
Defines the complete node graph, encounter composition, narrative events, and victory/defeat conditions for the demo mission. This is the content that SPEC-13 (Save System) and SPEC-14 (Boss Phase) operate on.

### 22.1 Mission Graph — 'The Corrupted Den'
Linear-with-branches structure. 5 nodes total. Player cannot backtrack. No rest between nodes (SPEC-13 attrition rule). Resources carry over exactly.




| Node | Type | Encounter | Description |
| --- | --- | --- | --- |
| 1: Perimeter | Combat | 1 Nocte bande (200) | Nocte swarm defends outer perimeter. Teaches bande/violence mechanics. |
| 2A: Nest | Combat | 2 Bestians + 1 Nocte bande (100) | Bestian nest with residual noctes. Teaches individual enemy + Anathème. |
| 2B: Ruins | Combat + Event | 1 Faune | Lone Faune in abandoned ruins. Narrative event: discover survivor (PEs recovery opportunity). |
| 3: Breach | Combat | 1 Faune + 2 Bestians | Converging enemies at the den entrance. Hardest non-boss fight. |
| 4: Antechamber | Narrative | No combat | [v8] Calm before storm. Random narrative event: may discover Ours Corrompu Point Faible (not guaranteed). Optional: use Nods. |
| 5: The Den | Boss | Ours Corrompu + 1 Nocte bande (200) | Final boss. Two-phase fight per SPEC-14. |


### 22.2 Node Transitions
Between nodes, the following occurs:
Combat ends: All surviving enemies are destroyed (no fleeing in demo).
Loot/recovery: No automatic recovery. Knights may use Nods (no action cost between combats).
State persists: PS, PA, PE, PEs, injuries, Heroism, ammo — all carry over exactly.
Dead knights: Remain dead. Empty DD rank slot. Cannot be replaced mid-mission.
Branch selection (Node 2): Player chooses path. Choice is permanent. Both converge at Node 3. [v8.4 L-2 FIX] Branch selection screen: show node title + short narrative description + difficulty rating (stars). Do NOT show exact enemy composition. Example display: ‘2A: Le Nid — Corrupted creatures nest here. ★★☆☆☆’ vs ‘2B: Les Ruines — A lone hunter stalks the ruins. ★★★☆☆ (Event)’. The ‘Event’ tag hints at a narrative opportunity without revealing specifics.

### 22.3 Narrative Events

| Node | Event | Trigger | PEs Effect |
| --- | --- | --- | --- |
| 2B | Survivor discovered | Auto on entry | All knights: +1D6 PEs (minor motivation: protecting innocents) |
| 4 | [v8] [v8.4 M-3 FIX] Point Faible discovery: roll 1D6 at Node 4 entry. On 6 only: Sang-Froid is revealed as Point Faible (displayed on boss UI for remainder of mission). On 1–5: no discovery. Intentionally very difficult — in full game, Warmaster class and analysis modules provide more reliable paths | Roll 1D6 on entry; triggers on 6 only | Sang-Froid highlighted on boss UI. +3 PEs to knight with highest Perception. |
| 4 | Pre-battle prayer | Player choice | Spend 1 Heroism (any knight): all knights +1D6 PEs. Or skip. |
| 5 (phase 2) | Darkness erupts | Phase transition |  |


### 22.4 Victory Conditions

| Condition | Result |
| --- | --- |
| Ours Corrompu Phase 2 PS = 0 | MISSION SUCCESS. Return to Camelot. All surviving knights preserved. |
| All 4 knights dead/incapacitated | MISSION FAILURE. All deployed knights permanently dead. Return to Camelot with remaining 4 roster knights. |
| Player voluntary retreat (Node 1–4 only) | RETREAT. Deployed knights survive with current state. Mission available for retry. No Retreat from Node 5. |


### 22.5 Defeat Consequences
Total Party Kill: All 4 deployed knights are permanently dead. Player returns to Camelot with 4 remaining roster knights. If fewer than 4 remain in total roster, campaign over.
Partial losses: Dead knights are removed from roster permanently. Surviving knights return with current HP/PA/PE/injuries. Injured knights can receive cybernetic implants at Camelot (see SPEC-25).
Retreat: No permadeath. All deployed knights return to Camelot with current state. Mission resets but knight state persists.

### 22.6 Difficulty Scaling

Nocte Cohésion: Easy 100 / Normal 200 / Hard 300 (selectable at mission start or auto-scaling).
Enemy stat multiplier: Future missions can apply ×1.1 to ×1.5 on enemy PS/damage.
Additional enemies: Later missions add more enemies per node.
For the demo, difficulty is fixed. Scaling system is scaffolded but not active.

### Acceptance Criteria
5 nodes load sequentially. Node 2 branches. Node 3 converges.
No rest between nodes. Nods usable between combats (no action cost).
TPK at node 3: 4 knights permanently dead. Return to Camelot with 4 reserves.
Retreat from node 2: all 4 knights survive. Mission resets. State persists.
Boss fight: 2 phases. Phase 2 PS reset. [v8.4] Nocte bande: fresh spawn (200 Cohésion, Débordement 1) if destroyed; AddCohesion(200) if still alive.

## [v5] SPEC-23: Complete Injury Table
### Purpose
Full 4×6 random injury matrix from the tabletop, with digital adaptations for permanently disabling injuries. Replaces the partial table in SPEC-08. [v5] Resolves H1.

### 23.1 Injury Roll Procedure
When a knight enters Agony (PS = 0):
Roll 1D6
Roll 1D6

Injuries persist until cybernetic implantreconstruction therapy (100 PG, removes all injuries).

### 23.2 Full Injury Matrix
Column 1 (D6 = 1) — Catastrophic:

| Row | Injury | Digital Effect |
| --- | --- | --- |
| 1 | Mort (Death) | Knight is permanently dead. Remove from roster. |
| 2 | Organes Internes Touchés | [v8.8 H-1] All 15 characteristics -1 (floor 0). Recalculate all derived values (SPEC-01). |
| 3 | Cerveau Touché | [v8.8 H-1] 12 characteristics -1: all Bête (3) + Machine (3) + Dame (3) + Masque (3). Chair characteristics unaffected. Lose 1 action per turn (knight chooses which: Combat OR Movement). Recalculate Defense, Reaction, Initiative. |
| 4 | Colonne Vertébrale Touchée | [v8.8 H-1] 9 characteristics -2: all Chair (3) + Bête (3) + Masque (3). Recalculate maxPS, Defense, Initiative. |
| 5 | Hémorragie Interne | [v8.8 H-1] Same as standard Hémorragie (SPEC-08): 3-turn countdown to permadeath. Heal ≥1 PS before countdown expires to survive. |
| 6 | Poumons Perforés | [v8.8 H-1] Endurance -2, Déplacement -2 (floor 0). Recalculate maxPS. |


Column 2 (D6 = 2–3) — Severe:

| Row | Injury | Digital Effect |
| --- | --- | --- |
| 1 | Crâne Brisé | [v8.8 H-1] 9 characteristics -1: all Machine (3) + Dame (3) + Masque (3). Recalculate Reaction, Initiative. |
| 2 | Yeux Détruits | [v8.8 H-1] All attack rolls -3 dice (blind penalty, floor 0 dice). |
| 3 | Dos Brisé | [v8.8 H-1] 6 characteristics -1: all Chair (3) + Bête (3). Recalculate maxPS, Defense. |
| 4 | Main Brisée | [v8.8 H-1] Combat -1, Technique -1, Tir -1, Dextérité -1 (floor 0). Recalculate Defense, Reaction. |
| 5 | Oreille Éclatée | [v8.8 H-1] Perception -1, Déplacement -1, Instinct -1 (floor 0). Recalculate maxPS, Defense, Initiative. |
| 6 | Côtes Brisées | [v8.8 H-1] Endurance -1, Combat -1 (floor 0). Recalculate maxPS, Defense. |


Column 3 (D6 = 4–5) — Moderate:

| Row | Injury | Digital Effect |
| --- | --- | --- |
| 1 | Colonne Vertébrale Brisée | [v8] Movement costs 2 actions (any combination of movement or combat actions) per rank shift. Cannot Sprint. |
| 2 | Mâchoire/Langue/Dents Détruites | [v8.8 H-1] Aura -3, Parole -3, Sang-Froid -3 (floor 0). |
| 3 | Pied Brisé | [v8.8 H-1] Combat -1, Déplacement -1, Discrétion -1, Force -1 (floor 0). Recalculate maxPS, Defense, Initiative. |
| 4 | Dos Touché | [v8.8 H-1] Force -1, Déplacement -1 (floor 0). Recalculate maxPS. |
| 5 | Tempe Touchée | [v8.8 H-1] Hargne -1, Sang-Froid -1 (floor 0). Recalculate Defense. |
| 6 | Blessure Mineure | [v8.8 H-1] Random non-zero characteristic -1 (floor 0). Recalculate affected derived value. |


Column 4 (D6 = 6) — Light:

| Row | Injury | Digital Effect |
| --- | --- | --- |
| 1 | Bras Mutilé | Cannot use Lourd or Deux Mains weapons. +1 difficulty to relevant tests. |
| 2 | Jambe Mutilée | Movement costs doubled. Cannot use movement-dependent abilities. |
| 3 | Traumatisme Crânien | [v8.8 H-1] Savoir -1, Technique -1, Instinct -1 (floor 0). Recalculate Reaction, Defense. |
| 4 | Borgne | [v8.8 H-1] Tir -1, Perception -1 (floor 0). Recalculate Reaction, Initiative. |
| 5 | Blessure Mineure | [v8.8 H-1] Random non-zero characteristic -1 (floor 0). Recalculate affected derived value. |
| 6 | Rien! | No injury. Knight is in Agony but otherwise unscathed. |


### 23.3 Digital Adaptations
The tabletop leaves some injuries to 'GM discretion.' Digital implementation requires deterministic rules:
Colonne Vertébrale Brisée: [v8] Tabletop = ‘legs unusable, tests impossible.’ Digital = movement costs 2 actions (any combination of movement or combat actions). In demo (1 movement + 1 combat), this means a position shift costs the entire turn. Implement as doubled action cost, not “whole turn” — future capacities may grant additional actions.
Yeux Détruits:
Bras/Jambe Mutilé: Tabletop = 'MJ increases difficulty.' Digital = equipment restrictions + movement penalty.
Random characteristic (Blessure Mineure): System picks random non-zero characteristic and reduces by 1.

### 23.4 Trompe la Mort (Tarot Advantage)
Knights with the Trompe la Mort advantage (Arcane sans Nom, Tarot XIII): when rolling Mort on the injury table, reroll and take next result. Once per mission.

### Acceptance Criteria
Roll [1,1]: Mort. Knight permanently dead unless Trompe la Mort active.
Roll [1,5]: Hémorragie. 3-turn countdown. Nod Soin before 0: survives.

Roll [6,6]: Rien. Knight in Agony but no injury penalty.
Cerveau Touché: [v8.8 L-1 FIX] 12 characteristics reduced by 1: all Bête (3) + Machine (3) + Dame (3) + Masque (3). Chair characteristics (Déplacement, Force, Endurance) unaffected. Lose 1 action per turn (knight chooses which: Combat OR Movement).

## [v5] SPEC-24: Tarot Advantages & Disadvantages
### Purpose
Complete mechanical definitions for all Tarot-derived advantages and disadvantages used in knight generation (SPEC-02 step 4). Each knight draws 5 cards from the full 22-card Major Arcana, selects 2 advantages and 1 disadvantage. [v6] Corrected: 5 cards drawn (not 3), full 22-card pool (not 12).

### 24.1 Full Tarot Pool
[v6] The demo uses the full 22-card Major Arcana (Tarot de Marseille, cards 0–XXI). Each card grants +1 Aspect point, +3 Characteristic points, 1 Advantage, and 1 Disadvantage. [v7] All 22 Major Arcana cards are fully defined below with complete mechanical advantage/disadvantage pairs. Note: Le Fou (0) and La Maison-Dieu (XVI) have special bonus structures — see 24.4. Incompatibility: L'Arcane sans Nom (XIII) and La Maison-Dieu (XVI) cannot both appear in the same knight's draw — if both are drawn, redraw one.

### 24.2 Advantages

| Tarot | Advantage | Mechanic |
| --- | --- | --- |
| I. Le Bateleur | Infatigable | No PS loss from PA damage (normally 1 PS per 5 PA lost in one hit). |
| II. La Papesse | Connaissance Secrète | One knowledge type (chosen at gen): difficulty reduced by 1 level for Savoir tests. |
| V. Le Pape | Forteresse Spirituelle | +5 max PEs at creation. |
| VII. Le Chariot | Sûr de Soi | +1 auto-success on all Parole tests vs humans. |
| VIII. La Justice | Bon Sens |  |
| IX. L'Ermite | Esprit d'Acier | All PEs losses reduced by 1 (minimum 0). |
| X. La Roue | Chanceux | Once per mission: reroll all dice on one failed test. |
| XI. La Force | Dur à Cuire | +5 max PS at creation. |
| XIII. L'Arcane | Trompe la Mort | On Mort injury result: reroll, take next result. Once per mission. |
| XIV. La Tempérance | Guérison Rapide | Healing recovery doubled. Nod Soin: +3 extra PS recovered. |
| XV. Le Diable | Instinct Animal | +10 Initiative during ambushes. Keep Def/React when surprised or flanked. |
| XIX. Le Soleil | Rayonnement | Allies within Courte range: +1 auto-success on all tests. Once per turn. |
| [v7] 0. Le Fou | Chevalier Véritable | Heroic Mode triggers at 4 Héroïsme (instead of 6). Total still capped at 6. Any malevolent act: lose 1 Héroïsme. |
| [v7] III. L'Impératrice | Mémoire Efficace | Once per mission: ask system for one lore hint about current encounter (enemy weak point, hidden path, etc.). |
| [v7] IV. L'Empereur | Magnétique | +1 auto-success on all Aura tests vs human NPCs. |
| [v7] VI. L'Amoureux | Aisance | Ignores the Lourd weapon effect. Still requires two hands to wield. |
| [v7] XII. Le Pendu | Code Moral |  |
| [v7] XVI. La Maison-Dieu | Soif d'Apprendre |  |
| [v7] XVII. L'Étoile | Rêves Prémonitoires | Once per mission: receive a cryptic hint about the current mission node (enemy type, trap, or objective clue). |
| [v7] XVIII. La Lune | Menteur Professionnel | All Parole combo Discrétion tests: difficulty reduced by 1 level. |
| [v7] XX. Le Jugement | Empathie | +1 auto-success on Instinct tests to read emotions/detect lies. Ignore crit fail on those tests. |
| [v7] XXI. Le Monde | Créateur-né | [v8.3 H-2 FIX] +1 additional Minor Motivation (Réaliser une Œuvre) — Tag: VOEU_MONDE_ART. Trigger condition: NEVER in demo. The Art system is not part of the demo and requires dedicated design for the full game. In Knight lore, Art is a surviving fragment of the Ether and Light — a source of hope that can counter Despair. The Créateur-né knight is one of the few remaining artists. In demo: this motivation exists in the data model, takes a 4th minor motivation slot, but provides no PEs recovery. The knight effectively has 3 active minor motivations + 1 dormant slot. This is the intended cost/benefit: Le Monde grants strong stat bonuses (+1 Chair, +3 Chair chars) but one wasted motivation slot until Art is implemented. FUTURE: when Art system is designed, this motivation will trigger on art creation/performance events and become one of the most powerful PEs recovery tools in the game. |


### 24.3 Disadvantages

| Tarot | Disadvantage | Mechanic |
| --- | --- | --- |
| I. Le Bateleur | Colérique |  |
| II. La Papesse | Curiosité Maladive | Once per mission: system forces knight to investigate a non-priority target (1D6 turns distracted). |
| V. Le Pape | Fanatique | Roleplay-driven. Digital: once per mission, knight may refuse to execute a tactically optimal order. |
| VII. Le Chariot | Ennemi Juré | Narrative: a recurring enemy NPC targets this knight specifically. +1 enemy in one random encounter. |
| VIII. La Justice | Trop Prudent | Initiative roll: 2D6 instead of 3D6 (+ bonuses). |
| IX. L'Ermite | Solitaire | Assist tests involving this knight: difficulty +1 level. |
| X. La Roue | Mauvaises Intuitions | Once per mission: one Instinct/Perception test auto-fails (system-triggered). |
| XI. La Force | Forcené | Once per mission: system forces knight to attack nearest enemy (no tactical choice). |
| XIII. L'Arcane | Vétéran | Aspect scores cannot exceed 7 (normally cap at 9). |
| XIV. La Tempérance | Immunité Déficiente | Poison/toxin damage: always maximum (no roll, take max face values). |
| XV. Le Diable | Brute | Machine aspect cannot exceed 5. |
| XIX. Le Soleil | Egoiste | [v8.4 M-2 FIX] Egoiste: Knight cannot serve as Adjuvant during Mode Héroïque (SPEC-09). Knight cannot use the Assist action on allies (SPEC-04). Self-centered — refuses to subordinate to others or support them directly. |
| [v7] 0. Le Fou | Trouble Mental | Knight has a phobia or compulsion (random at gen). Once per mission: system triggers phobia/compulsion, knight loses 1 combat action (frozen/distracted). |
| [v7] III. L'Impératrice | Esprit de Contradiction | Once per mission: system forces knight to contest an ally's action. Target ally's next test: +1 difficulty level. |
| [v7] IV. L'Empereur | Présomptueux | Once per mission: system forces knight to attack highest-tier enemy present (boss/colosse over hostile), ignoring tactical priority. |
| [v7] VI. L'Amoureux | Graveleux | All Parole and Aura tests vs specific NPC types (random at gen): +1 difficulty level. |
| [v7] XII. Le Pendu | Sacrifice Total | [v8.3 H-2 FIX] Knight has no Major Motivation at character creation and cannot gain one during the game. SPEC-02 step 6: skip major motivation draw entirely (majorMotivation = null). Minor motivations function normally (+1D6 PEs per trigger). The knight permanently lacks the +25 PEs current+max bonus that other knights can earn. This is the price of the Code Moral advantage (4th minor motivation). |
| [v7] XVI. La Maison-Dieu | Amnésique | Knight's background is hidden. Advantages from other cards may be randomly reassigned by system at generation. One advantage is replaced with a system-chosen one. |
| [v7] XVII. L'Étoile | Cauchemars | On rest/heal: roll 1D6. Odd = nightmare, no PS recovery from rest, lose 1 PEs. |
| [v7] XVIII. La Lune | Lunatique |  |
| [v7] XX. Le Jugement | Prisonnier |  |
| [v7] XXI. Le Monde | Porte-malheur | Once per mission: system converts one failed test (by this knight or any ally) into a critical failure. |


### 24.4 Aspect Bonuses per Card

| Tarot | Aspect +1 | Characteristic Points |
| --- | --- | --- |
| I. Le Bateleur | Dame | +3 in Dame chars |
| [v7] 0. Le Fou | (none) | +6 Char pts free [v8.4 L-4 FIX] (distribute using armor class weights from SPEC-02 Weighted Distribution Table across ALL 15 characteristics, not aspect-locked. Each point rolled independently against full 15-char weight table, normalized. Characteristic scores still capped at parent Aspect score — if at cap, redistribute weight among remaining candidates) |
| [v7] III. L'Impératrice | Machine | +3 in Machine chars |
| [v7] IV. L'Empereur | Dame | +3 in Dame chars |
| [v7] VI. L'Amoureux | Bête | +3 in Bête chars |
| [v7] XII. Le Pendu | Chair | +3 in Chair chars |
| [v7] XVI. La Maison-Dieu | (none) | +2 Aspect pts free [v8.4 L-4 FIX] (pick 2 random aspects from knight’s 2 primary aspects per SPEC-02 Slot Allocation table, may repeat. Apply +1 to each selected aspect. NO Characteristic point distribution — this card raises ceilings only. Intentional and unique to La Maison-Dieu) |
| [v7] XVII. L'Étoile | Masque | +3 in Masque chars |
| [v7] XVIII. La Lune | Masque | +3 in Masque chars |
| [v7] XX. Le Jugement | Dame | +3 in Dame chars |
| [v7] XXI. Le Monde | Chair | +3 in Chair chars |
| II. La Papesse | Masque | +3 in Masque chars |
| V. Le Pape | Machine | +3 in Machine chars |
| VII. Le Chariot | Bête | +3 in Bête chars |
| VIII. La Justice | Machine | +3 in Machine chars |
| IX. L'Ermite | Machine | +3 in Machine chars |
| X. La Roue | Bête | +3 in Bête chars |
| XI. La Force | Chair | +3 in Chair chars |
| XIII. L'Arcane | Masque | +3 in Masque chars |
| XIV. La Tempérance | Chair | +3 in Chair chars |
| XV. Le Diable | Bête | +3 in Bête chars |
| XIX. Le Soleil | Dame | +3 in Dame chars |


### 24.5 Generation Integration
[v6] During SPEC-02 step 4:
Draw 5 random cards from the 22-card Major Arcana pool (no duplicates within the same knight's draw).
Apply Aspect +1 and +3 Characteristic points from all 5 drawn cards.
Auto-select 2 advantages and 1 disadvantage from the 5 drawn cards ([v8] random selection for demo — pick 2 advantages and 1 disadvantage at random from the drawn cards. Player choice if manual generation is enabled).
Unused advantages/disadvantages from the remaining 2 cards are discarded (stat bonuses from all 5 still apply).
Cross-knight uniqueness: NOT enforced. Multiple knights may draw the same card. Each knight draws independently from the full 22-card pool.

### Acceptance Criteria

Trop Prudent: initiative = 2D6 + Masque bonus. Not 3D6.


[v6] Generation: 5 cards drawn from 22-card pool. All 5 apply stat bonuses (+1 Aspect, +3 Char pts each = +5 Aspects, +15 Char pts total). 2 advantages + 1 disadvantage selected from the 5.

## [v5] SPEC-25: Camelot Hub & Mission Loop
### Purpose
Defines the between-mission hub screen, available player actions, the Points de Gloire (PG) economy, and the full campaign loop. [v5] Resolves M2, M3, M7.

### 25.1 Campaign Loop


CAMELOT HUB (between missions)
  ├─ Review Roster (8 knights, view stats/injuries/equipment)
  ├─ Spend PG (implants, therapy, future: weapons/modules)
  ├─ Select Mission (demo: 1 mission available)
  ├─ Select Squad (choose 4 of 8 knights)




  [Loop until campaign end: all 8 knights dead OR final mission cleared]

### 25.2 Camelot Hub Actions

| Action | Description | Cost |
| --- | --- | --- |
| View Roster | See all 8 knights: PS/PA/PE/PEs, injuries, equipment, armor class, abilities. | Free |
| Auto-Refill Ammo | Grenades (5), Nods (3 each type), Chargeur weapons: all reset to max. | Free (auto) |
| Cybernetic Implant | Remove one injury from a knight. [v8.8 M-4 FIX] Each implant permanently reduces maxPEs by 3 (tabletop canon). Max 6 implants per knight. Minimum maxPEs floor: 10 — if an implant would reduce maxPEs below 10, block the implant. |  |
| Reconstruction Therapy | Remove ALL injuries from a knight. Removes tattoos/scars. | 100 PG. |
| Select Mission | Choose next mission from available list (demo: 1 mission). | Free |
| Select Squad | Choose 4 knights to deploy. Final once mission starts. | Free |
| Memorial | View fallen knights. Flavor/narrative only. | Free |


### 25.3 Points de Gloire (PG) Economy
PG is the meta-currency earned through play and spent at Camelot.

Earning PG:

| Source | PG Earned | Notes |
| --- | --- | --- |
| Complete mission (success) | 30 PG | Base reward |
| Complete mission (all knights alive) | 10 PG bonus | Survival bonus |
| Kill Patron (boss) | 15 PG | Per boss defeated |
| Kill Colosse | 5 PG | Per colosse defeated |
| Mode Héroïque activation | 5 PG | Per activation (SPEC-09) |
| Exploit roll | 1 PG | Per Exploit achieved in combat |
| Minor Motivation fulfilled | 3 PG | Per motivation (SPEC-12) |
| Major Motivation fulfilled | 10 PG | Once per mission if triggered |


Spending PG:

| Purchase | PG Cost | Notes |
| --- | --- | --- |
| Cybernetic Implant | 20 PG |  |
| Reconstruction Therapy | 100 PG | Removes ALL injuries. Resets implant count to 0. |
| Purchase Standard Weapon | Weapon PG value | See SPEC-19 (e.g. Fusil de Précision = 40 PG) |
| Purchase Module (future) | Module PG value | Not in demo scope. Scaffolded. |
| Armor Evolution (future) | 150–250 PG | SPEC-17 evolution tiers. Not in demo scope. |


### 25.4 Campaign End Conditions

| Condition | Result |
| --- | --- |
| All 8 roster knights dead | CAMPAIGN OVER. Show memorial screen. Offer New Campaign. |
| Final mission cleared | CAMPAIGN VICTORY. Show victory screen with statistics. |
| Player quits | Save state. Resume from Camelot Hub. |


### 25.5 State Persistence at Camelot
The following state persists between missions:
Knight HP/PA/PE: Fully restored to max at Camelot (free healing between missions).
PEs: [v8] No recovery at Camelot in demo. PEs carry over between missions unchanged. Future content will add Camelot rooms for PEs support (e.g. psychologist). Motivation bonuses earned during mission carry over as permanent PEs gains.
Injuries: Persist until treated (implant or therapy).
Heroism: Resets to 0 at mission start.
Equipment: Persists. New purchases added to inventory.
Dead knights: Permanently removed. No resurrection.
PG: Running total. Carries over indefinitely.

### Acceptance Criteria
Mission success with 4 alive, 1 Exploit, boss killed, minor motivation: 30 + 10 + 15 + 1 + 3 = 59 PG.

All 8 dead: campaign over screen. No further play possible.
[v8.2 CR-1 FIX] Return to Camelot: PS/PA/PE fully restored. PEs unchanged (carry over between missions as-is). Future content: psychologist room for PEs recovery. Injuries persist.
Ammo auto-refill: grenades 5, nods 3 each, chargeur weapons at max.



[v7] SPEC-26: UI Screen Flow (High-Level)
[v7] Minimum Viable Screen Flow for Agent Implementation:

Combat HUD requires: Initiative timeline (top), Knight stat bars (bottom), Rank display (center), Action menu (context), Status effects (per-knight icons), Tarot hand (collapsible panel).
Deployment Screen: 4 rank slots, drag-and-drop knight placement, cover indicators, enemy preview.
Camelot Hub: Armory (buy/sell weapons+modules), Knight roster (stats, injuries, equipment), Mission select, Tarot management.
Agent Note: Full UI wireframes are out of scope for this spec. The above flow defines the minimum screen set. UI layout decisions are delegated to the UI agent with the constraint that all SPEC-referenced UI elements (HUD countdown, initiative timeline, Tarot hand) must be present.
[v7] SPEC-27: Audio/VFX Event Hooks
[v7] Placeholder event trigger points for the combat chain and state transitions. Agents must fire these events at the specified moments. Actual audio/VFX assets are out of scope for this spec.
COMBAT EVENTS:
  OnAttackRoll(attacker, target, successes) — after dice roll resolved
  OnHitConfirmed(attacker, target, damage) — attack exceeds Defense/Reaction
  OnMiss(attacker, target) — attack fails to exceed threshold
  [v8.3 L-1 FIX] OnExploit(attacker, target, rerollSuccesses) — ALL dice show even on attack roll (SPEC-04 special outcome). Fired after re-roll resolves.
  [v8.7 M-3 FIX] OnCriticalHit removed — duplicate of OnExploit(attacker, target, rerollSuccesses) which already fires on all-even dice rolls.
  OnDamageApplied(target, amount, layer) — after CdF/PA/PS reduction
STATE TRANSITION EVENTS:
  OnFoldTriggered(knight) — PA reaches 0, Guardian Suit activated (SPEC-10)
  OnFoldEnded(knight) — Fold state exits
  OnAgonyEntered(knight) — PS reaches 0, injury roll triggered
  OnInjuryRolled(knight, injuryType, bodyPart) — injury table result applied
  OnHémorragieStart(knight, turnsRemaining) — bleeding countdown begins
  OnHémorragieTick(knight, turnsRemaining) — each turn countdown
  OnPermadeath(knight) — permanent death, remove from roster
ENEMY/BOSS EVENTS:
  OnPhaseTransition(boss, fromPhase, toPhase) — boss PS depleted, phase swap
  OnDébordement(bande, targetRank) — bande overflows to knight rank
  OnBandeEliminated(bande) — cohesion reaches 0
  OnPeurTest(knight, peurLevel, result) — fear test resolved
MISSION EVENTS:
  OnNodeEntered(nodeIndex, nodeType) — new mission node started
  OnBranchChosen(nodeIndex, branchId) — player selects branch at Node 2
  OnMissionComplete(success, knightsAlive) — final node resolved
  OnKnightDeployed(knight, rank) — placement during deployment phase
ESPOIR EVENTS:
  OnEspoirGained(knight, amount, source) — PEs increased
  OnEspoirLost(knight, amount, source) — PEs decreased
  OnMotivationTriggered(knight, motivationType) — motivation condition met
  OnHeroicModeActivated(knight) — 6 Héroïsme spent, heroic mode entered
[v8] DESPAIR EVENTS:
  OnDespairEntered(knight, duration) — PEs reaches 0, hostile AI begins (SPEC-07)
  OnDespairTick(knight, turnsRemaining) — Despaired knight's turn ends, countdown decremented
  OnStabilizationAttempt(stabilizer, target, success) — ally rolls to end Despair
  OnDespairExited(knight, reason) — Despair ends (stabilized / PEs restored / defeated)
  OnDespairPermanent(knight) — duration expires, Despair becomes permanent hostile AI
Agent Note: All events are fire-and-forget. No gameplay logic depends on event listeners. Events exist solely for presentation layer (audio, VFX, screen shake, UI animations).
End of Technical Specification v8.8. 32 SPEC sections + SPEC-02.5 (Blason) + SPEC-02.8 (Haut Fait) + SPEC-18.12 (Hybrid AI) + Issue Tracking. ALL v7 through v8.8 findings RESOLVED. Document is agent-ready.
CHANGELOG
v8.8 (2026-02-26) — Pass 8 Deep-Dive Validation + Remediation: HIGH: H-1 Complete injury table digital effects filled for all 24 entries (19 were empty). Tabletop-accurate characteristic reductions with specific Aspect/characteristic targets, derived value recalculation triggers. All effects from Livre de Base p.91. MEDIUM: M-1 PEs dice penalty canonical formula added: pesPenalty = max(0, 10 − currentPEs), stated in SPEC-04 and SPEC-07. M-2 Style gate corrections: Puissant requires Lourd Contact weapon, Pilonnage requires Deux Mains Ranged weapon. SPEC-20 Lourd/Deux Mains definitions updated. M-3 Demo weapon availability note: Puissant scaffolded (no Lourd weapons in demo), Pilonnage active (Fusil de Précision, Fusil d'Assaut, Shotgun Escamotable). M-4 Cybernetic implant PEs penalty: −3 maxPEs per implant, max 6, floor 10. M-5 Hémorragie Interne linked to standard 3-turn countdown (SPEC-08). LOW: L-1 Cerveau Touché clarified (12 chars = Bête+Machine+Dame+Masque, action loss is knight's choice). L-2 HautFaitData struct standalone with ConditionType enum. L-3 Mise à couvert prose (Dispersion/Débordement/Cover interactions). L-4 Phase 2 AI change verified in SDT (false positive). Total: 81 issues across 8 reviews, ALL RESOLVED.

v8.7 (2026-02-26) — Pass 7 Deep-Dive Validation + Remediation: CRITICAL: C-1 maxPS formula corrected from flat 10 to canonical tabletop formula (10 + 6 × Max(Force,Endurance,Déplacement) [OD excluded]). HIGH: H-1 IsSquadDefeated canonical code fixed — temp Despair (isDespairPermanent==false) no longer triggers squad defeat. H-2 MN templates verified — all 10 present in SDT tags (false positive). MEDIUM: M-1 Injury recalculation AC corrected (Chair→maxPS, Bête→Defense, Machine→Reaction, Masque→Initiative). M-2 Stray } and empty paragraphs removed near IsSquadDefeated. M-3 OnCriticalHit removed (duplicate of OnExploit). M-4 TriggerEvent ev removed from EncounterTrigger (belongs in TriggerCondition only). M-5 Combat style table filled (Agressif/Défensif/Puissant/Pilonnage modifiers + restrictions). LOW: L-1 Haut Fait table completed (14 entries with conditions, callsigns, weighted dual-Aspect resolution per Armor Class primary aspects). L-2 isInDespair standardized to isDespair across all schemas. L-3 despairDuration field added to CombatantState. L-4 Ghost activation cost clarified. Total: 71 issues across 7 reviews, ALL RESOLVED.

v8.6 (2026-02-25) — Structural Remediation + Expert Review Pass 6: CRITICAL: C-1 Duplicate SPEC-01 merged (original runtime schema retained, enhanced with derived value pseudocode, Aspect/Characteristic reference, SPEC-01-AC1→AC4). HIGH: H-2 IsSquadDefeated stub removed (canonical implementation only). MEDIUM: M-1 Hash Algorithm block moved outside IRng interface. M-2 TriggerCondition/TriggerAction schemas moved outside EncounterTrigger struct with cross-reference note. M-3 SPEC-30-AC3 filled (Despair temporary vs permanent defeat evaluation). LOW: L-1 Part 3 intro corrected from "OPEN" to audit trail. Total: 59 issues across 6 reviews, ALL RESOLVED.

v8.5 (2026-02-24) — Agent Handoff Addendum + v8.5 Expert Review: Part 4 Addendum (SPEC-28→32). Expert Review 3H+5M+4L resolved: H-1 SPEC-01 KnightBase standalone section. H-2 xxHash32 mandatory hash, Xoshiro128** PRNG, uint seed types. H-3 canonical IsSquadDefeated(). M-1 debordementScore aligned. M-2 countdown timers. M-3 Nocte vs Shrine worked example. M-4 state machine updated. M-5 version header. L-1 TriggerCondition/TriggerAction schemas. L-2 Combat State Isolation. L-3 hémorragieCountdown. L-4 SPEC-22/29 cross-ref. Total: 56 issues across 5 reviews, ALL RESOLVED.

v8.4 (2026-02-24) — v8.4 Expert Review: Edge Cases, Parameters, Polish

HIGH: H-1 Le Cerf tracking contradiction fixed (neverMovedFlag tracking replaces ambiguous final-rank comparison). MEDIUM: M-1 Prisonnier PEs freeze scope clarified (minor motivations only, other sources function normally). M-2 Le Soleil disadvantage corrected from Egoiste to Egoiste (cannot be Adjuvant, cannot Assist allies). M-3 Node 4 Point Faible discovery probability set (1D6, triggers on 6 only — intentionally rare). M-4 Phase 2 Nocte bande respawn fully specified (fresh spawn if destroyed, AddCohesion if alive). M-5 MJ-06 post-liberation flow corrected (Prisonnier disadvantage removed, new major motivation drawn at Camelot). LOW: L-1 Le Dragon removed from demo Blason table (roll scope ambiguous, deferred to full game; 9 demo Blasons remain). L-2 Node 2 branch choice UX specified (title + narrative + star rating, no enemy preview). L-3 Le Pendu commandment/Blason Voeu overlap confirmed intentional with design note. L-4 Le Fou (+6 free char pts via full 15-char weighted distribution) and La Maison-Dieu (+2 Aspect pts only, no char distribution) documented.

v8.3 (2026-02-24) — v8.3 Expert Review: Blason System, Tarot Special Cases, Hybrid AI
HIGH: H-1 Complete Blason system with 10 demo Blasons, Voeu event tags, uniqueness rule, data model (SPEC-02.5). Le Cheval and Le Corbeau deferred to full game. H-2 Tarot special cases resolved: Le Pendu advantage “Code Moral” with 6 feasible Code d’Honneur commandments (1 random per mission); Le Pendu disadvantage “Sacrifice Total” clarified (no major motivation ever); Le Jugement disadvantage “Prisonnier” with micro-bomb (PEs < 10 = permadeath), frozen PEs gain, MJ-06 “Retrouver la Liberté” (2+1D3 missions, visible progress); Le Monde advantage “Créateur-né” as Art system stub (VOEU_MONDE_ART, never triggers in demo).

LOW: L-1 OnExploit event description corrected (“ALL dice even” not “6+ excess successes”). L-2 Version header updated to v8.3. L-3 Changelog updated with v8.2 + v8.3 entries.
v8.2 (2026-02-24) — v8.2 Expert Review: Medium + Low Priority Fixes
CRITICAL: CR-1 PEs restoration at Camelot contradiction resolved (PS/PA/PE restored, PEs unchanged). CR-2 Meta-Armor Base Stats Table added to SPEC-02.7 with full armor data (PA, PE, CdF, Gen, Slots, Base Overdrives).

MEDIUM: M-1 Module vs innate ability separation clarified. M-2 Complete Haut Fait table (14 entries from tabletop source). M-3 Motivation Template Pool with 10 minor + 5 major templates (SPEC-12.1). M-4 Ghost detection terminology standardized (automatic successes, not extra dice). M-5 Points de Contact future design note.
LOW: L-1 Agent-Readiness Scorecard refreshed to v8.2 values (9.5/10). L-2 Firing beyond range simplified (out-of-range = greyed out, no penalty system). L-3 Ranger column preserved as next planned expansion class.
v8.1 (2026-02-23) — v8 Expert Review Remediation
CRITICAL: CR-1 Pénétrant CdF double-subtraction bug (SPEC-06). CR-2 Priest weight column added, Ranger kept as future class (SPEC-02). CR-3 Exploit added as SPEC-05 excess successes exception. CR-4 Global Math Rules added (round down, divide-then-subtract, successive doubling=tripling). CR-5 OnFoldTriggered corrected to PA=0 (SPEC-27).
HIGH: H-1 Bande attack procedure added, violence-only damage (SPEC-11). H-2 ANTI_VEHICULE future-proofing note. H-3 Despair events added to SPEC-27. H-4 Barrage targeting constraints. H-5 Charge Brutale routes through ArmourLayerResolver. H-6 Anathème to-hit procedure clarified.

LOW: L-1 AC numbering convention note. L-2 Flashbang 0-damage clarified (effects only). L-3 XIII+XVI incompatibility check added to SPEC-02 generation. L-4 PEs do NOT recover at Camelot in demo. L-5 Enemy Exploit = bonus damage dice (same as knight Exploit).
v8.0 — Despair, Weighted Distribution, Grenade Resolution, Barrage Rewrite
Added SPEC-07 Despair system (trigger, hostile AI, stabilization, exit, permanence). Armor Class Weighted Distribution Table (SPEC-02). Grenade resolution procedure. Barrage effect rewritten as suppress-mode alternative. Knight Defense vs Anathème chain. PEs Recovery on Anathème Kill. Global Duration Definition.
v7.0 — v7 Expert Review: 20 findings resolved
5 Critical fixes (Hit Threshold, Destructeur/Meurtrier Pseudocode, Faune Majeur auto-successes, Nocte Bande targeting, NanoC Cover). 15 additional findings (H/M/L) resolved. Full issue tracking in Part 3.
v1.0–v6.0 — Initial specification + iterative development
v1: Initial 27 SPEC framework. v2: DD positioning, Rogue Ghost, enemy AI. v3: MDD findings integration. v4: Nod/Grenade, Combo Roll, DD rank system. v5: Expert review integration (SPEC 18–25). v6: Tarot 22-card pool, 5-card draw, Haut Fait correction.

# PART 3 — v7 EXPERT REVIEW: ISSUE TRACKING
Independent expert review of Technical Specification v6. 20 issues identified across 25 SPEC sections. Organized by severity using the same scale as Part 1. All 20 issues were identified and resolved in v7. This section serves as an audit trail.

## 3.1 Issue Summary
20 findings (5C + 5H + 5M + 5L). ALL 20 RESOLVED in v7. 0 remain OPEN. Document is agent-ready. Critical and High findings must be addressed before development begins. Medium findings should be resolved during Phase 1-2 implementation. Low findings can be deferred to polish.


| Severity | Count | Resolution Requirement |
| --- | --- | --- |
| CRITICAL | 5 | ALL 5 RESOLVED in v7. No longer blocking. |
| HIGH | 5 | Cross-system ambiguity. Will cause subtle bugs if unresolved. |
| MEDIUM | 5 | Requires agent assumptions. May differ from design intent. |
| LOW | 5 | Polish, edge cases, missing hooks. Non-blocking. |




| v7-C1 | Affected: SPEC-04 (§5.1 step 8), SPEC-18.9 (Step 6). Impacts every combat roll in the game. Recommendation: |
| --- | --- |



| v7-C2 | Affected: SPEC-06 (Armour Layer Resolver), SPEC-20.1 (Destructeur/Meurtrier definitions), SPEC-05 (Damage Resolver step 8). Recommendation: Rewrite SPEC-06 as an explicit 8-step pseudocode function: (1) Apply CdF, (2) Apply Bouclier, (3) CHECK Destructeur trigger — if remaining > 0 AND target has PA: roll +2D6, add to remaining, (4) Apply PA absorption, (5) CHECK Meurtrier trigger — if remaining > 0 after PA: roll +2D6, add to remaining, (6) Apply PS damage. Include worked example with both effects on same hit. |
| --- | --- |



| v7-C3 | SPEC-18.3 defines Majeur (N) as ‘N automatic successes’ (flat bonus to success count, not extra dice). SPEC-21.3 Note states: ‘Bête Majeur (6) means +6 dice offensively.’ These are mechanically different — +6 auto-successes is guaranteed, while +6 dice yields ~3 successes on average. The tabletop uses automatic successes. This error would halve the Faune’s effective offensive power if an agent follows the Note instead of SPEC-18.3. Affected: SPEC-21.3 (Faune stat block note), SPEC-18.3 (Aspects Exceptionnels rules). Recommendation: Correct the Note in SPEC-21.3 to read: ‘Bête Majeur (6) means +6 automatic successes offensively AND incoming attacks using Bête-based characteristics deal half damage.’ Remove the word ‘dice’. |
| --- | --- |



| v7-C4 | SPEC-21.1 (Nocte stat block) lists ‘Weapon (AoE): Anti-véhicule damage’ and includes a full AI profile (targeting: NEAREST_RANK, aggression: 3). SPEC-11 states unambiguously: ‘Bandes do not attack and never roll to hit. Débordement is the Bande’s ONLY offensive mechanic.’ An agent encountering these contradicting instructions will either implement attack logic for Nocte (violating SPEC-11) or ignore the weapon entry (losing the Anti-Véhicule trait). The key question: does Anti-Véhicule apply to Débordement damage? Affected: SPEC-21.1 (Nocte stat block), SPEC-11 (Bande Controller). Recommendation: Remove the ‘Weapon’ and ‘AI’ fields from the Nocte stat block. If Anti-Véhicule is intended to apply to Débordement damage, add an explicit ‘Débordement Tags’ field (e.g., debordementTags: [ANTI_VEHICULE, IGNORE_CDF]) and update SPEC-11 to check for Bande-level tags when applying Débordement. |
| --- | --- |



| v7-C5 | Affected: SPEC-17.3 (NanoC Menu), SPEC-03 (Cover System), SPEC-20.4 (Tir en Sécurité, Artillerie). Recommendation: |
| --- | --- |




| v7-H1 | SPEC-01 Derived Value Formulas contains an unresolved decision marker: ‘DECISION REQUIRED: OD included for Defense/Reaction/Initiative means add OD of only the highest Characteristic (not all Aspect ODs). Recommended default: single highest + its OD.’ This is the most fundamental combat formula in the game. Every attack roll, every defense check, every initiative calculation depends on it. An agent cannot implement combat without this resolved. Affected: SPEC-01 (KnightBase Derived Values). Propagates to SPEC-04, SPEC-05, SPEC-15, SPEC-18. Recommendation: Resolve immediately. The tabletop confirms: Defense = highest BÊTE characteristic + that characteristic’s OD. Same pattern for Reaction (MACHINE) and Initiative (MASQUE). Remove the DECISION REQUIRED tag and state the formula definitively. |
| --- | --- |



| v7-H2 | SPEC-17.2 Shrine grants +6 CdF to ‘all inside (stacks). 6m dome.’ The DD rank system has no concept of spatial distance or ‘inside a dome.’ An agent must know: does Shrine affect all allies at the same rank as the Paladin? All allies within Courte range? All 4 allies regardless of rank? The entry also says ‘Immobile — Cannot move once placed’ but ranks are abstract positions, not physical locations. Affected: SPEC-17.2 (Shrine), SPEC-03 (Position System). Recommendation: |
| --- | --- |



| v7-H3 | Affected: SPEC-18.10 (Peur rules), SPEC-21.4 (Behemot), SPEC-21.5 (Ours Corrompu). Recommendation: |
| --- | --- |



| v7-H4 | Affected: SPEC-14 (Boss Phase Controller), SPEC-21.5 (Ours Corrompu stat blocks), SPEC-22 (Mission Structure boss encounter). Recommendation: |
| --- | --- |



| v7-H5 | SPEC-21.2 Bestian Anathème capacity says: ‘Attacker may choose to inflict damage on PEs instead of PS. CdF still applies, PA does not.’ The tabletop has the MJ (attacker/GM) decide. In a digital game with no GM, the AI must have a deterministic decision rule. Should the AI always target PEs? Only when PEs < threshold? Target whichever is lower? Without this rule, an agent will either hardcode an arbitrary choice or leave it undefined. Affected: SPEC-21.2 (Bestian), SPEC-18.7 (EnemyAI profile), SPEC-21.5 (Ours Corrompu also has Anathème). Recommendation: Add an ‘anathemeTarget’ field to EnemyAI: enum {ALWAYS_PES, ALWAYS_PS, PREFER_LOWER, SMART}. Default Bestian to PREFER_LOWER (target whichever pool is lower to maximize pressure). Ours Corrompu Souffle ténébreux: ALWAYS_PES (narrative: darkness attacks hope). |
| --- | --- |




| v7-M1 | SPEC-24.1 states that only 12 of 22 Major Arcana cards have complete mechanical definitions. The remaining 10 ‘provide stat bonuses only with no advantage/disadvantage.’ SPEC-02 step 4 requires selecting 2 advantages and 1 disadvantage from 5 drawn cards. If a knight draws 3+ undefined cards, they cannot fill all required slots. No fallback rule exists. Affected: SPEC-24 (Tarot), SPEC-02 (Generation Pipeline step 4). Recommendation: Add fallback rule: ‘If fewer than 2 defined advantages or 1 defined disadvantage are available from drawn cards, redraw from the 12-card defined pool until slots are filled. Stat bonuses from all 5 original draws still apply.’ Or: complete the remaining 10 card definitions before demo ship. |
| --- | --- |



| v7-M2 | SPEC-18.9 states enemies defend with ‘Aspect/2 (round down) + Exceptionnel bonus’ and that this is the ‘pre-calculated Defense/Reaction in the stat block.’ Bestian has Bête 8, Bête Mineur (2), listed Defense 4. If Defense = Bête/2 + Mineur = 4+2 = 6, but the listed value is 4 (= Bête/2 only). Either the listed values exclude Exceptionnel (and auto-successes are added at roll time), or the formula description is wrong. An agent will double-count if the stat block already includes the bonus. Affected: SPEC-18.9 (Enemy Attack Procedure Steps 5-6), SPEC-21 (all enemy stat blocks). Recommendation: Clarify that listed Defense/Reaction values in stat blocks represent Aspect/2 only. Exceptionnel auto-successes are added at attack roll time (Step 5), NOT baked into the stat block threshold. Add a note: ‘Stat block Defense/Reaction = base opposition score. Exceptionnel bonuses add to attacker’s success count, not to the stat block value.’ |
| --- | --- |



| v7-M3 | The spec covers game mechanics exhaustively but provides no screen flow diagram or UI state machine. An agent building UI needs to know: deployment screen layout, initiative timeline display format, injury/status indicator placement, branch selection (Node 2) presentation, combat HUD layout, Camelot Hub screen organization. Without this, the UI team will make assumptions that may conflict with gameplay requirements. Affected: All SPECs (UI layer), SPEC-25 (Camelot Hub), SPEC-03 (Deployment Phase), SPEC-15 (Turn Queue), SPEC-22 (Mission Structure). Recommendation: |
| --- | --- |



| v7-M4 | Affected: SPEC-04 (Action Economy Table), SPEC-15 (Turn Structure). Recommendation: Add an explicit design note: ‘Assist range (any) vs Nod range (Courte) is intentional per tabletop rules. Assistance is a morale/mental action; Nods require physical proximity to administer.’ This prevents agent ‘correction’ and informs UI tooltip text. |
| --- | --- |



| v7-M5 | SPEC-18.7 defines TargetPriority and RankBehavior enums but provides no step-by-step decision tree. Agent needs: Does enemy move first then attack, or attack first? If ADVANCE but target already in range, does it still advance? If HOLD but no target in range, does it break HOLD? If multiple targets match priority, which wins? If FLANK, what exact behavior occurs in the 4-rank system? Without a flowchart, each agent developer will implement different AI behaviors. Affected: SPEC-18.7 (EnemyAI), SPEC-21 (all enemy stat blocks with AI profiles). Recommendation: Add Section 18.11: Enemy AI Decision Flowchart. Pseudocode: (1) Check weapon range vs all valid targets, (2) If target in range: attack (select by TargetPriority), (3) If no target in range: execute RankBehavior to get closer, (4) If now in range: attack, (5) Else: hold. Define FLANK as ‘prefer targeting Rank 3-4 knights; move to Rank 1 if needed for contact range.’ |
| --- | --- |




| v7-L1 | SPEC-08 (Injury & Agony) lists ‘1,5 Hémorragie: 3-turn countdown.’ SPEC-23 (Complete Injury Table) listed ‘Hémorragie Interne: Dead in 10 turns unless healed.’ [v7] SPEC-08 is authoritative (3 turns). SPEC-23 updated to match. Affected: SPEC-08 (Special Injuries), SPEC-23.2 (Column 1, Row 5). Recommendation: Update SPEC-23 to match SPEC-08: ‘Hémorragie: 3-turn countdown.’ SPEC-08 is authoritative. |
| --- | --- |



| v7-L2 | SPEC-05 worked example (melee attack) describes Marteau-Épieu as ‘3D6 + Force, Lesté’ and calculates Force×2. The actual weapon profile in SPEC-19.3 shows Marteau-Épieu with effects: Perce Armure 40 (contact) — no Lesté. Either the worked example should use a different weapon, or the weapon profile is missing the Lesté tag. Affected: SPEC-05 (Worked Example), SPEC-19.3 (Marteau-Épieu stat block). Recommendation: Change the SPEC-05 worked example to use a hypothetical weapon ‘Masse d’Armes (3D6 + Force, Lesté)’ or add a note: ‘This example uses a Lesté variant for illustration. Default Marteau-Épieu does NOT have Lesté.’ |
| --- | --- |



| v7-L3 | Several numbered lists in the document have incorrect starting indices. For example, SPEC-02 Algorithm steps are numbered 5-14 instead of 1-10. Acceptance criteria across SPECs use inconsistent numbering ranges. While not functionally blocking, this creates confusion when referencing specific acceptance criteria by number during development and QA. Affected: Multiple SPECs (SPEC-02, SPEC-03, SPEC-04, SPEC-05 and others). Recommendation: Re-number all acceptance criteria sequentially across the entire document (AC-001 through AC-175) or restart numbering within each SPEC (e.g., SPEC-02-AC1 through SPEC-02-AC3). Use a consistent prefix to avoid ambiguity. |
| --- | --- |



| v7-L4 | The spec defines complete mechanical chains for combat, damage, injury, and state transitions, but includes no audio or visual effect trigger points. For combat feel (hit impacts, Exploit celebrations, Fold state alarm, Agony warning, Débordement rumble, Phase transition cinematic), the agent team needs at least placeholder event hooks in the damage chain and state transitions. Affected: SPEC-05 (Damage Chain), SPEC-06 (Armour Resolver), SPEC-08 (Agony), SPEC-10 (Fold), SPEC-14 (Boss Phase). Recommendation: Add a SPEC-27: Audio/VFX Event Hooks, listing all trigger points with placeholder event names (e.g., OnHitConfirmed, OnExploit, OnFoldTriggered, OnAgonyEntered, OnPhaseTransition, OnPermadeath). This allows the agent to fire events even if actual assets are not yet available. |
| --- | --- |



| v7-L5 | L1 (resolved) notes the tabletop difficulty gap at 7-8 is intentional, but no mapping exists for how digital encounters use difficulty thresholds. If enemies or narrative events trigger tests at specific difficulty levels (e.g., Peur tests, Perception checks to discover Point Faible), the agent needs to know which numeric threshold each difficulty label maps to in the Combo Roll system. Affected: SPEC-04 (skill tests), SPEC-18.10 (Peur), SPEC-22.3 (narrative events). Recommendation: Add a difficulty-to-threshold mapping table: Facile(1)=1 success, Normale(2)=2, Ardue(3)=3, Difficile(4)=4, Éprouvante(5)=5, Délicate(6)=6, Herculéen(9)=9. Include a note that 7-8 are unused in the demo. |
| --- | --- |



## 3.6 Agent-Readiness Scorecard

| Category | Score | Notes |
| --- | --- | --- |
| Completeness | 9 / 10 | [v8.8] 32 SPECs (single merged SPEC-01 KnightBase, SPEC-28–32 Agent Handoff Addendum). All 81 review findings across 8 review passes resolved. Injury table 100% complete (all 24 digital effects filled from tabletop source). Haut Fait table complete with standalone data struct.ighting. All 10 MN templates verified. Combat style table fully populated. |
| Consistency | 9.5 / 10 | [v8.8] All consistency issues resolved across 8 review passes. PEs penalty formula canonical (max(0, 10−PEs)). Style gate restrictions corrected (Puissant=Lourd, Pilonnage=Deux Mains). Implant PEs penalty documented. No stale references.CriticalHit removed). No redundant fields (TriggerEvent ev removed). |
| Implementability | 9.5 / 10 | [v8.8] Worked examples, pseudocode, and 195+ acceptance criteria. Every injury has deterministic digital effect. All combat styles have prose + edge cases. HautFaitData struct extractable. Demo weapon availability for each style documented.ambiguous. Combat styles fully specified with restrictions. All parameters specified. |
| Tabletop Fidelity | 9 / 10 | [v8.4] Faithful adaptation. 9 of 12 tabletop Blasons adapted for demo (Le Dragon deferred for scoping). Code d'Honneur mapped to 6 feasible commandments. Le Cheval/Le Corbeau/Le Dragon deferred pending dependent systems. |
| Agent-Readiness | 9.5 / 10 | [v8.7] 71 total issues across 7 review passes, ALL RESOLVED. No formula contradictions, no logic bugs in canonical code, no missing content. Combat styles fully specified. Haut Fait generation complete with weighted mechanics. Agent team can scaffold from Phase 1 dependency map with full confidence. No known blockers. |



End of Part 3 — v8.7 Expert Review. 20 original v7 issues (5C + 5H + 5M + 5L): ALL RESOLVED. v8.1 review (2C + 4H + 5M + 3L): ALL RESOLVED in v8.2. v8.3 review (2H + 5M + 3L): ALL RESOLVED in v8.3. v8.4 review (1H + 5M + 4L): ALL RESOLVED in v8.4. v8.5 review (3H + 5M + 4L): ALL RESOLVED in v8.5. v8.6 review (1C + 2H + 3M + 3L): ALL RESOLVED in v8.6. v8.7 review (1C + 1H + 5M + 4L): ALL RESOLVED in v8.7. v8.8 review (1H + 5M + 4L, 1 false positive): ALL RESOLVED in v8.8. Total issues tracked across all reviews: 81. All 81 RESOLVED. Document is agent-ready for development handoff.
Below is a drop-in “Agent Handoff Addendum” written in the same style as your SPEC sections. It’s designed to remove implementation drift for an AI agent team, especially around determinism, authoring formats, state modeling, UI “must show,” and Unity architecture.

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
  uint runSeed;       // generated once at run start (unsigned 32-bit, per v8.5 H-2)
  string missionId;
  string branchPathId; // stable hash of player choices (demo: node branch choice)
  RNGPolicy policy;   // default DeterministicNodeSeed
}

struct NodeRngContext {
  int nodeIndex;      // 0..N
  string nodeId;      // stable authoring id
  int attemptIndex;   // increments only if policy == FullRNGOnRetry
  uint nodeSeed;      // derived
}

enum RngStreamId { Combat, Loot, Cosmetic }

interface IRng {
  uint NextUInt();
  int Range(int minInclusive, int maxExclusive);
  bool Chance01(float p);
}
```

[v8.5 H-2 FIX] Hash Algorithm (Mandatory)
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
  string encounterId;     // stable
  string missionId;
  int nodeIndex;
  EncounterType type;     // Combat / Boss / Event / Rest (demo: Combat+Boss)
  CoverLayout cover;      // per rank

  PlayerStartLayout playerStart; // optional restrictions (demo: free)

  List<EnemySpawnEntry> initialEnemies;  // max 4 on-track
  ReinforcementQueue reinforcement;      // optional
  BandeConfig bande;                    // optional off-track

  List<EncounterTrigger> triggers;      // optional minimal scripting
}

struct CoverLayout { bool r1, r2, r3, r4; }

struct EnemySpawnEntry {
  string enemyId;
  int startRank;          // 1..4
  bool startsInCover;     // if true, requires cover at that rank
  bool isElite;           // informational tag
}

struct ReinforcementQueue {
  List<string> enemyIdsInOrder;
  ReinforcementRule rule; // WhenSlotFree / OnTurnN / OnTrigger
  int turnN;              // used if OnTurnN
}

struct BandeConfig {
  bool enabled;
  string bandeId;        // for UI/log
  int cohesionMax;
  int cohesionStart;
  int debordementScore;  // flat integer per SPEC-11 (e.g. Nocte = 4). Damage = débordementTurn × debordementScore
  // plus any special flags already defined in SPEC-11
}

enum TriggerEvent { OnCombatStart, OnTurnStart, OnEnemyDeath, OnBossPhaseChanged }
enum TriggerActionType { SpawnReinforcement, ApplyEffect, ChangeCover, SwapAiProfile }

struct EncounterTrigger {
  // [v8.7 M-4 FIX] TriggerEvent ev removed — event type stored in TriggerCondition.triggerEvent only
  TriggerCondition cond;
  TriggerAction action;
}
```

[v8.6 M-2 FIX] The following structs define the fields used by EncounterTrigger.cond and EncounterTrigger.action:
[v8.5 L-1 FIX] Lightweight Generic Trigger Schemas:
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
enum ActionState { CanAct, Incapacitated }              // can take turn actions
enum ArmorState { Normal, Folded }                      // from SPEC-10

struct CombatantState {
  LifeState life;
  ControlState control;
  ActionState action;
  ArmorState armor;
  bool isDespair;           // derived from PEs system
  bool isDespairPermanent;  // if stabilized failed or per spec rule
  int despairDuration;      // [v8.7 L-3 FIX] -1 = not in despair, 1D6 initial, decrements per turn, 0 = permanent
  int hemorragieCountdown;   // [v8.5 L-3 FIX] -1 = inactive, 3..1 = turns remaining before permadeath (SPEC-08). Affects targeting priority.
}
```


### Rules
Dead: life == Dead
Agony: sets action = Incapacitated.
Despair: sets control = EnemyControlled (temporary or permanent as per SPEC-07/08), but does not automatically mean incapacitated unless explicitly stated elsewhere.
Fold: armor = Folded affects available actions/modules per SPEC-10, but does not change action unless another rule does.
Single authoritative defeat function used everywhere:
[v8.6 H-2 FIX] [v8.7 M-2 FIX] Single authoritative defeat function — canonical implementation below.

```csharp
(Canonical implementation — this is the ONLY defeat evaluation function, no other code path may determine squad defeat: bool IsSquadDefeated(List<CombatantState> knights) { var living = knights.Where(k => k.life == Alive); if (!living.Any()) return true; return living.All(k => k.action == Incapacitated || (k.isDespair && k.isDespairPermanent && k.control == EnemyControlled)); })
``` [v8.7 H-1 FIX] Temporary Despair (isDespairPermanent == false) does NOT count as defeated — the knight may still be stabilized. Only permanent Despair (duration expired, isDespairPermanent == true) counts toward squad defeat.
### Acceptance Criteria
SPEC-30-AC1: Every system queries CombatantState (not ad-hoc flags) to decide: can act, is ally/enemy, is targetable.
SPEC-30-AC2: Only one implementation of IsSquadDefeated() exists and is used by CombatManager.
SPEC-30-AC3: A squad with 2 living knights — one Incapacitated (Agony), one in permanent Despair (EnemyControlled) — evaluates IsSquadDefeated() == true. Same squad where the Despaired knight has despairDuration > 0 (not yet permanent): IsSquadDefeated() == false (the Despaired knight may still be stabilized).

## SPEC-31: UI Minimum Information Contract (Demo Scope)
### Purpose
Ensure a usable dice-heavy tactics UI without needing wireframes; prevents UI agents from omitting critical feedback.
### Dependencies
All combat & progression systems; especially SPEC-04/05/06/07/08/10/11/14/15.
### Mandatory UI Elements (Must Show)
Combat Screen
Roll Breakdown Panel (on demand, per action):
dice pool size, OD autos, number of evens, total successes, target (Defense/Reaction), hit/miss result
for damage: number of dice, summed total, flat bonuses, final damage, and per-layer absorption (CdF/PA/PS)
Combat Log (scrollable, last N events):
attacks, hits/misses, damage breakdown, effects applied/expired, PEs changes, Despair enter/exit, Agony, Fold
Intent/Target Clarity
selected target highlight, range/gap indication, cover indicator, blocked actions reason (“out of range”, “no line”, “folded”, etc.)
Resource HUD per combatant: PS / PA / CdF / PE / PEs with tooltips describing current penalties tier
State Badges: Folded, Agony, Despair (temporary/permanent), key buffs/debuffs
Enemy Intent Preview (minimum)
at least target + action type icon/name (exact numbers optional unless you want full telegraphing). [v8.5 M-2 FIX] Active Countdown Timers (mandatory on affected combatant): Despair (turns remaining before permanence), Hémorragie (turns to death — highest priority, red pulsing), Dégâts Continus (turns remaining + damage per tick), Choc/Parasitage (actions lost remaining). Display format: icon + number badge on combatant portrait. Hémorragie and Despair countdowns must be visible at ALL times, not just on hover — these are life-or-death timers
Mission Node Screen
shows remaining nodes, branch choice, party status summary (PS/PA/PE/PEs), and explicit warning that no auto-heal occurs between nodes (if that’s your rule).
Hub (Camelot)
shows injury list + treatment costs, current roster condition summary, and what resets vs persists.
### Acceptance Criteria
SPEC-31-AC1: Player can always answer: “Why did I miss?” and “Where did the damage go?” without guessing.
SPEC-31-AC2: Every combat action produces a log entry including roll result + damage breakdown.
SPEC-31-AC3: Any disabled action must display a reason string (not just greyed out).

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
### Recommended Project Structure (minimal)
Scripts/Core (state machine, services, RNG, logging)
Scripts/Combat (CombatManager, resolvers, effects, AI)
Scripts/Data (SO definitions)
Scripts/UI (views, presenters, widgets)
Scripts/Meta (hub, injuries, progression)
Assets/Data (SO instances: enemies, encounters, missions)
### Game State Machine (demo)
State transitions are explicit; no scene spaghetti required (single-scene recommended for demo). [v8.5 L-2 FIX] Combat State Isolation (Single-Scene Rule): Even in single-scene architecture, combat and hub operate in fully isolated state contexts. (1) CombatSession — created on Deployment entry, destroyed on combat exit. Owns: turn queue, all runtime CombatantState instances, active effects, RNG streams (SPEC-28), encounter runtime data. No reference to CombatSession may persist after combat exit. (2) HubSession — created on Hub entry, destroyed on Hub exit. Owns: shop state, UI navigation, treatment queue. (3) RunState (shared, persistent) — owns: knight roster (KnightBase[8]), inventory, PG total, injury list, mission progress, PEs values. This is the ONLY state that crosses the Hub↔Combat boundary. (4) Read-only contract during combat: RunState is snapshot-copied into CombatSession at combat start. Combat mutates the SESSION copy only. On combat exit, mutations are batched and applied back to RunState in a single atomic commit (ApplyCombatResults). This prevents partial state corruption on crash. (5) No direct cross-references between CombatSession and HubSession. Communication goes through RunState or the global state machine. SPEC-32-AC5: Destroying CombatSession after combat produces zero NullReferenceExceptions in Hub systems.
### Serialization (demo)
Save stores: RunSeed context, current node id, roster runtime state, inventory/ammo, injuries, hub progress.
No mid-combat serialization required (per your design); resume at node start.
### Acceptance Criteria
SPEC-32-AC1: Combat can run headless (no UI) via unit/integration tests using EncounterSO + roster state.
SPEC-32-AC2: UI does not reference or mutate internal combat collections directly (no List<T> exposed).
SPEC-32-AC3: Grep check: no UnityEngine.Random in gameplay assemblies.
SPEC-32-AC4: All state transitions pass through the state machine (no direct scene loads in combat logic).
