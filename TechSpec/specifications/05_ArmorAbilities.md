# Armor Ability System
> **SPECs:** 17
> **Domain:** `Scripts/Combat`
> **Dependencies:** [01_KnightDataModel.md](./01_KnightDataModel.md) (SPEC-01), [02_PositionAndTurns.md](./02_PositionAndTurns.md) (SPEC-03), [03_CombatRolls.md](./03_CombatRolls.md) (SPEC-04)
> **Depended on by:** [04_ArmorAndState.md](./04_ArmorAndState.md) (SPEC-10)
---
## [v2] SPEC-17: Armor Ability System (§4.5)
### Purpose
Manages all demo armor signature abilities. Each armor class has distinct abilities with PE costs, action requirements, and tactical effects.

### 17.1 Warrior — Les Types
Interchangeable OD boosts linked to Aspects.

| Field | Value |
| --- | --- |
| Types at creation | 3 from 5 (demo: Soldier + Hunter + 1 random from {Scholar, Herald, Scout}). Full game: player chooses 3. |
| Activation | 1 Movement Action |
| PE cost | 1 PE/turn combat, 6 PE/scene outside |
| Duration | 1 turn (until start of next turn). Spend PE to prolong, no action. |
| Effect | +1 OD to all 3 Characteristics of the linked Aspect |
| OD stacking | [v8.9] Normal OD hard cap is 5 (from generation + any non-Type source). Warrior Type +1 is the **only** exception that can exceed this cap, pushing effective OD to **6**. No other source in demo can exceed 5. The +1 is a runtime bonus applied while the Type is active — it is not persisted to `odLevels`. When the Type deactivates, OD returns to its base value. Formula: `effectiveOD = min(baseOD, 5) + (typeActive && typeMatchesAspect ? 1 : 0)`. Maximum effective OD in demo: **6**. |
| Constraint | 1 Type active at a time. Max 1 switch per turn. |
| Fold interaction | Type deactivates on fold. Cannot reactivate until armor restored. |



| Type | Aspect | Characteristics Boosted |
| --- | --- | --- |
| Soldier | CHAIR | Force, Endurance, Déplacement |
| Hunter | BÊTE | Hargne, Combat, Instinct |
| Scholar | MACHINE | Tir, Savoir, Technique |
| Herald | DAME | Aura, Parole, Sang-Froid |
| Scout | MASQUE | Discrétion, Dextérité, Perception |


Demo generation: Default 3 Types = Soldier + Hunter + 1 random from {Scholar, Herald, Scout} (the 3 non-default Types). No duplicate Types allowed. Each Warrior knight rolls independently — the two Warriors in the roster may end up with different 3rd Types or the same one by chance.  
Evolutions (locked): 150 PG: no-action activation. 200 PG: all 5 Types. 250 PG: +2 OD per Type (would push max effective OD to 7 — out of demo scope, scaffolded only).

### Acceptance Criteria
[v8.9] Warrior with Combat baseOD 5 activates Hunter Type (+1 OD to Bête aspect): effective Combat OD = 6. Attack with Combat as Base: 6 auto-successes from Combat OD alone.  
[v8.9] Warrior with Combat baseOD 3 activates Hunter Type: effective Combat OD = 4. Deactivates Type: OD returns to 3.  
[v8.9] Warrior with Tir baseOD 5 activates Scholar Type (+1 OD to Machine): effective Tir OD = 6. Switches to Hunter Type: Tir OD returns to 5, Combat/Instinct/Hargne ODs get +1.  
[v8.9] Non-Warrior knight (e.g., Paladin) with Tir baseOD 5: OD stays at 5. No Type ability available. Hard cap enforced.  

Double switch in 1 turn: rejected.
Fold: Type deactivates.

### 17.2 Paladin — Shrine + Watchtower
Shrine — defensive force field dome.

| Field | Value |
| --- | --- |
| Activation | [v8.9] Free (no action cost). Placing a Shrine never costs an action — not on initial placement, not on re-placement. |
| PE cost | [v8.9] 2 PE to place. +1 PE per turn to maintain (deducted at start of Paladin's turn). |
| Duration | [v8.9] Persists until disabled. Shrine stays active as long as the Paladin can maintain it (1 PE/turn). |
| Effect | +6 CdF to all allied combatants at the Shrine's rank, rank−1, and rank+1 (up to 3 ranks total, clamped to valid ranks 1–4). The Shrine's own rank IS included. Example: Shrine at Rank 2 covers Ranks 1, 2, and 3. Shrine at Rank 1 covers Ranks 1 and 2 only (no Rank 0). |
| Entry restriction | **SCAFFOLDED — entry restriction deferred to post-demo.** The tabletop rule (enemies with Force<5 or Chair<10 can't enter the Shrine zone) requires redesign for the DD linear rank system, where enemies occupy the opposite side of the track and don't physically "enter" knight ranks. Future redesign: define how the Shrine zone interacts with enemy movement, contact attacks, and Charge Brutale across the center line. For demo: Shrine provides the +6 CdF bonus only — no movement restriction on enemies. Bande Débordement CdF still applies (Shrine bonus is part of total CdF per SPEC-06). |
| [v8.9] Shrine anchors to the rank where it was placed and cannot be moved or repositioned. The Paladin is free to move away after placing the Shrine. CdF bonus applies to all allies at the Shrine's rank ±1 regardless of Paladin position. | Shrine is fixed to placed rank. Paladin free to move away. |
| Limit | 1 per Paladin. No overlap. |
| vs Ignore CdF | Shrine CdF also bypassed. |

[v8.9] **Shrine Disable Conditions** (any of the following disables the Shrine immediately):
- Paladin's **PE reaches 0**: no energy to maintain → Shrine disabled. [v8.9] This includes maintenance deduction: if Paladin has 1 PE at turn start, maintenance deducts 1 → PE = 0 → Shrine disabled immediately.
- Paladin enters **Fold**: Shrine disabled. Does NOT auto-reactivate when exiting Fold.
- Paladin enters **Agony**: Shrine disabled.
- Paladin enters **Despair** (PEs = 0): Shrine disabled. (Despair is a separate condition from PE = 0 — both independently disable the Shrine.)
- Paladin **dies**: Shrine disabled permanently.
- Paladin **voluntarily disables** the Shrine (Free Action).

**Re-placement after disable:** Same as initial placement — Free Action, 2 PE cost. No additional action cost. The Paladin simply places a new Shrine at any rank (subject to the same rules as initial placement).


Watchtower — stationary fire platform.

| Field | Value |
| --- | --- |
| Activation | 1 Movement Action |
| PE cost | 2 PE to activate. +1 PE per extra shot. |
| Duration | Until deactivated (free action) |
| Effect | [v8.9] Cannot move. **Reaction halved:** Paladin's base Reaction score is halved (floor: `floor(baseReaction / 2)`). Rank and cover modifiers are applied AFTER halving. **Modifier application order** (per Knight tabletop rule: Division/Multiplication always before Addition/Subtraction): (1) Halve base Reaction → `floor(baseReaction / 2)`, (2) apply rank modifiers (e.g., Rank 4 = −1), (3) apply cover modifiers (+3 if at cover rank), (4) apply style modifiers (Mise à Couvert +2, Défensif +0 React, Agressif −2), (5) apply Barrage debuffs (−X from each active Barrage source). Barrage is applied last as it is an external subtractive effect. All results floor at 0. Halving persists while Watchtower is active. On deactivation: base Reaction is fully restored. **+1 ranged-only Combat Action per turn (starting the turn AFTER activation).** On the activation turn, the Paladin gains no extra action (they spent 1 Movement Action to activate). Starting on the Paladin's next turn and every subsequent turn while Watchtower is active: the Paladin gains +1 Combat Action that can ONLY be used for a ranged attack (not melee, not movement substitution, not abilities). This is separate from the standard 1 Combat Action + 1 Movement Action. |
| Style restriction | Incompatible with Ambidextre. |


Lente et Lourde (passive) — Cannot use Course/Déplacement Silencieux/Saut modules. Discrétion and Déplacement tests +1 difficulty.
Evolutions (locked): 150 PG: slot ranged weapons on arms. 200 PG: Shrine +8 CdF. 250 PG: Lente et Lourde removed.

### Acceptance Criteria
[v8.9] Shrine placement: Free Action, 2 PE cost. +6 CdF to all allies at Shrine rank ±1. Shrine anchors to placed rank. Allies who move beyond rank gap > 1 lose the bonus.
[v8.9] Shrine maintenance: Paladin places Shrine (2 PE). Next turn start: −1 PE maintenance. Paladin at 1 PE: maintenance deducts 1 PE → 0 PE → Shrine disabled (no energy).
[v8.9] Shrine Fold: Paladin enters Fold → Shrine disabled. Paladin exits Fold → Shrine NOT active. Paladin places new Shrine: Free Action, 2 PE. No combat action needed.
[v8.9] Shrine Agony: Paladin enters Agony → Shrine disabled.
[v8.9] Shrine Despair: Paladin enters Despair → Shrine disabled.
Watchtower: immobile, +1 ranged action next turn.
[v8.9] Watchtower Reaction: Paladin base Reaction 5 activates Watchtower. Effective base Reaction = floor(5/2) = 2. At Rank 4 (−1 Reaction modifier): effective Reaction = 2 − 1 = 1. Paladin deactivates Watchtower: base Reaction restored to 5, effective at Rank 4 = 5 − 1 = 4.
[v8.9] Watchtower + Barrage: Paladin base Reaction 6 in Watchtower. Halved = floor(6/2) = 3. Rank modifier: −0 (Rank 2). Enemy Barrage 2 active on Paladin. Order: (1) halve → 3, (2) rank → 3, (3) Barrage −2 → 1. Final Reaction = 1. If Barrage were applied before halving: floor((6−2)/2) = 2 — WRONG. Division always before subtraction.
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
| Cover Wall | Simple | 3 | Movement | Creates hasCover = true at target rank. 10 turns. On expiry, Priest Fold, or Priest death: cover is removed — triggers Cover Removal Procedure (SPEC-03): `hasCover = false`, combatants using Mise à Couvert forced to Standard, others unaffected. See SPEC-03 Cover System. |
| Barricade | Simple | 3 | Movement | [v8.9] Placed at a target rank. Blocks **all involuntary (forced) displacement** through, into, or out of that rank. Any effect that would forcibly move a combatant through, into, or out of the Barricade rank is negated — the combatant stays at their current position. **Voluntary movement is NOT blocked** — combatants can freely walk through a Barricade rank on their own turn using Movement or Combat Actions. Duration: 10 turns. Examples of blocked movement: Choc push, enemy charge knockback, any capacity that forces rank change. Examples of allowed movement: knight moves into/through the rank voluntarily, enemy advances through voluntarily. |
| Repair Platform | Detailed | 6 | Combat | Mechanic repairs here gain +1D6 PA. 10 turns. |
| Nano-Trap | Mechanical | 9 | Full Turn | Next enemy entering loses 1 action. Single use. |


Mechanic — repairs allied armor PA.

[v8.9] **Agony targeting rule:** Priest Mechanic **cannot** target a knight in Agony (PS = 0). Agony requires PS restoration to exit — PA restoration has no immediate benefit for an incapacitated knight. Only Nod de Soin (SPEC-33) can target an Agony knight. Mechanic CAN target a knight in Fold (PA = 5, Guardian Suit) — this is the secondary Fold exit path after Nod d'Armure.

| Range | Activation | PE | Effect |
| --- | --- | --- | --- |
| Contact (gap = 0, same rank as Priest) | 1 Movement Action | 4 | Restores 3D6+6 PA |
| Long (any allied knight rank, gap ≤ 6 — effectively unlimited on 4-rank track) | 1 Movement Action | 4 | Restores 2D6+6 PA |


Evolutions (locked): 150 PG: NanoC 1hr duration. 200 PG: Mechanic +1D6+6. 250 PG: create vehicles.

### Acceptance Criteria

Mechanic contact: 3D6+6 PA. Range: 2D6+6 PA.  
NanoC expires at turn 10: effect removed.  
[v8.9] NanoC Cover Wall expires: `hasCover = false` at rank. K1 at that rank using Mise à Couvert: forced to Standard. K2 at that rank using Agressif: style unchanged.  
[v8.9] Priest Fold: all active NanoC constructions removed (Cover Wall, Barricade, etc.). Cover Removal Procedure triggered for each Cover Wall.  
[v8.9] Priest dies: same as Fold — all NanoC constructions removed immediately.  
Fold: NanoC and Mechanic unavailable.  

### 17.4 Rogue — Mode Ghost
Stealth system with alpha-strike damage bonus.

#### Ghost Activation

| Field | Value |
| --- | --- |
| Activation | Free Action (no combat/movement action cost) |
| PE cost | 2 PE/turn (combat), 6 PE/min (outside combat). Deducted on activation and at each subsequent turn start while Ghost remains active. |
| Duration | [v8.9] PE maintenance: 2 PE/turn (combat), 6 PE/min (outside). Auto-deactivates if 0 PE at turn end. **Important:** Ghost deactivates **immediately** upon attacking (see §Attack from Ghost below). The 1-turn PE cost is the maintenance billing period — it does NOT guarantee Ghost persists for a full turn if the Rogue attacks. A Rogue who activates Ghost and attacks on the same turn pays 2 PE and loses Ghost mid-turn after the first attack resolves. |
| Stealth roll | On activation: standard Combo Roll with Discrétion as Base. Player (or auto-gen) selects Combo Characteristic per standard rules (any Characteristic except Discrétion). OD auto-successes from both Base and Combo apply normally. +3 auto-successes added on top (Ghost-specific bonus, stacks with ODs). Store the total result as `stealthThreshold`. [v8.9] PEs penalty applies (SPEC-07 — all Combo Rolls are affected). A Rogue with low PEs will have a significantly weaker stealth threshold. |

#### Enemy Detection of Ghosted Knights
[v8] During enemy turn, **before target selection**, each enemy attempts detection against all Ghosted knights.

**Detection procedure (per enemy, per Ghosted knight):**
1. **Dice pool:** `floor(enemy.Machine / 2)` D6 dice. Alt-vision enemies: use `enemy.Machine + 3` instead of `enemy.Machine`.
2. **Roll:** Count evens = successes.
3. **Auto-successes:** Add `enemy.machineExcMineure` score as flat automatic successes (NOT extra dice).
4. **Compare:** Total detection successes vs target knight's `stealthThreshold`.
5. **Result:** `detectionSuccesses ≥ stealthThreshold` → target **revealed TO THIS ENEMY ONLY**. Other enemies must detect independently.

**Special cases:**
- **Machine Exceptionnelle Majeure:** Auto-detect ALL Ghosted knights without rolling. Ghost is completely ineffective against these enemies.
- **Per-enemy scope:** Each enemy resolves detection independently. A Ghosted knight can be visible to one enemy and invisible to another.
- **Re-stealth after attack:** Next turn, spend 2 PE to reactivate Ghost. New Discrétion combo roll → new `stealthThreshold`.

**Worked Example:**
```
Two Rogues in Ghost: R1 (stealthThreshold=6), R2 (stealthThreshold=3).

Enemy E1: Machine 12, Machine Exc. Mineure 3.
  Pool = floor(12/2) = 6D6.
  Roll: 6D6 → 2 successes.
  Add auto-successes: 2 + 3 = 5 total.
  vs R1: 5 < 6 → R1 remains hidden from E1.
  vs R2: 5 ≥ 3 → R2 detected by E1.

Enemy E2: Machine 6, no Exc. Mineure.
  Pool = floor(6/2) = 3D6.
  Roll: 3D6 → 1 success.
  vs R1: 1 < 6 → R1 hidden from E2.
  vs R2: 1 < 3 → R2 ALSO hidden from E2 (even though E1 detected R2).
```

#### Attack from Ghost

| Field | Value |
| --- | --- |
| Trigger | Any attack while Ghost active. |
| Ghost deactivation | Immediate after the first attack resolves (before any subsequent attack in the same Combat Action). |
| Bonus dice | +Discrétion score (with ODs) added to attack roll pool. |
| Bonus damage | +Discrétion score (with ODs) as flat damage bonus. |
| Weapon restriction | [v8] Ranged: requires **Silencieux** effect. Contact: requires **NO Lumière** effect. Violation breaks Ghost before damage (no stealth bonus). |
| Scope | [v8.9] **First attack only.** Ghost deactivates immediately after the first attack resolves. If the knight performs multiple attacks in the same Combat Action (e.g., Ambidextre), only the **first** attack receives Ghost bonuses (+Discrétion dice and +Discrétion damage). Ghost deactivates after the first attack resolves — the second attack proceeds without Ghost (no bonus dice, no bonus damage, knight is now visible). This is not a per-turn limit — it's a per-activation limit. Ghost must be reactivated (2 PE, next turn) to get the bonus again. |
| Re-stealth | Next turn: 2 PE to reactivate Ghost. New Discrétion combo roll (PEs penalty applies). |

#### Ghost Interactions
- **Débordement:** Bande Débordement hits Ghosted Rogues. Ghost does NOT protect against automatic damage.
- **Fold:** Ghost deactivates on Fold. Cannot reactivate until armor restored.
- **0 PE:** Ghost auto-deactivates at turn end if knight has 0 PE remaining.
- **Combat end:** [v8.9] Ghost is disabled at combat end. `isGhostActive` resets to `false` at each combat start (SPEC-32). Ghost is irrelevant between combat nodes — no stealth checks exist outside combat in demo.

Evolutions (locked): 150 PG: 6-turn duration. 200 PG: auto-success. 250 PG: 3 PE to stay invisible through attacks.

### Acceptance Criteria
Ghost activation: Free Action, 2 PE/turn maintenance. Invisible to enemies, can't be targeted.
Detection: E1 (Machine 12, Exc. Min. 3) gets 5 total → detects R2 (stealth 3) but not R1 (stealth 6). R2 remains hidden from E2 (1 success < 3).
Discrétion 5 + OD 2 attacks: +7 dice, +7 flat damage. Ghost drops.
[v8.9] Ghost + Ambidextre: Rogue (Discrétion 5, OD 2) in Ghost attacks with Ambidextre (2 Couteau de Combat). 1st attack: +7 bonus dice, +7 bonus flat damage. Ghost deactivates immediately after 1st attack resolves. 2nd attack: no Ghost bonus (0 bonus dice, 0 bonus damage). Rogue is now visible to enemies.
Lumière weapon: no Ghost bonus.
Débordement: Rogue in Ghost still takes damage.
0 PE at turn end: Ghost deactivates.


