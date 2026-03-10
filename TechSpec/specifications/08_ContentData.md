# Content Data: Enemy Roster, Injury Table & Motivations
> **SPECs:** 12 (incl. 12.1), 21, 23
> **Domain:** `Assets/Data`
> **Dependencies:** [06_EnemySystem.md](./06_EnemySystem.md) (SPEC-18), [07_WeaponsAndEffects.md](./07_WeaponsAndEffects.md) (SPEC-19, SPEC-20), [04_ArmorAndState.md](./04_ArmorAndState.md) (SPEC-07, SPEC-08)
> **Depended on by:** [10_MissionAndHub.md](./10_MissionAndHub.md) (SPEC-22)
---
## SPEC-12: Motivation Event Detector
### Purpose
Listens to game events, checks Motivations for fulfillment. Primary PEs recovery. 
* On event: check all knights’ 3 minor Motivations against event tags.
* Match: +1D6 PEs. Fire MotivationFulfilled UI event.
* Major Motivation: +25 PEs current+max. Erased. New one at Camelot. 

**SPEC-12.1 — Motivation Template Pool** 
Each Motivation template has: id, displayText (French flavor), eventTag (matched by SPEC-12 event system), type (minor/major). At generation (SPEC-02 step 6): draw 2 unique minor templates + 1 major template. No duplicates within a knight. 3rd minor = Blason Voeu (always tag BLASON_VOEU, fulfilled by narrative mission events). At Camelot between missions: spent major is replaced by a new random draw from the pool (excluding the one just completed). 
**Minor Motivation Templates (10) — repeatable, +1D6 PEs per trigger:** 
* [MN-01] «Protéger un allié» — Tag: ALLY_SAVED_FROM_DEATH — Trigger: An allied knight whose `currentPS ≤ 10% of maxPS` is hit by an attack, AND the attack deals 0 PS damage (fully absorbed by CdF/PA/Bouclier — no damage reaches PS), AND the knight with this motivation is at a rank within 1 of the hit ally (`abs(motivatedKnight.rank - hitAlly.rank) <= 1`). The motivated knight's presence "protected" the ally by proximity. 
* [MN-02] «Abattre plusieurs ennemis» — Tag: MULTI_KILL_3 — Trigger: knight kills 3+ enemies during a single combat turn (across all actions).
* [MN-03] «Exploit héroïque» — Tag: EXPLOIT_LANDED — Trigger: knight lands an Exploit (SPEC-04) that hits its target.
* [MN-04] «Vaincre d’un seul coup» — Tag: ONE_HIT_KILL — Trigger: a single attack reduces an enemy from ≥50% maxPS to 0.
* [MN-05] «Tenir bon» — Tag: SURVIVED_LOW_PA — Trigger: knight survives a combat node while at ≤25% maxPA at any point during that node.
* [MN-06] «Ramener un allié de l’abîme» — Tag: DESPAIR_STABILIZED — Trigger: knight stabilizes a Despaired ally (SPEC-07.3 Stabilization).
* [MN-07] «Tir de précision» — Tag: LONG_RANGE_KILL — Trigger: knight kills an enemy with a ranged attack at the weapon’s maximum range band.
* [MN-08] «Bouclier impénétrable» — Tag: CDF_FULL_ABSORB — Trigger: knight’s CdF absorbs 100% of an incoming attack (0 damage reaches PA).
* [MN-09] «Assassinat furtif» — Tag: GHOST_KILL — Trigger: Rogue knight kills an enemy while Ghost mode is active (same attack that breaks Ghost).
* [MN-10] «Réparer sous le feu» — Tag: COMBAT_REPAIR — Trigger: Priest knight restores PA to an ally via NanoC/Mechanic (SPEC-17) during a combat turn.
**Major Motivation Templates (5) — one-time, +25 PEs current+max, replaced at Camelot:**
* [MJ-01] «Aucune perte» — Tag: MISSION_NO_DEATHS — Trigger: complete a full mission (all 5 nodes) with zero knight deaths.
* [MJ-02] «Terrasser le monstre» — Tag: BOSS_KILLED — [v8.9] At character creation, one boss enemy is randomly selected from the campaign's boss pool and assigned as this knight's target (`targetBossId`). Trigger: the party kills the assigned boss during any mission where it appears. **FUTURE CONTENT for demo:** The demo has only one mission with the Ours Corrompu as the only boss. In the full game, the boss pool would include multiple patrons across missions, making the random assignment meaningful. In demo: if the assigned boss happens to be the Ours Corrompu (the only option), the motivation can trigger. Implementation: store `targetBossId` on KnightBase at generation, check on boss kill event.
* [MJ-03] «Indemnes» — Tag: MISSION_HEALTHY_FINISH — Trigger: complete a mission with all 4 deployed knights above 50% maxPA.
* [MJ-04] «Machine de guerre» — Tag: HIGH_DAMAGE_DEALER — Trigger: a single knight deals 100+ effective PS damage to enemies across one complete mission. Track cumulative damage actually subtracted from enemy PS (after all armor layers: CdF, Bouclier, PA). Raw damage and PA damage do not count. Only PS reduction is tracked.
* [MJ-05] «Revenant» — Tag: FOLD_SURVIVOR — Trigger: a knight who entered Fold state (PA=0, SPEC-10) during the mission survives to mission end.

---

## [v5] SPEC-21: Demo Enemy Roster
### Purpose
Complete stat blocks for all enemies in the demo, using the EnemyBase schema from SPEC-18. Covers 5 enemy types spanning all tiers. Includes AI targeting profiles, weapon data, and capacity definitions.

[v8.9] **Stat block values are authoritative.** Defense, Reaction, and Initiative values listed in each enemy's stat block are hand-authored from the tabletop bestiary. Do NOT derive them from aspects or Exceptionnels — no single formula reproduces all values. Agents must load these directly from the EnemyBase ScriptableObject. See SPEC-18 §18.3 for details.

### 21.1 Nocte (T1 Bande — La Bête)
Shadow-born flying swarm. First enemies encountered. Weak individually, devastating in numbers.

| Field | Value |
| --- | --- |
| Tier | T1 Bande (recrue) |
| Seigneur | La Bête |
| Aspects | Chair 4 / Bête 9 / Machine 1 / Dame 0 / Masque 4 (**data completeness only** — Bandes do not use aspects for attacks or defense rolls, see SPEC-11) |
| Aspects Exceptionnels | None |
| Défense | 5 (hand-authored, NOT derived from aspects — see SPEC-18 §18.3) |
| Réaction | 1 (hand-authored) |
| Initiative | 1 (Bandes always act last — SPEC-11) |
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
| Défense | 4 (hand-authored — see SPEC-18 §18.3) |
| Réaction | 3 (hand-authored) |
| Initiative | 2 (hand-authored) |
| PS | 20 |
| PA / Bouclier | 0 / 0 |
| Weapon (Contact) | Griffes et crocs: 5D6 + 2, Contact, no effects. Attack Aspect: Bête (8). |
| Weapon (Ranged) | Flot de ténèbres: 3D6, Moyenne, Choc 1 + Anathème. Attack Aspect: Bête (8). Bête Exceptionnel Mineur (2) adds +2 auto-successes to this roll. |
| AI: Primary/Secondary | HIGHEST_DAME / HIGHEST_AURA |
| AI: RankBehavior | HYBRID_MELEE_75 (see SPEC-18.12 Hybrid AI Pattern) |
| AI: Aggression | 2 |


Capacities:
Capacity (Saut nv1): Can leap to any rank in one movement action. [v8.9] Renamed from "Module" for demo clarity. Future human-type enemies may have actual modules; for now all enemy special abilities are capacities.
Anathème (Flot de ténèbres): [v7] [v8.9] AI prioritizes this weapon to deal Anathème PEs damage (CdF → PEs route per SPEC-06 §Knight Defense vs Anathème). The attack roll is standard — successes must exceed target's Reaction at Moyenne range. CdF still applies to reduce incoming damage, PA is completely bypassed. If Bestian is killed, PJs who suffered this effect recover 1D6 PEs per 6 PEs lost.

### 21.3 Faune (T3 Salopard — La Bête)
Corrupted humans. Elite melee fighters with brutal charge capability.

| Field | Value |
| --- | --- |
| Tier | T3 Salopard (recrue) |
| Seigneur | La Bête |
| Aspects | Chair 8 / Bête 10 / Machine 4 / Dame 4 / Masque 7 |
| Aspects Exceptionnels | Bête: Majeur (6), Masque: Mineur (1) |
| Défense | 6 (hand-authored — includes Masque Mineur +1, see SPEC-18 §18.3) |
| Réaction | 2 (hand-authored) |
| Initiative | 3 (hand-authored) |
| PS | 80 |
| PA / Bouclier | 0 / 0 |
| Weapon (Contact) | Griffes corrompues: 4D6 + 16, Contact, no effects. Attack Aspect: Bête (10). |
| AI: Primary/Secondary | NEAREST_RANK / HIGHEST_DAME |
| AI: RankBehavior | ADVANCE |
| AI: Aggression | 3 |


Capacities:
Charge Brutale: Once per combat. See SPEC-18.13. Damage = 4D6+16 + 2×10 (Bête) = 4D6+36.
Capacity (Saut nv2): Enhanced leap. Can cross 2 ranks in one movement. [v8.9] Renamed from "Module" for demo clarity.
[v8.9] **Flat bonus breakdown:** The weapon's +16 is pre-baked from Bête Exceptionnels: +10 (Bête Majeur adds full Bête Aspect score to contact damage) + +6 (Bête Mineur adds Exceptionnel score to contact damage). The weapon base is 4D6 (dice only). For Charge Brutale, add 2× Bête score (+20) ON TOP of the pre-baked weapon stat: 4D6+16+20 = 4D6+36. Do NOT strip out the +16 before adding Charge — the Charge bonus is an additional modifier, not a replacement. See SPEC-18.13 for the canonical formula.

### 21.4 Behemot (T4 Colosse — La Bête)
Massive fused beast. Requires Anti-Véhicule or sustained focus fire. Demo mid-boss.

| Field | Value |
| --- | --- |
| Tier | T4 Colosse (initié) |
| Seigneur | La Bête |
| Aspects | Chair 15 / Bête 16 / Machine 2 / Dame 2 / Masque 2 |
| Aspects Exceptionnels | Chair: Majeur (5), Bête: Majeur (6) |
| Défense | 8 (hand-authored — see SPEC-18 §18.3) |
| Réaction | 1 (hand-authored) |
| Initiative | 1 (hand-authored) |
| PS | 200 |
| PA / Bouclier | 0 / 5 |
| Colosse Rule | YES — 10 raw damage = 1 effective. Anti-Véhicule bypasses. |
| Point Faible | Endurance |
| Weapon (Contact) | Griffes et crocs: 3D6 + 22, Contact, Choc 2 + Dispersion 3. Attack Aspect: Bête (16). |
| Weapon (Ranged) | Langue épineuse: 3D6, Courte, Ignore Armure. Attack Aspect: Bête (16). |
| AI: Primary/Secondary | NEAREST_RANK / HIGHEST_MACHINE |
| AI: RankBehavior | ADVANCE |
| AI: Aggression | 3 |


Capacities:
Peur (1):
Charge Brutale: Once per combat. See SPEC-18.13. Damage = 3D6+22 + 2×16 (Bête) = 3D6+54.
Actions Multiples (1): 2 combat actions per turn total.
Immune to Forced Displacement: `immuneToForcedDisplacement = true` (SPEC-18). Too massive to push. All forced displacement effects (Choc push, charge knockback, any capacity that forces rank change) are negated. Voluntary movement is unaffected.
DD Rank placement: Behemot always occupies Rank 1 (front).

### 21.5 Ours Corrompu (T5 Patron — La Bête)
Demo final boss. Two-phase fight. **This file (SPEC-21.5) is canonical for stat block values.** For the phase transition procedure and timing, see [SPEC-14](./06_EnemySystem.md) (canonical). Corrupted bear incarnation of La Bête.

[v7] Phase 1 — Raging Beast (PS > 0)

| Field | Value |
| --- | --- |
| Tier | T5 Patron |
| Seigneur | La Bête |
| Aspects | Chair 12 / Bête 14 / Machine 4 / Dame 6 / Masque 8 |
| Aspects Exceptionnels | Chair: Mineur (3), Bête: Majeur (6) |
| Défense | 8 (hand-authored — see SPEC-18 §18.3) |
| Réaction | 4 (hand-authored) |
| Initiative | 6 (hand-authored) |
| PS | 120 |
| PA / Bouclier | 0 / 8 |
| Point Faible | Sang-Froid |
| Weapon (Contact) | Griffes: 5D6 + 18, Contact, Choc 1. Attack Aspect: Bête (14). |
| Weapon (Ranged) | Souffle ténébreux: 3D6, Moyenne, Dispersion 3 + Anathème. Attack Aspect: Bête (14). | [v8.9] Tabletop Dispersion 2 → DD Dispersion 3 (radius 1). |
| AI: Primary/Secondary | HIGHEST_BETE / HIGHEST_COMBAT |
| AI: RankBehavior | ADVANCE |
| AI: Aggression | 2 |


Phase 1 Capacities:
Peur (2):
Charge Brutale: Once per phase. See SPEC-18.13. Damage = 5D6+18 + 2×14 (Bête) = 5D6+46. [v8] Routes through ArmourLayerResolver (SPEC-06).
Actions Multiples (1): 2 combat actions per turn.



| Field | Change from Phase 1 |
| --- | --- |
| PS | Resets to 120 (total encounter PS = 240) |
| Bouclier | Increases to 12 (from 8) |
| Défense | Increases to 10 (from 8) |
| Actions Multiples | Increases to (2) — 3 combat actions per turn |
| New Weapon | Ombre dévorante: 6D6, Longue, Ignore CdF + Dégâts Continus 3 + Anathème. Attack Aspect: Bête (14). |
| New Capacity | Régénération: at the **start of the boss's turn, before any attacks**, if Nocte bande is present AND Cohésion ≥ 50: consume 50 Cohésion, heal 25 PS. Once per turn. If Cohésion < 50: does NOT fire (no partial consume). If no bande exists: does NOT fire. |
| AI: Primary/Secondary | LOWEST_PEs / LOWEST_PS — Phase 2 hunts psychologically fragile knights to trigger Despair cascades via Ombre dévorante (Anathème → PEs damage). |
| AI: RankBehavior | ADVANCE (unchanged from Phase 1) |
| AI: Aggression | 3 (up from 2 — cornered and desperate) |
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


---

## [v5] SPEC-23: Complete Injury Table
### Purpose
Full 4×6 random injury matrix from the tabletop, with digital adaptations for permanently disabling injuries. Replaces the partial table in SPEC-08. [v5] Resolves H1.

### 23.1 Injury Roll Procedure
When a knight enters Agony (PS = 0):

[v8.9] **Step 1 — Roll severity (1D6).** This determines the injury column:

| D6 Result | Severity | Column | Probability |
|-----------|----------|--------|-------------|
| 1 | Catastrophic | Column 1 | 1/6 (≈17%) |
| 2–3 | Severe | Column 2 | 2/6 (≈33%) |
| 4–5 | Moderate | Column 3 | 2/6 (≈33%) |
| 6 | Light | Column 4 | 1/6 (≈17%) |

**Step 2 — Roll specific injury (1D6).** This determines the row (1–6) within the selected column. Read the injury at [column, row].

**Notation:** Injury results are written as `[severity die, row die]`. Examples: `[1,1]` = Column 1 Row 1 = Mort. `[3,5]` = Column 2 (severity die 3 → Severe) Row 5 = Oreille Éclatée. `[6,6]` = Column 4 (severity die 6 → Light) Row 6 = Rien.

**Implementation:** Both dice use the SPEC-28 Combat RNG stream. Roll sequentially (severity first, then row). Store both raw die results in the injury record for UI display and debug.

Injuries persist until treated at Camelot Infirmary: Cybernetic Implant (20 PG, removes one injury, maxPEs −3) or Reconstruction Therapy (100 PG, removes all injuries, resets implant count, restores maxPEs). See SPEC-25.

### 23.2 Full Injury Matrix
Column 1 (D6 = 1) — Catastrophic:

| Row | Injury | Digital Effect |
| --- | --- | --- |
| 1 | Mort (Death) | Knight is permanently dead. Remove from roster. |
| 2 | Organes Internes Touchés | All 15 characteristics -1 (floor 0). Recalculate all derived values (SPEC-01). |
| 3 | Cerveau Touché | 12 characteristics -1: all Bête (3) + Machine (3) + Dame (3) + Masque (3). Chair characteristics unaffected. Lose 1 **standard** action per turn (knight chooses which: Combat OR Movement). Bonus actions from other sources (e.g., Mode Héroïque, Watchtower extra ranged action) are unaffected. Recalculate Defense, Reaction, Initiative. |
| 4 | Colonne Vertébrale Touchée | 9 characteristics -2: all Chair (3) + Bête (3) + Masque (3). Recalculate maxPS, Defense, Initiative. |
| 5 | Hémorragie Interne | [v8.9] **NOT permanent** (exception). 3-turn countdown to permadeath. See [SPEC-08](./04_ArmorAndState.md) §Hémorragie System for complete specification (countdown timing, save conditions, Ignorer l'Agonie interaction, between-node rules). |
| 6 | Poumons Perforés | Endurance -2, Déplacement -2 (floor 0). Recalculate maxPS. |


Column 2 (D6 = 2–3) — Severe:

| Row | Injury | Digital Effect |
| --- | --- | --- |
| 1 | Crâne Brisé | 9 characteristics -1: all Machine (3) + Dame (3) + Masque (3). Recalculate Reaction, Initiative. |
| 2 | Yeux Détruits | All attack rolls -3 dice (blind penalty, floor 0 dice). |
| 3 | Dos Brisé | 6 characteristics -1: all Chair (3) + Bête (3). Recalculate maxPS, Defense. |
| 4 | Main Brisée | Combat -1, Technique -1, Tir -1, Dextérité -1 (floor 0). Recalculate Defense, Reaction. |
| 5 | Oreille Éclatée | Perception -1, Déplacement -1, Instinct -1 (floor 0). Recalculate maxPS, Defense, Initiative. |
| 6 | Côtes Brisées | Endurance -1, Combat -1 (floor 0). Recalculate maxPS, Defense. |


Column 3 (D6 = 4–5) — Moderate:

| Row | Injury | Digital Effect |
| --- | --- | --- |
| 1 | Colonne Vertébrale Brisée | [v8.9] **Canonical definition.** Spine broken — character loses use of legs, armor handles movement. Movement costs 2 actions (any combination of movement or combat actions) per rank shift. Cannot Sprint. Implement as doubled action cost, not "whole turn" — future capacities may grant additional actions. Forced displacement still applies normally (armor absorbs the push). |
| 2 | Mâchoire/Langue/Dents Détruites | Aura -3, Parole -3, Sang-Froid -3 (floor 0). |
| 3 | Pied Brisé | Combat -1, Déplacement -1, Discrétion -1, Force -1 (floor 0). Recalculate maxPS, Defense, Initiative. |
| 4 | Dos Touché | Force -1, Déplacement -1 (floor 0). Recalculate maxPS. |
| 5 | Tempe Touchée | Hargne -1, Sang-Froid -1 (floor 0). Recalculate Defense. |
| 6 | Blessure Mineure | Random non-zero characteristic -1 (floor 0). Recalculate affected derived value. |


Column 4 (D6 = 6) — Light:

| Row | Injury | Digital Effect |
| --- | --- | --- |
| 1 | Bras Mutilé | Cannot use Lourd or Deux Mains weapons. +1 difficulty to relevant tests. |
| 2 | Jambe Mutilée | Movement costs doubled. Cannot use movement-dependent abilities. |
| 3 | Traumatisme Crânien | Savoir -1, Technique -1, Instinct -1 (floor 0). Recalculate Reaction, Defense. |
| 4 | Borgne | Tir -1, Perception -1 (floor 0). Recalculate Reaction, Initiative. |
| 5 | Blessure Mineure | Random non-zero characteristic -1 (floor 0). Recalculate affected derived value. |
| 6 | Rien! | No injury. Knight is in Agony but otherwise unscathed. |


### 23.3 Digital Adaptations
The tabletop leaves some injuries to 'GM discretion.' Digital implementation requires deterministic rules:
Colonne Vertébrale Brisée: See Column 3, Row 1 above (canonical definition). Implement as doubled action cost.
Yeux Détruits:
Bras/Jambe Mutilé: Tabletop = 'MJ increases difficulty.' Digital = equipment restrictions + movement penalty.
Random characteristic (Blessure Mineure): System picks random non-zero characteristic and reduces by 1.

### 23.4 Trompe la Mort (Tarot Advantage)
Knights with the Trompe la Mort advantage (Arcane sans Nom, Tarot XIII): when rolling Mort on the injury table, reroll and take next result. Once per mission.

### Acceptance Criteria
Roll [1,1]: Severity 1 → Catastrophic (Column 1), Row 1 → Mort. Knight permanently dead unless Trompe la Mort active.
Roll [1,5]: Severity 1 → Catastrophic (Column 1), Row 5 → Hémorragie Interne. 3-turn countdown. Nod Soin before 0: survives.
Roll [3,4]: Severity 3 → Severe (Column 2), Row 4 → Main Brisée. Combat/Technique/Tir/Dextérité all −1.
Roll [6,6]: Severity 6 → Light (Column 4), Row 6 → Rien. Knight in Agony but no injury penalty.
Cerveau Touché: 12 characteristics reduced by 1: all Bête (3) + Machine (3) + Dame (3) + Masque (3). Chair characteristics (Déplacement, Force, Endurance) unaffected. Lose 1 standard action per turn (knight chooses which: Combat OR Movement). Bonus actions (Mode Héroïque, Watchtower) unaffected.

