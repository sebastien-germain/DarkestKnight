# Combat Rolls: Combo Roll, Damage Resolver & Heroism
> **SPECs:** 04, 05, 09
> **Domain:** `Scripts/Combat`
> **Dependencies:** [01_KnightDataModel.md](./01_KnightDataModel.md) (SPEC-01), [04_ArmorAndState.md](./04_ArmorAndState.md) (SPEC-06, SPEC-07), [07_WeaponsAndEffects.md](./07_WeaponsAndEffects.md) (SPEC-19, SPEC-20)
> **Depended on by:** [04_ArmorAndState.md](./04_ArmorAndState.md), [06_EnemySystem.md](./06_EnemySystem.md)
---
## SPEC-04: Combo Roll System
### Purpose
Resolves all attacks, abilities, and contested actions.
### Roll Procedure
Base Characteristic: Set by action type (Combat for melee, Tir for ranged, etc.).  
[v2] Choose Combo Characteristic: Player selects from any Characteristic EXCEPT the Base. Choice is per-roll — may change each action. Must differ from Base.  
Pool size: Base Characteristic score (dice) + Combo Characteristic score (dice) + style mods + PEs penalty + injury penalties.    
**[v8.9] Pool floor: Dice pool minimum is 0.** If the sum of all modifiers would reduce the pool below 0, set pool = 0. Modifier application order: (1) Base + Combo scores, (2) style modifier (+3/−3/−N/−2), (3) PEs penalty (−pesPenalty, see SPEC-07), (4) injury penalties (e.g., Yeux Détruits −3), (5) floor at 0. **Pool = 0 is an automatic failure** — the roll does not happen, no dice are thrown, and OD auto-successes are NOT applied. The knight is too debilitated to even attempt the action. OD represents technique built on an actual attempt; no attempt means no technique.
PEs penalty: See SPEC-07. Formula: `pesPenalty = max(0, 10 − currentPEs)`. Subtracted from dice pool (not from successes). [v8.9] Applies to **every** Combo Roll a knight performs — attacks, Assists, Ghost stealth, Stabilization, Peur tests. No exceptions.  
Roll D6s equal to pool size.
Count successes: Even results (2, 4, 6) = 1 success each.  
Add OD auto-successes: Base Characteristic's OD level + Combo Characteristic's OD level. These are flat successes added after the roll, not extra dice. [v8.9] OD hard cap: 5 from normal sources. Warrior Type +1 can push to 6 (SPEC-17.1). Use `effectiveOD` (base + Type bonus if active).  
Special outcomes:  
Compare: [v7] Successes > Defense (melee) or Reaction (ranged) = hit. Must strictly exceed, not equal.  

### Style Modifiers

| Style | Attack Pool | Defense/Reaction Mod | Restriction |
| --- | --- | --- | --- |
| Standard | — | — | None. Default style. |
| Agressif | +3 dice | −2 Defense, −2 Reaction | None. |
| Défensif | −3 dice | +2 Defense only | None. Melee defense focus. Reaction (ranged) unaffected. |
| Mise à couvert | −3 dice | +2 Reaction (ranged only) | **Prerequisite: rank must have `hasCover = true` (SPEC-03).** Cannot activate at a rank without cover — action is blocked, UI displays "No cover at this rank." Ranged Reaction only. Defense (melee) unaffected. [v8.9] If cover is removed at this rank (NanoC expiry, etc.), knight is forced to Standard — see SPEC-03 Cover Removal Procedure. |
| Puissant | −N dice (player chooses N, 1–6) | −2 Defense, −2 Reaction | Requires Contact weapon with **Lourd** effect. N sacrificed dice become N bonus damage/violence dice (see SPEC-05 step 6). |
| Pilonnage | −2 dice | — | Requires Ranged weapon with **Deux Mains** effect. Consecutive turns attacking the same target accumulate +1 damage/violence die each (see Pilonnage details below). |
| Précis | +3rd Characteristic | Costs Combat + Movement Action | Uses all actions. 3rd Characteristic's OD also adds auto-successes. |

#### Pilonnage — Sustained Fire Details
Pilonnage represents suppressive fire on a single target, growing deadlier with each consecutive turn.

**Accumulation rule:** Track `pilonnageTurns` counter per knight. Starts at 0 when Pilonnage style is activated or when the target changes.
- Each turn the knight attacks the **same target** in Pilonnage style: `pilonnageTurns += 1` (after the attack resolves).
- Bonus: `pilonnageTurns` bonus dice added to the damage AND violence roll (not the attack pool). [v8.9] **Clarification:** for each accumulated `pilonnageTurn`, add 1 bonus D6 to both the damage dice pool AND the violence dice pool for that attack. The bonus dice are rolled independently for each pool — they are not a single shared roll.
- The −2 dice attack pool penalty applies **every** turn regardless of accumulated bonus.

**Reset conditions** (set `pilonnageTurns = 0`):
- Knight switches to a different target.
- Knight switches to a different combat style.
- Knight is unable to attack on their turn (Choc, Parasitage, Incapacitated, etc.).
- Combat ends / new encounter starts.

**Worked Example:**
```
K1 attacks E1 with Fusil de Précision (Deux Mains, Ranged), Pilonnage style:
  Turn 1: attack E1 → −2 attack dice, +0 bonus damage/violence dice. After: pilonnageTurns = 1.
  Turn 2: attack E1 → −2 attack dice, +1 bonus damage/violence die. After: pilonnageTurns = 2.
  Turn 3: attack E1 → −2 attack dice, +2 bonus damage/violence dice. After: pilonnageTurns = 3.
  Turn 4: attack E2 → target changed! Reset. −2 attack dice, +0 bonus. After: pilonnageTurns = 1.
  Turn 5: attack E2 → −2 attack dice, +1 bonus damage/violence die. After: pilonnageTurns = 2.
  Turn 6: attack E1 → target changed! Reset. −2 attack dice, +0 bonus. After: pilonnageTurns = 1.
```


### Special Outcomes

| Outcome | Condition | Effect |
| --- | --- | --- |
| Exploit | All dice show even results (100% successes) | Re-roll pool, add new successes. +1 Héroïsme, +1 PEs. |
| Failure Critique | All dice show odd results (0 successes) | Attack auto-fails. Weapon may jam, fumble, or backfire (per encounter script). −1 PEs. |
| Assistance | Up to 3 allies Combo Roll | [v4] From any rank (no proximity required). Each spends 1 Movement Action. Successes added. Characteristic used must not duplicate Base, Combo, or another assistant's choice. PEs penalty applies to each assistant's roll individually (SPEC-07). [v8.9] **Solitaire** (Tarot IX disadvantage): if any assistant has Solitaire, the main roll's difficulty threshold is increased by +1 (Defense+1, Reaction+1, or narrative successes+1). Stacks per Solitaire assistant (2 Solitaire assistants = +2 threshold). See SPEC-24. |


### Edge Cases
Pool = 0 (floored): **Automatic failure.** No dice rolled, no Exploit, no Failure Critique, no OD auto-successes. The action simply fails — the knight cannot muster enough to attempt it. See Roll Procedure above for pool floor rule and modifier application order.
Précis: 3rd Characteristic’s OD also adds auto-successes.
Exploit on attack: re-roll successes count for threshold AND damage. [v8] Clarification: on Exploit, re-roll the entire dice pool. New successes from the re-roll are ADDED to the original successes (not replacing). The combined total is used for: (a) hit threshold comparison, and (b) excess successes beyond threshold become bonus damage dice (same as Mode Héroïque excess). OD auto-successes are added once (not re-applied on re-roll). **No Exploit chaining:** if the re-roll is also all-even, do NOT re-roll again — only one re-roll per Combo Roll. The second all-even result still grants +1 Héroïsme and +1 PEs (these bonuses are awarded per Exploit trigger, so the knight earns 2 total: +2 Héroïsme, +2 PEs from the original roll and the re-roll).
[v8] Exploit/Failure Critique is determined solely by dice faces, not OD auto-successes. OD is added AFTER checking for special outcomes.
[v8] Exploit that still misses after re-roll + OD: no damage dealt, but Héroïsme and PEs bonuses are still awarded.

[v8] Pool = 1 die: yes, 50% of single-die rolls trigger special outcomes. This is intended — small pools are inherently volatile.

### [v4] Dual-Wielding Rules (L3 Resolved)
Dual-wielding uses one of two combat styles: **Ambidextre** or **Akimbo**. Both require one-handed weapons only (no Deux Mains effect). [v8.9] The distinction is **attack resolution style**, not weapon identity: Ambidextre = 2 separate attacks; Akimbo = 1 combined attack. **Two identical weapons can use either Ambidextre or Akimbo** at the knight's choice. Source: Knight Livre de Base, "Style ambidextre" and "Style akimbo."

**Style Ambidextre:**
- Two one-handed weapons (same or different). Main hand = directrice.
- Two **separate attacks** on the same or different targets, counting as **1 Combat Action**.
- Each attack is rolled independently (separate Combo Roll per weapon).
- **−3 dice** penalty on all attack rolls.
- Weapon with **Jumelé (Ambidextrie)** effect: reduces the penalty by 2 → **−1 die** penalty. See SPEC-20.

**Style Akimbo:**
- Two **identical** one-handed weapons (may have different ammo/modules, but same base weapon).
- **One single attack** combining both weapons' damage.
- **Primary weapon = main hand (directrice).** Used for all effects AND flat bonuses.
- Damage dice: sum of both weapons' damage dice. Flat bonus from primary weapon only (not doubled).
- Violence dice: primary weapon's full violence dice + half (floor) of secondary weapon's violence dice. Flat bonus from primary weapon only (not doubled).
- Effects: only the **primary** (main hand) weapon's effects apply. Secondary weapon contributes dice only.
- **−3 dice** penalty on all attack rolls.
- Weapon with **Jumelé (Akimbo)** effect: reduces the penalty by 2 → **−1 die** penalty. See SPEC-20.

Both weapons resolve against the same target (Akimbo) or same/different targets (Ambidextre). Damage rolled separately for each weapon (Ambidextre) or combined (Akimbo).
[v8.9] **Ambidextre Suppress:** If both weapons have Barrage X, the character can choose Suppress instead of Attack. This fires 2 separate Suppress actions (one per weapon) for 1 Combat Action. Each weapon is a separate source — both can target the same enemy and their Barrage values stack. No −3 dice penalty (Suppress has no attack roll). Example: main hand Barrage 2 + off-hand Barrage 4 → both Suppress E1 → E1 gets −6 Defense/Reaction.
Incompatible with Paladin Watchtower (which grants extra ranged shots, not dual-wield).

### [v4] Chargeur & Ammo Rules (L4 Resolved)
Weapons with Chargeur X: X uses per mission. Each attack consumes 1 charge.
No mid-mission reload. When charges reach 0, weapon cannot fire until Camelot return.
Auto-refill at Camelot between missions. No PG cost.
Grenades: Fixed 5 per mission per knight. Same auto-refill rule. Choose type before each throw.
Nods: Quantity set per mission (default 3 of each type per knight). Auto-refill at Camelot. See [SPEC-33](./13_NodSystem.md) for full Nod system.
### Acceptance Criteria
Combat 4 + Perception 3, Standard, PEs 50: rolls 7D6.  
Same with PEs 7 (penalty 3): rolls 4D6.  
[v8.9] Combat 3 + Instinct 2, Défensif (−3), PEs 1 (penalty 9): pool = 3+2−3−9 = −7 → floored to 0. Automatic failure. OD auto-successes (Combat OD 2) are NOT applied — no attempt possible. No Exploit, no Failure Critique.  
All even: Exploit. Zero even: Failure Critique.  
[v8.9] K1 attacks E1 (Defense 6). K2 (Solitaire) assists with Instinct, rolls 2 successes. K1 rolls 5 successes + 2 assist = 7 total. But threshold is 6+1 = 7 (Solitaire raises it). 7 > 7 is false (must strictly exceed) → miss. Without Solitaire, 7 > 6 → hit.  
[v8.9] K1 attacks E1 (Reaction 4). K2 (Solitaire) and K3 (normal) both assist. Threshold = 4+1 = 5 (only K2 has Solitaire, +1 once). K1 rolls 4, K2 adds 1, K3 adds 2 = 7 total. 7 > 5 → hit.    
[v8.9] Weapon switch: K1 has Marteau-Épieu equipped, switches to Fusil de Précision: costs 1 Movement Action. K1 can still attack this turn (1 Combat Action remaining).  
[v8.9] Profile switch (Free): K1 has Marteau-Épieu on Contact profile. Wants to use Tir (charge): switch is Free (weapon adapts). K1 attacks with Tir profile using Combat Action. Both Movement and Combat Actions available this turn.  
[v8.9] Profile switch (Movement): K1 has Lance-Grenade on Explosive. Wants Incendiaire: switch costs 1 Movement Action (manual magazine swap). K1 can still attack with Combat Action.  
[v8.9] Grenade type: K1 throws Flashbang grenade. Next throw wants Shrapnel: type chosen on throw, no switch cost. Both use 1 Combat Action each (if knight has 2 combat actions from Mode Héroïque or similar).  

### [v4] Action Economy Reference (M2, M5, M6 Resolved)
Consolidated action cost table for common combat activities. [v4] Updated for DD rank system.

[v8.9] **Standard turn:** 1 Combat Action + 1 Movement Action.  
- **Combat Action** can be used for any action requiring a Combat Action OR a Movement Action (it is the more flexible action type). This substitution is free and implicit — no explicit "forfeit" declaration is needed. A knight may spend their Combat Action on a Movement-cost action at any point during their turn.  
- **Movement Action** can ONLY be used for actions requiring a Movement Action.  
- "Forfeit Combat Action for 2nd Movement Action" (referenced in SPEC-03/SPEC-15) is simply the natural consequence of spending the Combat Action on a Movement-cost action. It is NOT a separate mechanic requiring a start-of-turn declaration.  
- This applies to both knights and enemies. Example: a knight can spend their Combat Action to move instead of attacking, giving them 2 movement-type actions that turn (but no attack).  

| Action | Cost | Notes |
| --- | --- | --- |
| Attack (melee or ranged) | 1 Combat Action | Combo Roll required. Rank targeting applies (see SPEC-03). |
| [v8.9] Suppress (Barrage) | 1 Combat Action | Alternative to Attack. Requires equipped weapon with Barrage X effect. No roll, no damage. Reduces target Defense & Reaction by X until start of suppressor's next turn. See SPEC-20 Barrage X. Ambidextre: 2 Suppress actions for 1 Combat Action (one per weapon). |
| [v4] Move into adjacent empty rank | 1 Movement Action | Shift 1 rank forward/back into empty slot. |
| [v4] Swap with adjacent ally | 1 Movement Action | Both knights exchange ranks. No cost for swapped ally. |
| [v4] Move 2 ranks | Spend Combat Action on movement | Chain two single-rank moves/swaps. Combat Action substitutes for Movement Action (no forfeit declaration needed). |
| Use a Nod (self) | 1 Movement Action | Restores 3D6 of PS/PA/PE per type. See [SPEC-33](./13_NodSystem.md). |
| [v4] Use a Nod (on ally) | 1 Movement Action | Courte range (Gap ≤ 2). Same 3D6 restore. See [SPEC-33](./13_NodSystem.md) for Agony targeting exception. |
| Assist an ally | 1 Movement Action | From any rank. Roll 1 unique Characteristic. Max 3 assistants. |
| Switch equipped weapon | 1 Movement Action | [v8.9] Holster current, draw another (physically swapping what's in hand). |
| Switch weapon profile | Per weapon (see below) | [v8.9] Cost depends on whether the switch requires manual manipulation. See §Profile Switch Cost Rules. |
| Throw grenade | 1 Combat Action | Base Tir. Courte range (+Force OD extends). 5/mission. |
| Activate Warrior Type | 1 Movement Action | 1 PE. Max 1 switch/turn. |
| Activate Paladin Shrine | Free Action | [v8.9] 2 PE to place. +1 PE/turn maintenance. See SPEC-17.2. |
| Activate Watchtower | 1 Movement Action | 2 PE. |
| Activate Rogue Ghost | Free Action | 2 PE/turn. |
| Use NanoC (Simple) | 1 Movement Action | 3 PE. |
| Use NanoC (Detailed) | 1 Combat Action | 6 PE. |
| Use NanoC (Mechanical) | Full Turn (both actions) | 9 PE. |
| Use Priest Mechanic | 1 Movement Action | 4 PE. Contact or Long range. |
| Delay initiative | Free (start of turn) | Announce lower initiative value. |
| Deactivate Watchtower | Free Action | Restores mobility. |
| Switch combat style | Free Action | Once per turn. |

#### [v8.9] Profile Switch Cost Rules
The cost to switch between profiles on the same weapon depends on whether the switch requires **manual physical manipulation** by the character. This is a per-weapon property stored as `profileSwitchCost` on the WeaponProfile (see SPEC-19).

**Free (automatic adaptation)** — The weapon adapts to the chosen action without manual intervention. The character simply uses the weapon differently (swing the hammer vs pull the trigger, stab with the knife vs fire the pistol). No action cost.

**1 Movement Action (manual change)** — The character must physically manipulate the weapon: swap a magazine/ammunition type, change grip configuration (1H ↔ 2H), reload a different bolt. This requires hands-on action.

**Demo weapons — profile switch costs:**

| Weapon | Profiles | Switch Cost | Reason |
|--------|----------|-------------|--------|
| Marteau-Épieu | Contact / Tir (charge) | Free | Swing vs trigger — weapon adapts. |
| Pistolet de Service | Contact (knife) / Tir (pistol) | Free | Polymorphic weapon — morphs to action type. |
| Lance-Grenade Léger | Explosive / Antiblindage / Incendiaire | 1 Movement Action | Manual ammo swap (remove magazine, insert new type). |
| Grenades Intelligentes | Shrapnel / Flashbang / Anti-Blindage / IEM / Explosive | Free | Type chosen on throw — no pre-switch needed. |

**Future weapons (scaffolded, not in demo):**

| Weapon | Profiles | Switch Cost | Reason |
|--------|----------|-------------|--------|
| Épée Bâtarde | 1-handed / 2-handed | 1 Movement Action | Grip change — physically alters what's in hand. |
| Arbalète Magnétique | Multiple bolt types | 1 Movement Action | Manual bolt/munition swap. |

**Design pattern for new weapons:** If the character must physically do something with their hands to switch profiles → 1 Movement Action. If the weapon automatically adapts to the action type → Free.

## SPEC-05: Damage Resolver
### Purpose
Computes final damage from a hit and routes through armor layers. [v2] Violence pathway to PEs removed. [v3] Canonical damage formula established (C1/C2 resolved).

### [v3] CRITICAL DISTINCTION: Attack Roll vs Damage Roll
These are two completely separate systems. The MDD previously conflated them. The tabletop is unambiguous:


| | Attack Roll (To-Hit) | Damage Roll (On Hit) |
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
Puissant style bonus: Knight sacrifices N dice (1-6) from attack pool → gains N dice added to damage/violence roll. −2 Defense, −2 Reaction. Restricted to contact weapons with Lourd property. Roll sacrificed dice, SUM faces, add to damage total. Demo weapon availability: Puissant requires a Lourd Contact weapon — no demo-scope weapon has the Lourd property; Puissant is scaffolded for future weapons. Pilonnage requires a Deux Mains Ranged weapon — available with Fusil de Précision, Fusil d'Assaut, and Shotgun Escamotable.
Mise à couvert style: Knight takes cover behind terrain features. −3 attack dice, +2 Reaction against ranged attacks only. Defense (melee) unaffected. Interactions: +2 Reaction bonus applies to Dispersion splash if the incoming attack is ranged. Does NOT apply to Débordement (automatic damage, no attack roll). Stacks with SPEC-03 Cover System bonuses (+3 Reaction from cover rank + +2 from Mise à couvert = +5 total Reaction vs ranged). Does not stack with Défensif (+2 Def) — knight must choose one style per turn.
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
| Lesté | Force×2 instead of ×1 | |
| Tir (Précision) | +Tir score + 1/OD | Ranged with Précision effect |
| Dextérité (Orfèvrerie) | +Dext score + 1/OD | Weapons with Orfèvrerie effect |
| Discrétion (Ghost/Silencieux) | +Discrétion + OD | Stealth attacks (see SPEC-17) |

### Acceptance Criteria
[v7] Marteau-Épieu (3D6), rolls [3,5,4], Force 5 OD 1 (NOT Lesté): 12 + 5 + 3 = 20 raw. (If weapon were Lesté: 12 + 10 + 3 = 25 raw.)
[v3] Fusil Précision (4D6+6), rolls [3,6,2,5], Tir 4 OD 2: 16 + 6 + 4 + 2 = 28 raw.
[v3] 6 successes vs Defense 4 with no Assistance: 0 extra damage from excess.

[v3] Dégâts Maximum on 3D6: dice = [6,6,6] = 18. Then add flat bonuses.
[v2] 4D6 Violence vs knight: ignored. Same vs Bande: SUM = Cohesion lost.


---

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
**SCAFFOLDED — objective system deferred.** The full Mode Héroïque objective declaration and end-condition system requires dedicated video game mechanic design (the tabletop relies on GM adjudication). Demo implementation: Mode Héroïque activates and persists until the hero reaches 0 PS or 0 PEs. No objective selection, no objective-complete check. The +5 PG bonus is awarded if the hero survives the encounter. Future redesign: implement an objective selection menu (kill target, survive N turns, clear encounter) with system-verified completion.
Declare objective: **DEFERRED** (see above). Spend all 6 Héroïsme.
[v8.9] **Ally auto-contribution:** During Mode Héroïque, all other alive and controllable allies (not in Agony, not permanently Despaired) automatically contribute as Assistants on every Combo Roll the hero makes. No action cost and no PE cost for these auto-assists. Each ally rolls a standard Combo Roll using their highest-scoring Characteristic not already used by the hero (Base or Combo) or another auto-assistant. PEs penalty applies to each ally's auto-assist roll individually (SPEC-07). Max 3 assistants rule still applies — if more than 3 allies are alive, the 3 with the highest relevant Characteristic scores contribute. Egoiste (SPEC-24.3): knights with this disadvantage cannot serve as auto-Adjuvants during Mode Héroïque.
PE cost = 0. Excess successes = extra damage dice.
Ends on: hero at 0 PS/PEs OR encounter ends (whichever comes first).
+5 PG on survival to encounter end.
Le Fou: triggers at 4 instead of 6.
Le Soleil (Egoiste disadvantage, SPEC-24.3): cannot be Adjuvant. Additionally cannot use Assist action on allies.

