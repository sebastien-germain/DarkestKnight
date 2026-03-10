# UI & Presentation Layer
> **SPECs:** 26, 27, 31
> **Domain:** `Scripts/UI`
> **Dependencies:** All combat & progression systems (read-only view layer)
> **Depended on by:** None (presentation only, no gameplay logic)
---
[v7] SPEC-26: UI Screen Flow (High-Level)
[v7] Minimum Viable Screen Flow for Agent Implementation:

Combat HUD requires: Initiative timeline (top), Knight stat bars (bottom), Rank display (center), Action menu (context), Status effects (per-knight icons), Tarot hand (collapsible panel).
Deployment Screen: 4 rank slots, drag-and-drop knight placement, cover indicators, enemy preview.
Camelot Hub: Armory (buy/sell weapons+modules), Knight roster (stats, injuries, equipment), [v8.9] Infirmary (Cybernetic Implant, Reconstruction Therapy — see SPEC-25), Mission select, Tarot management.
Agent Note: Full UI wireframes are out of scope for this spec. The above flow defines the minimum screen set. UI layout decisions are delegated to the UI agent with the constraint that all SPEC-referenced UI elements (HUD countdown, initiative timeline, Tarot hand) must be present.
[v7] SPEC-27: Audio/VFX Event Hooks
[v7] Placeholder event trigger points for the combat chain and state transitions. Agents must fire these events at the specified moments. Actual audio/VFX assets are out of scope for this spec.
COMBAT EVENTS:
  OnAttackRoll(attacker, target, successes) — after dice roll resolved
  OnHitConfirmed(attacker, target, damage) — attack exceeds Defense/Reaction
  OnMiss(attacker, target) — attack fails to exceed threshold
  OnExploit(attacker, target, rerollSuccesses) — ALL dice show even on attack roll (SPEC-04 special outcome). Fired after re-roll resolves.
  OnCriticalHit removed — duplicate of OnExploit(attacker, target, rerollSuccesses) which already fires on all-even dice rolls.
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
NOD EVENTS (SPEC-33):
  OnNodUsed(user, target, nodType, restoreAmount) — Nod consumed and effect applied
  OnNodHealFromAgony(user, target) — Nod de Soin brings knight out of Agony (triggers VOEU_OURS check)
[v8] DESPAIR EVENTS:
  OnDespairEntered(knight, duration) — PEs reaches 0, hostile AI begins (SPEC-07)
  OnDespairTick(knight, turnsRemaining) — Despaired knight's turn ends, countdown decremented
  OnStabilizationAttempt(stabilizer, target, success) — ally rolls to end Despair

---

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
at least target + action type icon/name (exact numbers optional unless you want full telegraphing). Active Countdown Timers (mandatory on affected combatant): Despair (turns remaining before permanence), Hémorragie (turns to death — highest priority, red pulsing), Dégâts Continus (turns remaining + damage per tick), Choc/Parasitage (actions lost remaining). Display format: icon + number badge on combatant portrait. Hémorragie and Despair countdowns must be visible at ALL times, not just on hover — these are life-or-death timers
Mission Node Screen
shows remaining nodes, branch choice, party status summary (PS/PA/PE/PEs), and explicit warning that no auto-heal occurs between nodes (if that’s your rule).
Hub (Camelot)
shows injury list + treatment costs, current roster condition summary, and what resets vs persists.
### Acceptance Criteria
SPEC-31-AC1: Player can always answer: “Why did I miss?” and “Where did the damage go?” without guessing.
SPEC-31-AC2: Every combat action produces a log entry including roll result + damage breakdown.
SPEC-31-AC3: Any disabled action must display a reason string (not just greyed out).

