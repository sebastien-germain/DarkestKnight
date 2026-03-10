# Command Interface Contract

**Source**: SPEC-32 (UI is read-only view + command sender, never mutates combat state)
**Purpose**: Defines all commands that the UI layer sends to the CombatManager for validation and execution.
**Pattern**: `CombatManager.Submit(ICommand cmd)` returns `CommandResult { bool success; string error; }`

UI MUST NOT mutate any combat state directly. All player actions flow through typed command structs.
CombatManager validates each command against the current game state and either executes it or returns
a rejection with a reason string for UI to display.

---

## Player Combat Commands (UI -> CombatManager)

### RequestAttack

```csharp
struct RequestAttack {
    CombatantId attacker;
    CombatantId target;
    int weaponIndex;    // index into knight's equipped weapons
    int profileIndex;   // index into weapon's attack profiles (e.g., burst vs single)
}
```

**Valid in states**: `CombatPhase.PlayerTurn` (when attacker's combat action is unspent)

**Validation rules**:
- `attacker` must be a living, player-controlled knight whose turn it is
- `attacker` must not be in Despair (hostile knights are AI-controlled)
- `target` must be a valid enemy (alive, on the battlefield)
- `weaponIndex` must reference an equipped weapon
- `profileIndex` must be valid for that weapon
- Target must be within the weapon's range band given current positions
- Knight must have their combat action available (not yet spent this turn)

**On success**:
1. CombatManager resolves the attack: applies current CombatStyle, builds dice pool
2. Rolls dice via Combat RNG stream (SPEC-28)
3. Fires event sequence: `OnAttackRoll` -> `OnExploit` (if applicable) -> `OnHitConfirmed`/`OnMiss` -> `OnDamageApplied` (per layer)
4. Marks the knight's combat action as spent

**On failure**: Returns `CommandResult { success: false, error: "OutOfRange" | "NoActionRemaining" | "InvalidTarget" | ... }`
UI displays the error and allows the player to choose a different action.

---

### RequestMove

```csharp
struct RequestMove {
    CombatantId knight;
    int targetRank;     // 1-4, the desired position on the linear track
}
```

**Valid in states**: `CombatPhase.PlayerTurn` (when knight's move action is unspent)

**Validation rules**:
- `knight` must be a living, player-controlled knight whose turn it is
- `targetRank` must be adjacent to current rank (rank +/- 1)
- Knight must have their move action available (FR-006b: Nod use or weapon switch also consumes move)
- If `targetRank` is occupied by an ally, they swap positions
- If `targetRank` is occupied by an enemy, move is rejected (no melee charge)

**On success**:
1. Knight moves to `targetRank` (swap with ally if occupied)
2. Marks the knight's move action as spent
3. UI updates knight positions

**On failure**: Returns `CommandResult { success: false, error: "NotAdjacent" | "NoMoveRemaining" | "EnemyOccupied" | ... }`

---

### RequestNodUse

```csharp
struct RequestNodUse {
    CombatantId user;
    CombatantId target;
    NodType type;       // Soin, Armure, Energie
}
```

**Valid in states**: `CombatPhase.PlayerTurn` (when user's move action is unspent)

**Validation rules**:
- `user` must be a living, player-controlled knight whose turn it is
- `user` must have at least 1 Nod of the specified `type` remaining (FR-028: 3 of each per mission)
- `target` must be a living ally (or self)
- `target` must be adjacent to `user` (Nods require contact range)
- Move action must be available (Nod use consumes move action per FR-006b)
- Nod de Soin on a target NOT in Agony: restores PS
- Nod de Soin on a target in Agony with Hemorragie: cancels countdown

**On success**:
1. Nod consumed (decrement inventory count)
2. Resource restored: Soin -> PS, Armure -> PA, Energie -> PE
3. Fires `OnNodUsed(user, target, type, amount)`
4. If Nod de Soin on Agony target: fires `OnNodHealFromAgony(user, target)`, cancels Hemorragie
5. If PA restored above 5 and knight was in Fold: fires `OnFoldEnded`
6. Marks the user's move action as spent

**On failure**: Returns `CommandResult { success: false, error: "NoNodsRemaining" | "NotAdjacent" | "NoMoveRemaining" | ... }`

---

### RequestStyleChange

```csharp
struct RequestStyleChange {
    CombatantId knight;
    CombatStyle newStyle;   // Agressif, Defensif, Precis, Couvert, Ambidextre, Akimbo
}
```

**Valid in states**: `CombatPhase.PlayerTurn` (at the start of the knight's turn, before any action)

**Validation rules**:
- `knight` must be a living, player-controlled knight whose turn it is
- `newStyle` must be a valid CombatStyle enum value
- Ambidextre/Akimbo require dual-wielding (two weapons equipped)
- Style change is free (does not consume move or combat action) per FR-007

**On success**:
1. Knight's active CombatStyle updated
2. Style modifiers applied to dice pool calculations for this turn:
   - Agressif: +3 attack dice, -2 Defense, -2 Reaction
   - Defensif: -3 attack dice, +2 Defense
   - Precis: +2nd characteristic added to dice pool
   - Couvert: half dice, +3 Defense (behind cover)
   - Ambidextre: 2 separate attacks at -3 dice each
   - Akimbo: 1 combined attack at -3 dice
3. UI updates style indicator

**On failure**: Returns `CommandResult { success: false, error: "NoDualWield" | "InvalidStyle" | ... }`

---

### RequestAbilityActivate

```csharp
struct RequestAbilityActivate {
    CombatantId knight;
    AbilityId ability;  // TypeSoldat, TypeChasseur, TypeErudit, Shrine, Watchtower, Mechanic, NanoC, Ghost
}
```

**Valid in states**: `CombatPhase.PlayerTurn` (when knight's combat action is unspent, or free action for some abilities)

**Validation rules**:
- `knight` must be a living, player-controlled knight whose turn it is
- `knight` must have the armor class that grants the specified ability (FR-019 to FR-022)
- `knight` must NOT be in Fold state (armor abilities disabled in Fold per FR-013)
- Knight must have sufficient PE to activate:
  - Warrior Type: 1 PE/turn maintenance (FR-019)
  - Paladin Shrine: 2 PE deploy + 1 PE/turn (FR-020)
  - Paladin Watchtower: 2 PE deploy + 1 PE/turn (FR-020)
  - Priest Mechanic: 4 PE one-time cost (FR-021)
  - Priest NanoC: PE cost per spec
  - Rogue Ghost: PE cost per spec (FR-022)
- Mechanic requires a valid ally target (separate targeting step after activation)

**On success**:
1. PE deducted, fires `OnEspoirLost(knight, cost, "AbilityActivation")`
2. Ability effect applied:
   - Warrior Type: +1 to all 3 characteristics in the Aspect for 1 turn
   - Paladin Shrine: +6 CdF aura created at knight's position
   - Paladin Watchtower: extra ranged attack at halved Reaction
   - Priest Mechanic: target ally PA restored by 3D6+6 (contact) or 2D6+6 (ranged)
   - Priest NanoC: cover structure placed
   - Rogue Ghost: stealth activated (Discretion vs enemy Machine)
3. Marks combat action as spent (except Type maintenance which is free)

**On failure**: Returns `CommandResult { success: false, error: "InsufficientPE" | "InFoldState" | "WrongArmorClass" | ... }`

---

### RequestEndTurn

```csharp
struct RequestEndTurn {
    CombatantId knight;
}
```

**Valid in states**: `CombatPhase.PlayerTurn` (at any point during the knight's turn)

**Validation rules**:
- `knight` must be the currently active knight
- Always valid — the player can end their turn at any time, even without using actions

**On success**:
1. Any unused actions are forfeited
2. Turn passes to next combatant in initiative order
3. If knight has active maintenance abilities (Type, Shrine, Watchtower), PE is deducted
4. If knight has Hemorragie, fires `OnHémorragieTick`
5. If knight is in Despair, fires `OnDespairTick`

**On failure**: Returns `CommandResult { success: false, error: "NotYourTurn" }`

---

### RequestWeaponSwitch

```csharp
struct RequestWeaponSwitch {
    CombatantId knight;
    int weaponIndex;    // index of the weapon to switch to
}
```

**Valid in states**: `CombatPhase.PlayerTurn` (when knight's move action is unspent)

**Validation rules**:
- `knight` must be a living, player-controlled knight whose turn it is
- `weaponIndex` must reference a valid weapon in the knight's loadout
- `weaponIndex` must differ from the currently active weapon
- Move action must be available (weapon switch consumes move action per FR-033)

**On success**:
1. Active weapon changes to `weaponIndex`
2. Marks the knight's move action as spent
3. UI updates weapon display and available attack profiles

**On failure**: Returns `CommandResult { success: false, error: "AlreadyEquipped" | "NoMoveRemaining" | "InvalidWeapon" | ... }`

---

### RequestAssist

```csharp
struct RequestAssist {
    CombatantId assistant;
    CombatantId attacker;
    CharacteristicId characteristic;    // the characteristic used for the assist roll
}
```

**Valid in states**: `CombatPhase.PlayerTurn` (during another knight's attack resolution, before dice roll)

**Validation rules**:
- `assistant` must be a living ally who has not yet used their combat action this round
- `attacker` must be the currently active knight about to attack
- `assistant` and `attacker` must be different knights
- `characteristic` must be relevant to the attack context
- Assistant must be in a rank where they can contribute (adjacent or same rank)

**On success**:
1. Assistant's characteristic value added as bonus dice to attacker's pool
2. Assistant's combat action is consumed (they cannot attack on their own turn)
3. Attack proceeds with augmented dice pool

**On failure**: Returns `CommandResult { success: false, error: "NoActionRemaining" | "NotAdjacent" | "InvalidCharacteristic" | ... }`

---

### RequestGrenade

```csharp
struct RequestGrenade {
    CombatantId thrower;
    GrenadeType type;       // Shrapnel, Flashbang, AntiArmor, IEM, Explosive
    int targetRank;         // 1-4
}
```

**Valid in states**: `CombatPhase.PlayerTurn` (when thrower's combat action is unspent)

**Validation rules**:
- `thrower` must be a living, player-controlled knight whose turn it is
- `thrower` must have at least 1 grenade of the specified `type` remaining (FR-034: 5 per mission total)
- `targetRank` must be a valid rank (1-4) containing at least one enemy
- Combat action must be available
- Note: grenades hit ALL combatants at `targetRank` including allies (friendly fire for Explosive/Shrapnel)

**On success**:
1. Grenade consumed (decrement inventory count)
2. Effect applied by type:
   - Shrapnel: area damage to all at rank
   - Flashbang: stun (skip next turn) to all at rank
   - AntiArmor: damage ignoring CdF to all at rank
   - IEM: disable electronic/energy abilities at rank
   - Explosive: high damage + splash to adjacent ranks
3. Fires appropriate damage/status events for each affected combatant
4. Marks the thrower's combat action as spent

**On failure**: Returns `CommandResult { success: false, error: "NoGrenadesRemaining" | "NoActionRemaining" | "InvalidRank" | ... }`

---

### RequestHeroismSpend

```csharp
struct RequestHeroismSpend {
    CombatantId knight;
    HeroismAction action;   // Reroll, DégâtsMaximum, IgnorerAgonie, IgnorerDésespoir, ModeHéroïque
}
```

**Valid in states**: `CombatPhase.PlayerTurn` or during event resolution (context-dependent)

**Validation rules**:
- `knight` must be a living, player-controlled knight
- Knight must have sufficient Heroisme points:
  - Reroll: 1 Heroisme (reroll all failed dice in a just-completed roll)
  - DegatsMaximum: 1 Heroisme (maximize weapon damage dice)
  - IgnorerAgonie: 1 Heroisme (ignore Agony state for 1 turn)
  - IgnorerDesespoir: 1 Heroisme (ignore Despair penalties for 1 turn)
  - ModeHeroique: 6 Heroisme (enter Heroic Mode for rest of combat)
- Context-specific: Reroll only valid immediately after a dice roll, DegatsMaximum only during damage calculation

**On success**:
1. Heroisme points deducted
2. Effect applied:
   - Reroll: failed dice are rerolled, new successes added
   - DegatsMaximum: all weapon damage dice set to maximum value
   - IgnorerAgonie: Agony penalties suppressed for 1 turn
   - IgnorerDesespoir: Despair penalties suppressed for 1 turn
   - ModeHeroique: fires `OnHeroicModeActivated(knight)`, all penalties ignored for rest of combat
3. Corresponding events fired

**On failure**: Returns `CommandResult { success: false, error: "InsufficientHeroisme" | "InvalidContext" | ... }`

---

## Deployment Commands

### RequestDeployment

```csharp
struct RequestDeployment {
    CombatantId knight;
    int rank;           // 1-4 (front to back)
}
```

**Valid in states**: `GamePhase.Deployment`

**Validation rules**:
- `knight` must be a member of the selected 4-knight squad
- `rank` must be 1-4
- If `rank` is already occupied by another knight, they swap positions
- All 4 knights must be placed before deployment can be confirmed

**On success**:
1. Knight assigned to `rank`
2. Fires `OnKnightDeployed(knight, rank)`
3. If rank was occupied, the displaced knight moves to the placing knight's previous rank
4. UI updates formation display

**On failure**: Returns `CommandResult { success: false, error: "NotInSquad" | "InvalidRank" | ... }`

---

### ConfirmDeployment

```csharp
struct ConfirmDeployment { }
```

**Valid in states**: `GamePhase.Deployment` (after all 4 knights placed)

**Validation rules**:
- All 4 squad knights must have assigned ranks
- No two knights occupy the same rank

**On success**:
1. Deployment locked
2. Game state transitions from `Deployment` to `CombatPhase`
3. Enemy placement resolved
4. Initiative order calculated and turn queue populated
5. First combatant's turn begins

**On failure**: Returns `CommandResult { success: false, error: "IncompleteDeployment" }`

---

## Mission Commands

### RequestBranchChoice

```csharp
struct RequestBranchChoice {
    int nodeIndex;      // the node where branching occurs
    string branchId;    // identifier for the chosen branch (e.g., "nest", "ruins")
}
```

**Valid in states**: `GamePhase.MissionMap` (at a branching node, specifically node 2 per FR-026)

**Validation rules**:
- `nodeIndex` must be the current node in the mission graph
- `branchId` must be a valid branch option for that node
- The node must be a branching node (not all nodes offer choices)

**On success**:
1. Mission graph advances along the chosen branch
2. Fires `OnBranchChosen(nodeIndex, branchId)`
3. Next node loaded (fires `OnNodeEntered` when ready)

**On failure**: Returns `CommandResult { success: false, error: "InvalidBranch" | "NotAtBranchNode" | ... }`

---

### RequestRetreat

```csharp
struct RequestRetreat { }
```

**Valid in states**: `GamePhase.MissionMap` (nodes 1-4 only, NOT boss node)

**Validation rules**:
- Current node index must be between 1 and 4 (retreat is not available at boss node 5)
- At least one knight must be alive

**On success**:
1. Mission ends as failure (no bonus PG for incomplete nodes)
2. PG earned for completed nodes is kept
3. Fires `OnMissionComplete(false, knightsAlive)`
4. Transition to Camelot (HP/PA restored, PE preserved per FR-029)

**On failure**: Returns `CommandResult { success: false, error: "CannotRetreatFromBoss" | "AllKnightsDead" | ... }`

---

### RequestNodBetweenNodes

```csharp
struct RequestNodBetweenNodes {
    CombatantId user;
    CombatantId target;
    NodType type;       // Soin, Armure, Energie
}
```

**Valid in states**: `GamePhase.BetweenNodes` (after completing a node, before entering the next)

**Validation rules**:
- `user` must be a living knight in the squad
- `user` must have at least 1 Nod of the specified `type` remaining
- `target` must be a living knight in the squad (or self)
- No adjacency requirement between nodes (free action per FR-028)
- No action cost (free action between nodes)

**On success**:
1. Nod consumed (decrement inventory count)
2. Resource restored: Soin -> PS, Armure -> PA, Energie -> PE
3. Fires `OnNodUsed(user, target, type, amount)`
4. If healing from Agony: fires `OnNodHealFromAgony(user, target)`
5. If PA restored above 5 and in Fold: fires `OnFoldEnded`

**On failure**: Returns `CommandResult { success: false, error: "NoNodsRemaining" | "TargetDead" | ... }`

---

## Game State Machine — Command Validity

```
┌─────────────────┐
│   MainMenu      │  No commands accepted
└────────┬────────┘
         v
┌─────────────────┐
│   Camelot       │  Roster/shop commands (out of scope for this contract)
└────────┬────────┘
         v
┌─────────────────┐
│  SquadSelect    │  Squad selection commands (out of scope for this contract)
└────────┬────────┘
         v
┌─────────────────┐
│  MissionMap     │  RequestBranchChoice, RequestRetreat
└────────┬────────┘
         v
┌─────────────────┐
│  BetweenNodes   │  RequestNodBetweenNodes
└────────┬────────┘
         v
┌─────────────────┐
│  Deployment     │  RequestDeployment, ConfirmDeployment
└────────┬────────┘
         v
┌─────────────────┐
│  CombatPhase    │  RequestAttack, RequestMove, RequestNodUse, RequestStyleChange,
│  (PlayerTurn)   │  RequestAbilityActivate, RequestEndTurn, RequestWeaponSwitch,
│                 │  RequestAssist, RequestGrenade, RequestHeroismSpend
└────────┬────────┘
         v
┌─────────────────┐
│  CombatPhase    │  No player commands (AI resolves)
│  (EnemyTurn)    │  RequestHeroismSpend allowed (reactive: Reroll on defense)
└────────┬────────┘
         v
┌─────────────────┐
│  NodeComplete   │  Transitions to BetweenNodes or MissionComplete
└─────────────────┘
```

---

## Command Result Contract

```csharp
struct CommandResult {
    bool success;
    string error;       // null if success, machine-readable error code if failed
    string message;     // optional human-readable message for UI display
}
```

**Error codes** are string constants (not enums) to allow extensibility:
- `"NotYourTurn"` — command sent for a knight who is not the active combatant
- `"NoActionRemaining"` — move or combat action already spent
- `"NoMoveRemaining"` — move action consumed by Nod/weapon switch/move
- `"InvalidTarget"` — target does not exist or is already dead
- `"OutOfRange"` — weapon range does not reach target's rank
- `"InsufficientPE"` — not enough Espoir to activate ability
- `"InFoldState"` — armor abilities disabled while in Guardian/Fold mode
- `"WrongArmorClass"` — knight does not have the requested ability
- `"NoNodsRemaining"` — all Nods of that type consumed
- `"NotAdjacent"` — target not in adjacent rank (for Nod use in combat)
- `"AlreadyEquipped"` — trying to switch to already-active weapon
- `"InvalidWeapon"` — weapon index out of bounds
- `"InvalidRank"` — rank not in 1-4 range
- `"NoDualWield"` — Ambidextre/Akimbo style requires two weapons
- `"InsufficientHeroisme"` — not enough Heroisme for the requested action
- `"InvalidContext"` — Heroism action not applicable at this moment
- `"IncompleteDeployment"` — not all knights placed
- `"InvalidBranch"` — branch ID not valid for current node
- `"CannotRetreatFromBoss"` — retreat not allowed at node 5
- `"NoGrenadesRemaining"` — all grenades of that type consumed
- `"InvalidStyle"` — unrecognized combat style
- `"EnemyOccupied"` — cannot move into enemy-occupied rank
- `"NotInSquad"` — knight not part of the deployed squad
- `"TargetDead"` — target knight is dead
- `"AllKnightsDead"` — no living knights remain
- `"NotAtBranchNode"` — current node does not support branching
- `"InvalidCharacteristic"` — characteristic not applicable for assist
