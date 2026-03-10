# EventBus Events Contract

**Source**: SPEC-27 (UI & Presentation Layer) / SPEC-32 (UI is read-only view + command sender)
**Purpose**: Typed C# events that decouple combat systems from UI and cross-system observers.
**Pattern**: `EventBus<T>.Raise(payload)` / `EventBus<T>.Subscribe(handler)`

All events are fire-and-forget. Subscribers MUST NOT mutate combat state in response to events;
they are notifications only. The CombatManager is the sole authority for state mutations.

---

## Combat Events

```csharp
// Fired after dice roll resolved (SPEC-04 dice resolution)
// Subscribers: UI (dice animation), AI (threat assessment), logging
// Fires BEFORE OnHitConfirmed or OnMiss — always fires for every attack roll.
event Action<CombatantId, CombatantId, int> OnAttackRoll; // attacker, target, successes

// Attack successes STRICTLY EXCEED Defense (melee) or Reaction (ranged) threshold (FR-009)
// Subscribers: UI (hit VFX, damage numbers), AI (aggro update), motivation detector
// Fires AFTER OnAttackRoll. Followed by OnDamageApplied for each damage layer.
event Action<CombatantId, CombatantId, DamageResult> OnHitConfirmed; // attacker, target, damage

// Attack fails to exceed threshold (successes <= Defense/Reaction)
// Subscribers: UI (miss text, whiff animation), AI (threat reassessment)
// Fires AFTER OnAttackRoll. Mutually exclusive with OnHitConfirmed for same attack.
event Action<CombatantId, CombatantId> OnMiss; // attacker, target

// ALL dice in the pool show even results on an attack roll (SPEC-04 Exploit rule)
// Subscribers: UI (exploit VFX, bonus dice animation), logging
// Fires AFTER OnAttackRoll but BEFORE OnHitConfirmed. Reroll dice are added to success count.
event Action<CombatantId, CombatantId, int> OnExploit; // attacker, target, rerollSuccesses

// After CdF/PA/PS reduction applied through the damage pipeline (FR-011)
// Subscribers: UI (health bar update, damage number popup), AI (flee threshold check)
// Fires once per damage layer in order: CdF reduction -> PA absorption -> PS passthrough.
// DamageLayer enum: { CdF, PA, PS }
event Action<CombatantId, int, DamageLayer> OnDamageApplied; // target, amount, layer
```

### Event Flow — Single Attack Resolution

```
OnAttackRoll(attacker, target, successes)
  ├─ [if all dice even] OnExploit(attacker, target, rerollSuccesses)
  ├─ [if hit] OnHitConfirmed(attacker, target, damageResult)
  │     ├─ OnDamageApplied(target, cdfReduction, CdF)
  │     ├─ OnDamageApplied(target, paAbsorbed, PA)
  │     ├─ OnDamageApplied(target, psPassthrough, PS)   // 1 PS per 5 PA absorbed
  │     ├─ [if PA reaches 0] OnFoldTriggered(target)
  │     ├─ [if PS reaches 0] OnAgonyEntered(target)
  │     └─ [if Bande] OnDamageApplied(bande, violence, Cohesion)
  └─ [if miss] OnMiss(attacker, target)
```

---

## State Transition Events

```csharp
// PA reaches 0, Guardian Suit activated (FR-013, SPEC-10)
// Subscribers: UI (fold VFX, armor icon change), AI (target priority shift)
// Fires immediately when PA depleted. Armor class abilities are disabled in Fold state.
event Action<CombatantId> OnFoldTriggered;

// Fold state exits when PA restored above 5 (via Nod d'Armure or Priest Mechanic)
// Subscribers: UI (fold VFX removal, armor icon restored), AI (target reprioritize)
event Action<CombatantId> OnFoldEnded;

// PS reaches 0, injury roll triggered (FR-014, SPEC-08)
// Subscribers: UI (agony overlay, injury roll animation), AI (ignore incapacitated target)
// Fires AFTER final OnDamageApplied to PS layer.
event Action<CombatantId> OnAgonyEntered;

// Injury table result determined after Agony roll
// Subscribers: UI (injury result display, body part highlight), logging
// injuryType: e.g. "Hemorragie", "FractureBras", "TraumatismeCranien", etc.
// bodyPart: e.g. "Torse", "BrasDroit", "Tete", etc.
event Action<CombatantId, string, string> OnInjuryRolled; // knight, injuryType, bodyPart

// Hemorragie (internal bleeding) countdown starts — 3 turns to permadeath (FR-015)
// Subscribers: UI (bleeding icon + turn counter), AI (ally prioritize healing)
// turnsRemaining starts at 3.
event Action<CombatantId, int> OnHémorragieStart; // knight, turnsRemaining (3)

// Hemorragie tick at start of bleeding knight's turn
// Subscribers: UI (update turn counter, urgency escalation), AI (healing priority)
// If turnsRemaining reaches 0 and no Nod de Soin applied, fires OnPermadeath next.
event Action<CombatantId, int> OnHémorragieTick; // knight, turnsRemaining (each turn)

// Permanent death — knight is removed from roster forever (FR-031)
// Subscribers: UI (death animation, roster removal), save system (mark knight dead),
//              mission flow (check squad wipe)
// Fires when: Hemorragie countdown expires, or certain catastrophic injury results.
event Action<CombatantId> OnPermadeath;
```

---

## Despair Events (SPEC-07, FR-016)

```csharp
// PE reaches 0, knight becomes hostile for 1D6 turns
// Subscribers: UI (despair overlay, hostile indicator), AI (treat as enemy),
//              turn order (knight now attacks allies)
// duration is rolled via Combat RNG stream (1D6).
event Action<CombatantId, int> OnDespairEntered; // knight, duration (1D6 turns)

// Despair tick at start of despairing knight's turn
// Subscribers: UI (update turn counter), AI (stabilization priority)
// When turnsRemaining reaches 0, despair ends naturally.
event Action<CombatantId, int> OnDespairTick; // knight, turnsRemaining

// Ally attempts stabilization roll on despairing knight
// Subscribers: UI (stabilization animation, success/fail feedback), logging
// success = true: despair ends immediately, knight regains 1 PE.
// success = false: no effect, stabilizer's action is consumed.
event Action<CombatantId, CombatantId, bool> OnStabilizationAttempt; // stabilizer, target, success
```

---

## Enemy / Boss Events

```csharp
// Boss transitions between combat phases (FR-025, SPEC-14)
// Subscribers: UI (phase transition cinematic, new ability display), AI (update behavior),
//              spawn system (reinforcements)
// Fires when boss PS reaches 0 in current phase. Boss gains new stats/abilities.
// Ordering: fires BEFORE any reinforcement spawn events.
event Action<CombatantId, int, int> OnPhaseTransition; // boss, fromPhase, toPhase

// Bande automatic escalating damage applied to all knights (FR-023, SPEC-11)
// Subscribers: UI (debordement VFX on target rank), AI (bande threat display)
// targetRank: the rank receiving Debordement damage this turn.
// Damage formula: Turn N = N x baseDamage.
event Action<CombatantId, int> OnDébordement; // bande, targetRank

// Bande Cohesion reaches 0 — swarm destroyed (FR-023)
// Subscribers: UI (swarm death animation), AI (remove from threat list),
//              mission flow (check encounter complete)
event Action<CombatantId> OnBandeEliminated; // bande

// Knight Peur (fear) test against La Dame aura or similar ability
// Subscribers: UI (fear test result, debuff icon if failed), AI (flee behavior)
// passed = false: knight suffers dice penalty on all rolls while in range.
event Action<CombatantId, int, bool> OnPeurTest; // knight, peurLevel, passed
```

---

## Mission Events

```csharp
// Player enters a new node in the mission graph (FR-026)
// Subscribers: UI (node transition, encounter setup screen), save system (checkpoint),
//              music system (track change)
// EncounterType enum: { Combat, NarrativeEvent, Boss }
event Action<int, EncounterType> OnNodeEntered; // nodeIndex, nodeType

// Player chooses a branch at node 2 (FR-026 branching)
// Subscribers: UI (branch transition animation), mission graph (advance state),
//              narrative system (load branch content)
event Action<int, string> OnBranchChosen; // nodeIndex, branchId

// Mission ends — success or failure
// Subscribers: UI (victory/defeat screen), save system (persist results),
//              Camelot transition (restore HP/PA, preserve PE per FR-029)
// success = false when all 4 knights incapacitated.
event Action<bool, int> OnMissionComplete; // success, knightsAlive

// Knight placed on a rank during deployment phase
// Subscribers: UI (knight sprite placement), combat system (register position)
// rank: 1-4 (front to back)
event Action<CombatantId, int> OnKnightDeployed; // knight, rank
```

---

## Espoir Events (SPEC-07, FR-017, FR-018)

```csharp
// Knight gains Espoir (PE) from any source
// Subscribers: UI (PE bar update, gain popup), motivation detector
// source examples: "NodEnergie", "NarrativeEvent", "Stabilization", "MotivationTrigger"
event Action<CombatantId, int, string> OnEspoirGained; // knight, amount, source

// Knight loses Espoir (PE) from any source
// Subscribers: UI (PE bar update, loss popup), despair threshold check, AI
// source examples: "Anatheme", "AllyDeath", "DespairProximity", "AbilityMaintenance"
// Ordering: fires BEFORE OnDespairEntered if PE reaches 0.
event Action<CombatantId, int, string> OnEspoirLost; // knight, amount, source

// Knight's archetype motivation condition met (SPEC-07)
// Subscribers: UI (motivation popup, PE gain animation), logging
// MotivationType corresponds to one of 17 archetype motivations.
// Fires BEFORE the corresponding OnEspoirGained event.
event Action<CombatantId, MotivationType> OnMotivationTriggered; // knight, motivationType

// Knight spends 6 Heroisme to enter Heroic Mode (FR-018)
// Subscribers: UI (heroic mode VFX, stat overlay), AI (threat escalation)
// Knight ignores all penalties (injuries, despair, PE malus) for the remainder of combat.
event Action<CombatantId> OnHeroicModeActivated; // knight
```

---

## Nod Events (SPEC-33, FR-028)

```csharp
// Knight uses a Nod consumable on a target
// Subscribers: UI (nod animation, resource bar update, inventory count update),
//              AI (healing awareness), motivation detector (for archetype triggers)
// NodType enum: { Soin, Armure, Energie }
// amount: actual HP/PA/PE restored after calculation.
// Consumes the move action in combat (FR-006b). Free action between nodes (FR-028).
event Action<CombatantId, CombatantId, NodType, int> OnNodUsed; // user, target, type, amount

// Nod de Soin used on a knight in Agony — special case that triggers VOEU_OURS motivation
// Subscribers: UI (heal-from-agony VFX), motivation detector (VOEU_OURS check),
//              hemorragie system (cancel countdown)
// Fires AFTER OnNodUsed. Cancels active Hemorragie if present.
event Action<CombatantId, CombatantId> OnNodHealFromAgony; // user, target
```

---

## Enum Reference (for event payloads)

```csharp
enum DamageLayer { CdF, PA, PS, Cohesion }
enum EncounterType { Combat, NarrativeEvent, Boss }
enum NodType { Soin, Armure, Energie }
enum CombatStyle { Agressif, Defensif, Precis, Couvert, Ambidextre, Akimbo }
enum HeroismAction { Reroll, DégâtsMaximum, IgnorerAgonie, IgnorerDésespoir, ModeHéroïque }
enum MotivationType { /* 17 archetype motivations — defined in data model */ }
```

---

## Subscriber Matrix

| Event | UI | AI | Motivation Detector | Save System | Combat Manager |
|-------|----|----|---------------------|-------------|----------------|
| OnAttackRoll | dice animation | threat assessment | - | - | - |
| OnHitConfirmed | hit VFX, damage numbers | aggro update | ally-hit triggers | - | - |
| OnMiss | miss text | threat reassess | - | - | - |
| OnExploit | exploit VFX | - | - | - | - |
| OnDamageApplied | health bar update | flee check | - | - | - |
| OnFoldTriggered | fold VFX | target shift | - | - | - |
| OnFoldEnded | fold removal | reprioritize | - | - | - |
| OnAgonyEntered | agony overlay | ignore target | ally-down triggers | - | - |
| OnInjuryRolled | injury display | - | - | - | - |
| OnHémorragieStart | bleed icon + counter | heal priority | - | - | - |
| OnHémorragieTick | counter update | heal urgency | - | - | - |
| OnPermadeath | death animation | remove target | ally-death triggers | mark dead | check squad wipe |
| OnDespairEntered | despair overlay | treat hostile | - | - | - |
| OnDespairTick | counter update | stabilize priority | - | - | - |
| OnStabilizationAttempt | result feedback | - | stabilize triggers | - | - |
| OnPhaseTransition | phase cinematic | update behavior | - | - | spawn reinforcements |
| OnDébordement | debordement VFX | - | - | - | - |
| OnBandeEliminated | death animation | remove threat | - | - | check encounter end |
| OnPeurTest | fear result | flee behavior | - | - | - |
| OnNodeEntered | node transition | - | - | checkpoint | - |
| OnBranchChosen | branch animation | - | - | - | - |
| OnMissionComplete | victory/defeat | - | - | persist results | - |
| OnKnightDeployed | sprite placement | - | - | - | register position |
| OnEspoirGained | PE bar update | - | - | - | - |
| OnEspoirLost | PE bar update | - | despair threshold | - | - |
| OnMotivationTriggered | motivation popup | - | (source) | - | - |
| OnHeroicModeActivated | heroic VFX | threat escalation | - | - | - |
| OnNodUsed | nod animation | healing awareness | archetype triggers | - | - |
| OnNodHealFromAgony | heal VFX | - | VOEU_OURS check | - | cancel hemorragie |
