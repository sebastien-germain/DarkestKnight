# Mission Structure, Save System & Camelot Hub
> **SPECs:** 13, 22, 25
> **Domain:** `Scripts/Meta`
> **Dependencies:** [08_ContentData.md](./08_ContentData.md) (SPEC-21), [06_EnemySystem.md](./06_EnemySystem.md) (SPEC-14), [01_KnightDataModel.md](./01_KnightDataModel.md) (SPEC-02)
> **Depended on by:** None (top of dependency chain)
---
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
Nod use allowed: Knights may use Nods during node transitions at no action cost (free outside combat). This is the ONLY recovery mechanism between encounters. See [SPEC-33](./13_NodSystem.md) §33.4.
Injuries persist: All active injuries carry forward. No healing between nodes.
Dead knights stay dead. Permadeath is permanent. In DD rank system, empty rank remains empty.
Design intent: Maximum attrition pressure. Resource management (Nods, PE, PEs) is the core strategic layer. Matches roguelite philosophy of short, high-stakes runs.


---

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
[v8.9] **Combat end condition:** A combat node ends when ALL enemies are eliminated — this includes both on-track individual enemies (PS = 0) AND off-track bandes (Cohésion = 0). A bande that still has Cohésion > 0 keeps the combat going even if all on-track enemies are dead. No enemies flee in demo.
Loot/recovery: No automatic recovery. Knights may use Nods (no action cost between combats).
State persists: PS, PA, PE, PEs, injuries, Heroism, ammo — all carry over exactly.
Dead knights: Remain dead. Empty DD rank slot. Cannot be replaced mid-mission.
Branch selection (Node 2): Player chooses path. Choice is permanent. Both converge at Node 3. Branch selection screen: show node title + short narrative description + difficulty rating (stars). Do NOT show exact enemy composition. Example display: ‘2A: Le Nid — Corrupted creatures nest here. ★★☆☆☆’ vs ‘2B: Les Ruines — A lone hunter stalks the ruins. ★★★☆☆ (Event)’. The ‘Event’ tag hints at a narrative opportunity without revealing specifics.

### 22.3 Narrative Events

**Authoring Schema:** Each narrative event uses the following structure. Agents adding future events must follow this schema.
```csharp
struct NarrativeEvent {
  string eventId;           // Stable identifier (e.g., "node4_pointfaible_discovery")
  int nodeIndex;            // Which mission node this event belongs to
  string triggerType;       // AUTO (on node entry), PLAYER_CHOICE, PHASE_TRANSITION, ROLL_CHECK
  string description;       // Flavor text displayed to the player
  float rollThreshold;      // For ROLL_CHECK: result required on 1D6 (e.g., 6 for 1/6 chance). -1 if N/A.
  NarrativeEffect[] effects; // List of effects applied on trigger
}

struct NarrativeEffect {
  string targetScope;       // ALL_KNIGHTS, SINGLE_KNIGHT (+ selector), SPECIFIC_KNIGHT_ID
  string selectorRule;      // For SINGLE_KNIGHT: "HIGHEST_PERCEPTION", "RANDOM", etc. Null if ALL.
  string effectType;        // PES_GAIN, PES_LOSS, REVEAL_POINT_FAIBLE, GRANT_HEROISME
  string diceFormula;       // e.g., "1D6", "+3", "−1D6". Null for non-numeric effects.
  string metadata;          // Extra data (e.g., Point Faible characteristic name). Null if N/A.
}
```

**Demo Narrative Events:**

```csharp
// Event 1: Node 2B — Survivor discovered
NarrativeEvent {
  eventId = "node2b_survivor_discovery",
  nodeIndex = 2,  // Node 2B
  triggerType = "AUTO",
  description = "You discover a survivor hiding among the ruins — a spark of hope in the darkness.",
  rollThreshold = -1,  // N/A (AUTO trigger)
  effects = [
    NarrativeEffect {
      targetScope = "ALL_KNIGHTS",
      selectorRule = null,
      effectType = "PES_GAIN",
      diceFormula = "1D6",
      metadata = null
    }
  ]
}

// Event 2: Node 4 — Point Faible discovery
NarrativeEvent {
  eventId = "node4_pointfaible_discovery",
  nodeIndex = 4,
  triggerType = "ROLL_CHECK",
  description = "Your knights examine the den entrance for signs of the beast's weakness...",
  rollThreshold = 6,  // 1D6, triggers on 6 only (1/6 chance)
  effects = [
    NarrativeEffect {
      targetScope = "ALL_KNIGHTS",  // Point Faible revealed to entire squad
      selectorRule = null,
      effectType = "REVEAL_POINT_FAIBLE",
      diceFormula = null,
      metadata = "Sang-Froid"  // The Ours Corrompu's weak characteristic
    },
    NarrativeEffect {
      targetScope = "SINGLE_KNIGHT",
      selectorRule = "HIGHEST_PERCEPTION",
      effectType = "PES_GAIN",
      diceFormula = "+3",
      metadata = null
    }
  ]
}

// Event 3: Node 4 — Pre-battle prayer
NarrativeEvent {
  eventId = "node4_prebattle_prayer",
  nodeIndex = 4,
  triggerType = "PLAYER_CHOICE",
  description = "A moment of stillness before the final battle. Spend 1 Heroism to rally the squad?",
  rollThreshold = -1,  // N/A (player chooses yes/no)
  effects = [
    // Cost: 1 Heroism from any knight (player selects which)
    // On accept:
    NarrativeEffect {
      targetScope = "ALL_KNIGHTS",
      selectorRule = null,
      effectType = "PES_GAIN",
      diceFormula = "1D6",
      metadata = null
    }
    // On decline: no effect. Choice is optional.
  ]
}

// Event 4: Node 5, Phase 2 — Darkness erupts
NarrativeEvent {
  eventId = "node5_phase2_darkness_erupts",
  nodeIndex = 5,
  triggerType = "PHASE_TRANSITION",
  description = "Darkness erupts from the corrupted bear — a wave of dread engulfs the den.",
  rollThreshold = -1,  // N/A (triggered by boss phase transition)
  effects = [
    NarrativeEffect {
      targetScope = "ALL_KNIGHTS",
      selectorRule = null,
      effectType = "PES_LOSS",
      diceFormula = "1D6",  // Each knight rolls independently
      metadata = null
    }
  ]
}
```

**Summary table (quick reference):**

| Node | Event | Trigger | PEs Effect |
| --- | --- | --- | --- |
| 2B | Survivor discovered | Auto on entry | All knights: +1D6 PEs |
| 4 | Point Faible discovery | Roll 1D6 on entry; triggers on 6 only | Sang-Froid revealed. +3 PEs to knight with highest Perception. |
| 4 | Pre-battle prayer | Player choice (spend 1 Heroism) | All knights +1D6 PEs. Or skip. |
| 5 (phase 2) | Darkness erupts | Phase transition | All knights: −1D6 PEs |


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
Boss fight: 2 phases. Phase 2 PS reset. Nocte bande: fresh spawn (200 Cohésion, Débordement 4) if destroyed; AddCohesion(200) if still alive.


---

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

| Action | Description | Cost | Location |
| --- | --- | --- | --- |
| View Roster | See all 8 knights: PS/PA/PE/PEs, injuries, equipment, armor class, abilities. | Free | Hub main |
| Auto-Refill Ammo | Grenades (5), Nods (3 each type), Chargeur weapons: all reset to max. | Free (auto) | Hub main |
| Cybernetic Implant | Remove **one** injury from a knight. Each implant permanently reduces maxPEs by 3 (tabletop canon: replacing flesh with machine costs humanity). Max 6 implants per knight. Minimum maxPEs floor: 10 — if an implant would reduce maxPEs below 10, block the implant. Quick and cheap, but accumulates a lasting morale cost. [v8.9] **Prisonnier interaction:** The Prisonnier disadvantage (SPEC-24.3) blocks all PEs recovery but does NOT prevent implant installation — the floor rule applies to maxPEs, not PEs recovery. However, UI should display an additional warning for Prisonnier knights: "This knight cannot recover PEs — maxPEs reduction is effectively permanent until Prisonnier is resolved." | 20 PG | Infirmary |
| Reconstruction Therapy | [v8.9] Full-body restoration. Removes **ALL** injuries from a knight AND reverses all implant side effects: resets implant count to 0 and **restores maxPEs** to its pre-implant value (base 50 + Tarot modifiers, e.g., Forteresse Spirituelle +5). Lore: the therapy rebuilds the knight's organic body, replacing cybernetic parts with cloned tissue — their humanity is restored. Expensive but a complete reset. | 100 PG | Infirmary |
| Select Mission | Choose next mission from available list (demo: 1 mission). | Free | Hub main |
| Select Squad | Choose 4 knights to deploy. Final once mission starts. | Free | Hub main |
| Memorial | View fallen knights. Flavor/narrative only. | Free | Hub main |


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
| Cybernetic Implant | 20 PG | Infirmary. Removes 1 injury. maxPEs −3 per implant. |
| Reconstruction Therapy | 100 PG | Infirmary. Removes ALL injuries. Resets implant count to 0. Restores maxPEs to pre-implant value. |
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

> **⚠️ PE vs PEs — Critical distinction:**
> - **PE** (Points d'Énergie, energy for abilities) is **fully restored** to max at Camelot. Free.
> - **PEs** (Points d'Espoir, morale/hope) is **NOT restored** at Camelot in demo. PEs carries over between missions unchanged. Future: psychologist room.

Knight HP/PA/PE: Fully restored to max at Camelot (free healing between missions).
PEs: [v8] No recovery at Camelot in demo. PEs carry over between missions unchanged. Future content will add Camelot rooms for PEs support (e.g. psychologist). Motivation bonuses earned during mission carry over as permanent PEs gains.
Injuries: Persist until treated at Infirmary (implant removes one, therapy removes all).
Implants: Persist permanently. Each reduces maxPEs by 3. Only removed by Reconstruction Therapy (which also restores maxPEs).
Heroism: Resets to 0 at mission start.
Equipment: Persists. New purchases added to inventory.
Dead knights: Permanently removed. No resurrection.
PG: Running total. Carries over indefinitely.

### Acceptance Criteria
Mission success with 4 alive, 1 Exploit, boss killed, minor motivation: 30 + 10 + 15 + 1 + 3 = 59 PG.

All 8 dead: campaign over screen. No further play possible.
Return to Camelot: PS/PA/PE fully restored. PEs unchanged (carry over between missions as-is). Future content: psychologist room for PEs recovery. Injuries persist.
Ammo auto-refill: grenades 5, nods 3 each, chargeur weapons at max.
[v8.9] Implant: Knight has maxPEs 50, 2 injuries. Player buys 1 Cybernetic Implant (20 PG): 1 injury removed, maxPEs = 50 − 3 = 47, implantCount = 1. If currentPEs > 47, clamp to 47.
[v8.9] Implant blocked: Knight has maxPEs 10, 1 injury. Implant would reduce maxPEs to 7 < 10 floor → action blocked. UI: "maxPEs too low for implant."
[v8.9] Therapy: Same knight (maxPEs 47, implantCount 1, 1 remaining injury). Player buys Reconstruction Therapy (100 PG): all injuries removed, implantCount = 0, maxPEs restored to 50 (base + Tarot mods). If currentPEs was 30, it stays 30 (therapy restores the ceiling, not current PEs).



---

> **SPEC-26 (UI Screen Flow)** and **SPEC-27 (Audio/VFX Event Hooks)** are defined in [11_UIAndPresentation.md](./11_UIAndPresentation.md). Do not duplicate here — refer to the canonical source.
