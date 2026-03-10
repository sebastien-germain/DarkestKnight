# Feature Specification: Knight: Into the Darkness — Tactical RPG Demo

**Feature Branch**: `001-tactical-rpg-demo`
**Created**: 2026-03-10
**Status**: Draft
**Input**: Build a 2D side-scrolling tactical RPG demo adapted from the tabletop RPG "Knight" by Antre Monde Editions.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Turn-Based Combat Encounter (Priority: P1)

The player controls a squad of up to 4 knights in a turn-based combat
encounter on a 4-position linear track (similar to Darkest Dungeon).
Each turn, the player selects a knight, chooses a combat style
(Aggressive, Defensive, Precise, etc.), picks a target, and resolves
the attack with dice rolls. Enemies respond with their own attacks
according to faction-specific AI behaviors. Combat continues until all
enemies are defeated or all knights are incapacitated.

The player manages multiple interconnected resources during combat:
armor points (PA) absorb damage, health (PS) represents physical
endurance, and energy (PE/Espoir) represents psychological resilience.
Each knight carries consumable Nods (healing, armor repair, energy
restoration) that cost a movement action to use.

**Why this priority**: Combat is the core gameplay loop. Without
functional combat, no other system has context or purpose. A single
working combat encounter demonstrates the entire damage pipeline,
dice resolution, positioning system, and resource management.

**Independent Test**: Can be fully tested by placing 4 knights against
a group of enemies on the linear track and resolving combat to
completion. Delivers the core tactical experience.

**Acceptance Scenarios**:

1. **Given** a combat encounter with 4 knights and enemies on a
   4-position track, **When** the player selects a knight and chooses
   an attack style, **Then** the system resolves dice rolls, applies
   damage through the armor pipeline (CdF -> PA -> PS), and updates
   all combatant states.

2. **Given** a knight in Aggressive style, **When** they attack,
   **Then** they receive +3 bonus dice but suffer -2 to Defense and
   Reaction values.

3. **Given** an enemy with armor, **When** damage is dealt, **Then**
   damage is reduced by CdF (passive armor), remaining damage depletes
   PA, and for every 5 PA absorbed the knight takes 1 PS passthrough
   damage.

4. **Given** a knight whose PA reaches 0, **When** the armor is
   depleted, **Then** the armor folds into Guardian mode (weaker
   protection, abilities disabled) and the player is notified.

5. **Given** a knight whose PS reaches 0, **When** they are
   incapacitated, **Then** Agony triggers and the player rolls on
   the injury table to determine consequences (ranging from minor
   wounds to permadeath).

6. **Given** an ally in Agony with Hemorragie (internal bleeding),
   **When** another knight uses a Nod de Soin on them within 3 turns,
   **Then** the bleeding knight is stabilized and avoids permadeath.

7. **Given** a knight whose Espoir (PE) reaches 0, **When** despair
   triggers, **Then** the knight becomes hostile, attacks allies for
   1D6 turns, and can only be recovered via a stabilization roll by
   an ally.

8. **Given** enemies of the Bande type (swarm), **When** the player
   attacks them, **Then** only Violence damage reduces their Cohesion
   (shared health pool), and they deal automatic escalating damage
   (Debordement) each turn to all knights.

---

### User Story 2 - Knight Roster & Squad Selection (Priority: P2)

The player starts with a roster of 8 procedurally generated knights
(2 per armor class: Warrior, Paladin, Priest, Rogue). Each knight has
a unique identity: a name, an archetype personality (from 17 possible),
a heraldic emblem (Blason), Tarot cards that grant stat bonuses plus
2 advantages and 1 disadvantage, and a Haut Fait (notable deed).

Before each mission, the player reviews their roster at Camelot and
selects 4 knights to deploy. The selection screen displays each
knight's current state: health, armor condition, Espoir level,
active injuries, equipment loadout, and special abilities.

**Why this priority**: Squad selection gives the player strategic
agency and emotional investment. Without unique knights to choose
from, combat lacks identity and consequence.

**Independent Test**: Can be tested by generating 8 knights, displaying
their stats and identities, and allowing the player to select 4 for
deployment. Delivers roster management and squad composition decisions.

**Acceptance Scenarios**:

1. **Given** a new game, **When** the roster is generated, **Then**
   8 unique knights are created (2 Warrior, 2 Paladin, 2 Priest,
   2 Rogue), each with distinct archetype, Blason, Tarot cards,
   and stat spread.

2. **Given** the roster screen, **When** the player views a knight,
   **Then** they see the knight's armor class, all 5 Aspects and 15
   Characteristics, active Tarot advantages/disadvantages, current
   HP/PA/PE, any injuries, and equipped weapons.

3. **Given** the roster screen, **When** the player selects 4 knights,
   **Then** the selected squad is locked in for the mission and the
   player proceeds to mission deployment.

4. **Given** a knight with an active permanent injury, **When** the
   player views them, **Then** the injury and its mechanical penalty
   are clearly displayed.

5. **Given** a knight with low Espoir (PE), **When** the player views
   them, **Then** a warning indicates the risk of Despair if deployed.

---

### User Story 3 - Armor Class Abilities (Priority: P3)

Each armor class provides a unique tactical ability that defines the
knight's role in combat:

- **Warrior** activates one of 3 Types (Soldier, Hunter, Scholar) per
  turn, boosting an entire Aspect's characteristics by +1 for 1 PE/turn.
- **Paladin** deploys a Shrine (force field granting +6 armor to nearby
  allies) or a Watchtower (stationary gun emplacement granting an extra
  ranged attack per turn at halved Reaction).
- **Priest** uses Mechanic (repair ally armor at range for 4 PE) or
  NanoC (create temporary tactical cover structures).
- **Rogue** activates Ghost mode (stealth), becoming invisible to
  enemies until they attack. Attacking from stealth adds Discretion
  bonus to both attack and damage rolls.

**Why this priority**: Class abilities create tactical diversity and
give each knight a distinct role. Without them, all knights feel
interchangeable regardless of armor class.

**Independent Test**: Can be tested in a combat encounter by activating
each class ability and verifying its mechanical effect (Warrior stat
boost, Paladin shield/turret, Priest repair/cover, Rogue stealth
bonus).

**Acceptance Scenarios**:

1. **Given** a Warrior in combat, **When** the player activates Type
   Soldier, **Then** all 3 characteristics in the associated Aspect
   gain +1 for 1 turn, costing 1 PE per turn to maintain.

2. **Given** a Paladin in combat, **When** the player deploys a Shrine,
   **Then** all allies within 1 position of the Shrine gain +6 CdF
   armor protection. The Shrine costs 2 PE to deploy and 1 PE/turn
   to maintain.

3. **Given** a Priest in combat, **When** the player uses Mechanic on
   an ally, **Then** the targeted ally's PA is restored by 3D6+6
   (contact range) or 2D6+6 (long range), costing 4 PE.

4. **Given** a Rogue in combat, **When** the player activates Ghost
   mode, **Then** the Rogue becomes invisible to enemies (based on
   Discretion test vs enemy Machine stat). First attack from stealth
   adds Discretion to the dice pool and damage.

---

### User Story 4 - Mission Flow & Node Progression (Priority: P4)

The player deploys their squad on a mission consisting of 7 nodes:
5 combat encounters and 2 narrative event nodes. The mission follows
a fixed structure: node 1 (combat), node 2 branches (nest or ruins
with event), node 3 (combat convergence), node 4 (event + combat),
node 5 (boss encounter with Bande swarm).

Between nodes, the player can use Nods (consumables) for free to
heal, repair armor, or restore energy. All resources carry over
between nodes — there is no rest or recovery between encounters.
This creates mounting attrition pressure across the mission.

The boss encounter (node 5) features a two-phase fight: Phase 1
against the Ours Corrompu (Corrupted Bear) with normal combat,
Phase 2 triggers when the boss's health reaches 0, introducing new
abilities (Anatheme ranged attack hitting PE directly), reinforcement
Bande, and aggressive AI.

**Why this priority**: Mission flow ties individual combat encounters
into a cohesive experience with strategic resource management. Without
it, combat encounters are disconnected.

**Independent Test**: Can be tested by running a full mission from
node 1 through node 5, verifying resource carryover between nodes,
event triggers at narrative nodes, and the boss phase transition.

**Acceptance Scenarios**:

1. **Given** a deployed squad, **When** the mission begins, **Then**
   the first combat node loads with enemies appropriate to the
   mission difficulty.

2. **Given** completion of node 1, **When** the player reaches
   node 2, **Then** they choose between two branching paths (nest
   or ruins), each with different enemy compositions and event
   opportunities.

3. **Given** a combat node is completed, **When** transitioning to
   the next node, **Then** all knight states (HP, PA, PE, injuries,
   remaining Nods) carry over exactly as they were.

4. **Given** the player reaches node 5 (boss), **When** the boss's
   PS reaches 0, **Then** Phase 2 triggers: the boss gains new
   abilities, a reinforcement Bande spawns, and the AI becomes
   more aggressive.

5. **Given** a narrative event node, **When** the player interacts,
   **Then** a contextual event triggers offering potential PE
   restoration or tactical information.

---

### User Story 5 - Camelot Hub & Post-Mission Management (Priority: P5)

After completing (or failing) a mission, the player returns to Camelot.
All surviving knights have their HP and PA fully restored, but PE
(Espoir) does NOT reset — psychological damage carries between
missions.

At Camelot, the player spends Points de Gloire (PG) earned during
the mission. PG is spent on:
- **Reconstruction Therapy**: Remove permanent injuries via cybernetic
  implants (cost varies by injury severity)
- **Equipment**: Purchase new weapons or grenades
- **Recruitment**: Replace dead knights with new procedurally generated
  recruits

The player can also review their full roster, reassign equipment,
and prepare for the next mission deployment.

**Why this priority**: The hub closes the gameplay loop, giving
meaning to mission rewards and permadeath consequences. Without it,
PG has no purpose and dead knights cannot be replaced.

**Independent Test**: Can be tested by returning from a mission with
injured/dead knights and PG, then spending PG on therapy and
recruitment, verifying roster updates.

**Acceptance Scenarios**:

1. **Given** a completed mission, **When** the player returns to
   Camelot, **Then** all surviving knights have HP and PA fully
   restored, but PE remains at its mission-end value.

2. **Given** PG earned from combat, **When** the player visits the
   therapy facility, **Then** they can spend PG to remove specific
   permanent injuries from knights.

3. **Given** a dead knight, **When** the player visits recruitment,
   **Then** they can spend PG to generate a new knight of the same
   or different armor class to fill the roster slot.

4. **Given** the Camelot hub, **When** the player views their roster,
   **Then** they see each knight's full state including carried-over
   PE, any remaining injuries, and equipment.

5. **Given** all 8 roster slots are filled, **When** no knights have
   died, **Then** recruitment is unavailable and the player proceeds
   directly to mission selection.

---

### User Story 6 - Weapon Loadout & Grenade Management (Priority: P6)

Each knight carries weapons with distinct profiles: damage dice,
violence dice (for Bande damage), range band (Contact/Courte/Moyenne/
Longue), and special effects (Precise, Armor-piercing, Explosive,
Suppressive, Stunning, etc.).

The player equips weapons before deployment and can switch weapons
during combat (costs 1 movement action). Dual-wielding is available
in two styles: Ambidextre (2 separate attacks at -3 dice each) or
Akimbo (1 combined attack at -3 dice).

Each knight also carries 5 grenades per mission (auto-refilled at
Camelot). Grenades come in 5 types: Shrapnel, Flashbang, Anti-Armor,
IEM, and Explosive, each with different tactical effects.

**Why this priority**: Weapons and grenades add tactical variety to
combat but are less critical than the core combat loop and class
abilities. The system enriches existing combat rather than enabling it.

**Independent Test**: Can be tested by equipping different weapon
profiles and grenades, then using them in combat to verify damage
calculations, range restrictions, special effects, and dual-wield
mechanics.

**Acceptance Scenarios**:

1. **Given** a knight with an equipped weapon, **When** the player
   attacks, **Then** damage is calculated using the weapon's damage
   dice, modified by combat style and any weapon effects.

2. **Given** a weapon with the Explosive effect, **When** the knight
   attacks, **Then** splash damage hits combatants adjacent to the
   target (including allies if in range).

3. **Given** a knight in combat, **When** the player uses a grenade,
   **Then** the grenade's specific effect applies (Shrapnel: area
   damage, Flashbang: stun, Anti-Armor: ignore CdF, etc.).

4. **Given** a knight dual-wielding in Ambidextre mode, **When** they
   attack, **Then** two separate attack rolls are made, each with
   -3 dice penalty.

---

### User Story 7 - Enemy Faction Behaviors (Priority: P7)

Four enemy factions provide distinct tactical challenges:

- **La Chair** (flesh creatures): Swarm-based Bandes with high
  Cohesion, escalating automatic damage, weak to Violence.
- **La Bete** (corrupted beasts): Individual combatants with high
  physical stats, Charge Brutale (rush attacks), and the boss
  Ours Corrompu with two combat phases.
- **La Dame** (seductive horrors): Anatheme attacks that bypass armor
  and damage PE (morale) directly. Peur (fear) aura that debuffs
  nearby knights.
- **L'Ophidien** (serpent entities): Tactical enemies with
  poison/continuous damage effects and high Reaction.

Each faction's AI follows distinct behavior patterns: Bandes swarm
and overwhelm, beasts charge aggressively, Dame units target morale,
and Ophidien units apply debuffs and kite.

**Why this priority**: Faction variety adds replayability and tactical
depth but requires the core combat system to be functional first.
The demo mission uses La Bete as the primary faction with La Chair
Bandes as support.

**Independent Test**: Can be tested by running combat encounters
against each faction and verifying their AI behavior patterns,
special attacks, and unique mechanics.

**Acceptance Scenarios**:

1. **Given** a Bande (swarm) enemy, **When** the player attacks them,
   **Then** only Violence damage reduces their Cohesion, and they
   deal escalating Debordement damage each turn (Turn N = N x base).

2. **Given** an enemy with Charge Brutale, **When** their AI activates,
   **Then** they rush toward the nearest knight, dealing bonus damage
   based on distance traveled.

3. **Given** an enemy with Anatheme, **When** they attack, **Then**
   damage bypasses armor entirely and reduces the target's PE directly.

4. **Given** an enemy with Peur (fear aura), **When** a knight is
   within range, **Then** the knight suffers a dice penalty on all
   rolls until they move out of range or the enemy is defeated.

---

### Edge Cases

- What happens when all 4 knights are incapacitated in combat?
  The mission fails, and the player returns to Camelot. Knights in
  Agony roll their injury results; knights with Hemorragie who were
  not stabilized are permanently dead.

- What happens when a knight in Despair (hostile) kills another
  knight? The killed knight follows normal Agony/injury rules.
  Permadeath applies.

- What happens when all 8 roster knights are dead? The campaign
  ends (game over). The player must start a new campaign with a
  fresh roster.

- What happens when a Bande's Cohesion reaches 0? The swarm is
  destroyed. Any remaining individual enemies continue fighting.

- What happens when the player has 0 PG at Camelot? They cannot
  purchase therapy, equipment, or recruits. They must deploy with
  current roster state.

- What happens when a knight uses their last Nod during combat?
  The Nod is consumed normally. The knight has no more consumables
  of that type for the remainder of the mission.

- What happens when the boss transitions to Phase 2 mid-turn?
  The phase transition occurs immediately when PS reaches 0.
  The boss gains new stats and abilities; remaining enemies
  continue their turns normally.

## Requirements *(mandatory)*

### Functional Requirements

**Knight Generation & Roster**
- **FR-001**: System MUST generate 8 unique knights at campaign start (2 per armor class: Warrior, Paladin, Priest, Rogue)
- **FR-002**: Each knight MUST have a procedurally generated identity: archetype (from 17 options), Blason, Haut Fait, and name
- **FR-003**: Each knight MUST draw 5 Tarot cards that determine stat bonuses, 2 advantages, and 1 disadvantage
- **FR-004**: System MUST display complete knight information: 5 Aspects, 15 Characteristics, armor stats, Espoir, injuries, equipment, and Tarot effects

**Combat Core**
- **FR-005**: Combat MUST take place on a 4-position linear track with knights and enemies occupying positions
- **FR-006**: Turn order MUST be determined by initiative values, displayed in a visible queue
- **FR-007**: Players MUST choose a combat style each turn: Agressif (+3 dice, -2 Def/React), Defensif (-3 dice, +2 Def), Precis (+2nd characteristic), or other available styles
- **FR-008**: Attack resolution MUST use dice pools: roll Base + Combo + modifiers, count even results as successes, compare against Defense (melee) or Reaction (ranged)
- **FR-009**: Hit threshold MUST require successes to STRICTLY EXCEED defense/reaction (equal = miss)
- **FR-010**: All mathematical operations MUST floor (round down) for division

**Damage Pipeline**
- **FR-011**: Damage MUST flow through the armor pipeline: raw damage -> CdF reduction -> PA absorption -> PS passthrough (1 PS per 5 PA absorbed)
- **FR-012**: Violence damage MUST apply separately to Bande Cohesion
- **FR-013**: When PA reaches 0, armor MUST fold into Guardian mode with reduced protection and disabled abilities

**Status & Consequences**
- **FR-014**: When PS reaches 0, Agony MUST trigger with an injury table roll determining consequences
- **FR-015**: Hemorragie injury MUST impose a 3-turn countdown to permadeath, recoverable only by ally Nod de Soin
- **FR-016**: When PE reaches 0, Despair MUST trigger: knight becomes hostile for 1D6 turns, attackable only via stabilization roll
- **FR-017**: PE penalties MUST apply to all rolls when PE drops below threshold (scaling penalties as PE decreases)
- **FR-018**: Heroism MUST be spendable for: rerolling failed dice, maximizing weapon damage, ignoring injury/despair penalties

**Armor Class Abilities**
- **FR-019**: Warriors MUST be able to activate Types (Soldier/Hunter/Scholar) for +1 to an Aspect's characteristics at 1 PE/turn
- **FR-020**: Paladins MUST be able to deploy Shrine (+6 CdF to nearby allies, 2 PE deploy, 1 PE/turn) or Watchtower (extra ranged attack, halved Reaction)
- **FR-021**: Priests MUST be able to use Mechanic (repair ally PA at range, 4 PE) or NanoC (create cover structures)
- **FR-022**: Rogues MUST be able to activate Ghost mode (stealth with Discretion-based detection, bonus attack/damage from stealth)

**Enemies**
- **FR-023**: Bande (swarm) enemies MUST use shared Cohesion health, be damageable only by Violence, and deal escalating Debordement automatic damage each turn
- **FR-024**: Individual enemies MUST have distinct AI behaviors based on faction (aggression patterns, target selection, special abilities)
- **FR-025**: Boss encounters MUST support multi-phase transitions with stat changes and new ability unlocks

**Mission Structure**
- **FR-026**: Missions MUST consist of 7 nodes (5 combat, 2 narrative events) with branching at node 2
- **FR-027**: All knight state (HP, PA, PE, injuries, Nods) MUST carry over between nodes with no recovery except Nod usage
- **FR-028**: Each knight MUST carry 3 Nods of each type (Soin, Armure, Energie) per mission, usable as movement action in combat or free action between nodes

**Camelot Hub**
- **FR-029**: Surviving knights MUST have HP and PA fully restored upon return to Camelot; PE MUST NOT reset
- **FR-030**: Players MUST be able to spend PG on: Reconstruction Therapy (remove injuries), equipment, and recruitment (replace dead knights)
- **FR-031**: Dead knights MUST be permanently removed from the roster; replacement knights are procedurally generated

**Weapons & Equipment**
- **FR-032**: Each weapon MUST have a profile: damage dice, violence dice, range band, and special effects
- **FR-033**: Weapon switching in combat MUST cost 1 movement action
- **FR-034**: Grenades (5 types, 5 per mission) MUST apply faction-appropriate tactical effects and auto-refill at Camelot

### Key Entities

- **Knight**: A player-controlled combatant with armor class, 5 Aspects (each with 3 Characteristics), Tarot-derived advantages/disadvantages, archetype personality, Blason, Haut Fait, equipment loadout, and current state (PA, PS, PE, injuries)
- **Armor Class**: One of 4 types (Warrior, Paladin, Priest, Rogue) defining base stats, unique ability, and tactical role
- **Weapon Profile**: A weapon's combat statistics including damage, violence, range, and special effects
- **Enemy**: An AI-controlled combatant with faction, tier (Bande/Hostile/Initie/Colosse/Patron), stats, AI behavior, and special abilities
- **Bande**: A swarm enemy type with shared Cohesion health pool, automatic Debordement damage, and Violence-only vulnerability
- **Mission**: A sequence of 7 nodes (combat and narrative) with branching paths and escalating difficulty culminating in a boss fight
- **Nod**: A consumable item (3 types: Soin/Armure/Energie) that restores knight resources; limited supply per mission
- **Points de Gloire (PG)**: Meta-currency earned through combat, spent at Camelot for therapy, equipment, and recruitment
- **Tarot Card**: One of 22 Major Arcana cards granting stat bonuses, an advantage, and a disadvantage to the knight who draws it
- **Injury**: A permanent wound resulting from an Agony roll, imposing mechanical penalties until treated at Camelot

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A complete combat encounter (4 knights vs mixed enemy group) resolves from start to finish with correct damage pipeline, status effects, and win/loss conditions within 5-10 minutes of play time
- **SC-002**: A full demo mission (7 nodes) completes in 10-15 minutes with resource attrition creating meaningful tension by nodes 3-4
- **SC-003**: All 4 armor class abilities produce mechanically distinct combat outcomes when used (Warrior stat boost, Paladin shield/turret, Priest heal/cover, Rogue stealth bonus)
- **SC-004**: Permadeath consequences are permanent — a dead knight never reappears in the roster, and the player must recruit a replacement
- **SC-005**: 5 integration test scenarios from the TechSpec pass end-to-end, validating cross-system interactions (combat resolution, armor pipeline, Espoir/Despair, injuries, Bande mechanics)
- **SC-006**: All dice rolls and damage calculations produce deterministic results when given the same RNG seed
- **SC-007**: Players can complete the full game loop (Camelot -> mission -> Camelot) at least twice before running out of meaningful decisions
- **SC-008**: The boss encounter (Ours Corrompu) successfully transitions between Phase 1 and Phase 2, introducing new abilities and reinforcements
