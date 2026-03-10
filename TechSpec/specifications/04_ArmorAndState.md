# Armour Layers, Espoir/Despair, Injury & Fold State
> **SPECs:** 06, 07, 08, 10
> **Domain:** `Scripts/Combat`
> **Dependencies:** [01_KnightDataModel.md](./01_KnightDataModel.md) (SPEC-01), [03_CombatRolls.md](./03_CombatRolls.md) (SPEC-04, SPEC-05), [06_EnemySystem.md](./06_EnemySystem.md) (SPEC-18), [05_ArmorAbilities.md](./05_ArmorAbilities.md) (SPEC-17)
> **Depended on by:** [03_CombatRolls.md](./03_CombatRolls.md) (SPEC-09 references), [08_ContentData.md](./08_ContentData.md) (SPEC-23)
---
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
 if target.PEs == 0: triggerDespair(target) // SPEC-07
[v8] PEs Recovery on Anathème Kill:
Per tabletop: when an enemy with the Anathème capacity is killed, each knight who lost PEs to that enemy’s Anathème attacks recovers 1D6 PEs per complete 6 PEs lost. Track cumulative Anathème PEs damage per knight per source enemy. On enemy death: recoveredPEs = floor(totalAnathemeLost / 6) × roll(1D6). Fire OnPEsChanged. This can exit Despair if PEs rises above 0.

### Tag Overrides

| Tag | Effect |
| --- | --- |
| Ignore CdF | Skip Step 1. |
| Pénétrant X | Reduce CdF by X for this hit. |
| Perce Armure X | Caps PA absorption: X points of PA are "pierced" and cannot absorb. effectivePA = max(PA − X, 0). Only effectivePA absorbs damage. target.PA decreases by the absorbed amount only (not by X). See pseudocode Step 6 and worked examples. |
| Destructeur | [v6] [v7] If remaining > 0 after CdF+Bouclier AND target has PA: roll +2D6, add to remaining BEFORE PA absorption. See canonical pseudocode below. |
| Meurtrier | [v6] [v7] If damage reaches PS (remaining > 0 after PA fully depleted): +2D6 bonus damage added to remaining before PS reduction. See canonical pseudocode below. |
| Dégâts Continus X | Bypass CdF+Bouclier. X dmg/turn for 1D6 turns. [v8] Tick timing: damage applies at the start of the afflicted combatant’s turn, before they act. DoT damage bypasses CdF and Bouclier but IS absorbed by PA normally. Applies to both knights and enemies. |
| Chair Mineur | [v8.9] Chair Exceptionnelle Mineur: flat damage reduction = Chair Exceptionnel score. Subtracted from remaining damage after Bouclier, before PA. Applies to ALL incoming hits (including DoT). See SPEC-18.3. |


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
      // [v8.9] Chair Majeur does NOT block Destructeur
      destructeurBonus = roll(2, D6)  // Roll 2D6
      remaining += destructeurBonus
  Step 6: if target.PA > 0 AND NOT weapon.hasTag(IGNORE_ARMURE):
    // [v8] Perce Armure: caps how much PA can absorb, NOT a flat PA reduction.
    // X points of PA are "pierced" — they cannot absorb damage. The remaining PA absorbs normally.
    // target.PA only decreases by the amount actually absorbed (paAbsorbed), never by X itself.
    if weapon.hasTag(PERCE_ARMURE_X):
      effectivePA = max(target.PA - X, 0)  // PA available for absorption after piercing
      paAbsorbed = min(effectivePA, remaining)
    else:
      paAbsorbed = min(target.PA, remaining)
    target.PA -= paAbsorbed  // PA decreases ONLY by absorbed amount, not by X
    remaining -= paAbsorbed
    passthroughPS = floor(paAbsorbed / 5)  // unless Infatigable
    target.PS -= passthroughPS
    if target.PA == 0: triggerFold(target)  // SPEC-10
  Step 7 — MEURTRIER CHECK:
    // [v8.9] Chair Majeur: immune to Meurtrier — skip this step entirely
    if remaining > 0 AND weapon.hasTag(MEURTRIER) AND NOT target.hasChairMajeur:
      meurtrierBonus = roll(2, D6)  // Roll 2D6
      remaining += meurtrierBonus
  Step 8: target.PS -= remaining
  if target.PS <= 0: triggerAgony(target)  // SPEC-08
```
[v7] Worked Example — Both Destructeur + Meurtrier on same hit:
Weapon: Mitrailleuse lourde (5D6+10, Destructeur, Perce Armure 20). Target: Chevalier renégat (CdF 10, PA 25, PS 40).
Step 1: rawDamage = 26 (rolled 5D6=16, +10 flat). remaining = 26.  
Step 2: CdF. No IGNORE_CDF, no PENETRANT. effectiveCdF = 10. remaining = 26 − 10 = 16.  
Step 3: No Bouclier. remaining = 16.  
Step 4: No Chair Mineur. remaining = 16.  
Step 5: DESTRUCTEUR — remaining > 0 (16) AND weapon has Destructeur AND target.PA > 0 (25): YES. Roll 2D6 = [4,3] = 7. remaining = 16 + 7 = 23.  
Step 6: PA absorption. Perce Armure 20: effectivePA = max(25 − 20, 0) = 5. paAbsorbed = min(5, 23) = 5. target.PA = 25 − 5 = 20. remaining = 23 − 5 = 18. Passthrough: floor(5/5) = 1 PS. target.PS = 40 − 1 = 39. PA > 0, no Fold.  
Step 7: MEURTRIER — remaining > 0 (18) AND weapon has Meurtrier: YES. Roll 2D6 = [5,6] = 11. remaining = 18 + 11 = 29.  
Step 8: target.PS = 39 − 29 = 10. PS > 0, no Agony. **Final: CdF 10 (unchanged), PA 20, PS 10.** Both Destructeur (+7) and Meurtrier (+11) triggered — devastating combo.  

[v8.9] **Alternate outcome — no Meurtrier:** Same scenario but weapon only has Destructeur (no Meurtrier). After Step 6: remaining = 18. Step 7: skipped (no Meurtrier). Step 8: PS = 39 − 18 = 21. **Final: PA 20, PS 21.** Without Meurtrier, the PS hit is 11 points less severe.  

[v8.9] **Perce Armure Worked Example — PA tracking:**
**Case A — Perce Armure X ≥ target PA (PA fully pierced):**
Weapon: Marteau-Épieu (3D6, Perce Armure 40). Target: Bestian (CdF 0, PA 30, PS 20). rawDamage = 15.
Step 1: remaining = 15. Steps 2–5: no CdF, Bouclier, Chair, Destructeur. remaining = 15.
Step 6: Perce Armure 40. effectivePA = max(30 − 40, 0) = **0**. paAbsorbed = min(0, 15) = **0**. target.PA = 30 − 0 = **30 (unchanged)**. remaining = 15. No passthrough (0 PA absorbed). PA > 0, no Fold.
Step 8: target.PS = 20 − 15 = **5**. **Final: PA 30 (untouched!), PS 5.** Perce Armure pierced all 30 PA points, so PA could absorb nothing. All damage went straight to PS. But PA itself was NOT reduced — no damage was absorbed by it.

**Case B — Perce Armure X < target PA (PA partially pierced):**
Same weapon (Perce Armure 40). Target: Knight (CdF 0, PA 60, PS 40). rawDamage = 25.
Step 6: effectivePA = max(60 − 40, 0) = **20**. paAbsorbed = min(20, 25) = **20**. target.PA = 60 − 20 = **40**. remaining = 25 − 20 = 5. Passthrough: floor(20/5) = 4 PS → PS = 40 − 4 = 36. PA > 0, no Fold.
Step 8: target.PS = 36 − 5 = **31**. **Final: PA 40, PS 31.** PA absorbed 20 points (the unpierced portion). The 40 "pierced" PA points were bypassed, not destroyed.
### [v2] PA Passthrough Worked Example

### Acceptance Criteria
CdF 8, PA 100, PS 40. 15 raw: CdF absorbs 8, PA takes 7. Passthrough=1. PA=93, PS=39.
Same with Infatigable: PA=93, PS=40.



---

## SPEC-07: Espoir (PEs) System
### Purpose
Tracks morale. PEs < 10 = dice penalty. PEs = 0 = Despair. Canonical penalty formula: pesPenalty = max(0, 10 − currentPEs). Applied as negative modifier to **all** Combo Roll dice pools (SPEC-04) — this includes attack rolls, Ghost activation stealth rolls (SPEC-17.4), Stabilization rolls (SPEC-07 below), Assist rolls (SPEC-04), Peur tests (SPEC-18.10), and any other Combo Roll a knight performs. No exceptions. Dice pool floors at 0 — pool = 0 is automatic failure (no dice, no OD, see SPEC-04 Roll Procedure). [v2] No Violence input.
### State Table

| PEs Range | State | Dice Penalty | Additional |
| --- | --- | --- | --- |
| 10–50+ | Résolu | 0 | Full effectiveness. |
| 7–9 | Ébranlé | 1–3 | — |
| 4–6 | Brisé | 4–6 | Armor portrait cracking. |
| 1–3 | Au bord du gouffre | 7–9 | Failure Critique threshold +1. |
| 0 | Désespoir | N/A | Acts against team 1D6 turns. Parole+Aura diff 3 to stabilize. |


### [v2] Loss Sources (Canonical List)

| # | Source | Amount | Trigger | Defined In |
|---|--------|--------|---------|------------|
| 1 | Anathème attack vs knight | CdF → remainder hits PEs directly (PA bypassed) | On hit by Anathème-typed weapon | SPEC-06 (this file, §Knight Defense vs Anathème) |
| 2 | Narrative event choices | Variable per event (designer-authored) | Mission node entry or player choice | SPEC-22 ([10_MissionAndHub.md](./10_MissionAndHub.md)) |
| 3 | Cauchemars disadvantage (Tarot XVII) | −1 PEs | On rest/heal: roll 1D6, odd = nightmare | SPEC-24 ([09_TarotSystem.md](./09_TarotSystem.md)) |
| 4 | Cybernetic Implant (maxPEs reduction) | maxPEs permanently −3 per implant (reversible via Reconstruction Therapy, see Recovery #8) | At Camelot Infirmary, on implant purchase | SPEC-25 ([10_MissionAndHub.md](./10_MissionAndHub.md)) |

[v2] REMOVED: 'Enemy Violence = PEs loss.' Violence does not affect individual PEs.

**Modifier:** Esprit d'Acier (Tarot IX, L'Ermite): All PEs losses from any source reduced by 1 (minimum 0 loss). See SPEC-24 ([09_TarotSystem.md](./09_TarotSystem.md)).

### Recovery Sources (Canonical List)

| # | Source | Amount | Trigger | Defined In |
|---|--------|--------|---------|------------|
| 1 | Minor Motivation fulfilled | +1D6 PEs (repeatable) | Event tag match via SPEC-12 | SPEC-12 ([08_ContentData.md](./08_ContentData.md)) |
| 2 | Major Motivation completed | +25 currentPEs AND +25 maxPEs (one-time, replaced at Camelot) | Event tag match via SPEC-12 | SPEC-12 ([08_ContentData.md](./08_ContentData.md)) |
| 3 | Exploit | +1 PEs | All dice show even on any Combo Roll | SPEC-04 ([03_CombatRolls.md](./03_CombatRolls.md)) |
| 4 | Anathème Kill recovery | floor(totalAnathemeLost / 6) × 1D6 PEs per knight | Anathème enemy killed | SPEC-06 (this file, §PEs Recovery on Anathème Kill) |
| 5 | Narrative events | Variable (e.g., +1D6, +3 PEs) | Mission node entry or player choice | SPEC-22 ([10_MissionAndHub.md](./10_MissionAndHub.md)) |
| 6 | Stabilization from Despair | PEs resets to 1 | Ally succeeds stabilization roll | SPEC-07 (this file, §Despair Resolution) |
| 7 | Forteresse Spirituelle (Tarot V) | +5 maxPEs at creation | Knight generation | SPEC-24 ([09_TarotSystem.md](./09_TarotSystem.md)) |
| 8 | [v8.9] Reconstruction Therapy | maxPEs restored to pre-implant value (implantCount reset to 0). currentPEs unchanged. | Player purchases therapy at Camelot Infirmary | SPEC-25 ([10_MissionAndHub.md](./10_MissionAndHub.md)) |

**No between-mission PEs recovery in demo.** PEs carries over between missions unchanged. Future: psychologist room at Camelot.
### Acceptance Criteria
PEs 9: penalty=1. PEs 3: penalty=7.



### [v8] Despair Resolution (CR-3 — New Subsection)
When a knight’s PEs reaches 0, the Despair (Désespoir) state activates. This subsection provides the full mechanical specification for agent implementation.
[v8] Despair Trigger:
PEs reaches 0 from any source. Immediate — no action, no roll.
[v8.9] **Duration roll:** Immediately on Despair entry, roll 1D6 to determine `despairDuration` (number of turns the knight acts under hostile AI). This roll happens once — the result is stored and decremented each turn. It is NOT re-rolled per turn.

Fire OnDespairEntered(knight, despairDuration) event (SPEC-27).
Ignorer Désespoir (1 Héroïsme, SPEC-09): cancels this Despair trigger entirely. The PEs loss that would have triggered Despair is also prevented — PEs stays at 1 (not 0). Knight acts normally.
[v8] Despair Behavior (Hostile AI):
During Despair, the knight is player-uncontrollable and acts under hostile AI for despairDuration turns:
[v8.9] **Combat style: forced Agressif.** The Despaired knight loses control and fights recklessly against their former allies (+3 dice attack, −2 Defense, −2 Reaction). The knight cannot switch style while Despaired — Agressif is locked. Exception: if Fold is also active, Fold's forced Standard overrides Despair's forced Agressif (see Edge Cases below).
Target selection: Random allied knight within weapon range. If no ally in range, move toward nearest ally (1 Movement Action), then attack if now in range.
Attack resolution: Standard Combo Roll using the knight's currently equipped weapon. System auto-selects highest available Base + Combo pair. Roll vs target ally's Defense (contact) or Reaction (ranged). Damage resolved normally through ally's ArmourLayerResolver (SPEC-06).
Action economy:
Turn countdown: Decrement despairDuration at end of the Despaired knight’s turn. Display remaining turns on HUD.
[v8] Stabilization (Ally Intervention):
Who:
Cost: 1 Combat Action from the stabilizing knight.
Roll: Combo Roll with Parole as Base + Aura as Combo (or any other valid Combo, but Parole must be Base). Difficulty: Ardue (3 successes required). OD auto-successes apply normally. [v8.9] PEs penalty applies to this roll (SPEC-07 — all Combo Rolls are affected).
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
[v8.9] Despair forces Agressif style: Despaired knight's style is locked to Agressif (+3 dice, −2 Def, −2 React). Cannot switch. Allies attacking the Despaired knight benefit from the −2 Def/React penalty.
[v8.9] Despair + Fold: Knight is both Despaired and Folded. Fold forces Standard (overrides Agressif). Knight attacks with Standard style (no +3 dice bonus, no −2 Def/React penalty).



---

## SPEC-08: Injury & Agony System
### Purpose
Injury Table, Agony state, Hémorragie countdown, permanent injuries.
### Agony Trigger Sequence
PS reaches 0.
Immediately: roll Injury Table (SPEC-23).
Apply injury to KnightBase.
Set isInAgony = true.
Disable all action slots.

### Agony State Rules
No actions possible. The knight is incapacitated — cannot attack, move, use Nods on self, activate abilities, or take any voluntary action.
An ally can use Nod de Soin on the Agony knight to restore PS >= 1 and exit Agony (see [SPEC-33](./13_NodSystem.md) §33.3).

Ignorer l'Agonie (1 Héroïsme): stay at 1 PS, no Agony, **no Injury roll**. Once/combat. This completely prevents the Agony trigger sequence — if no injury is rolled, Hémorragie cannot occur. However, Ignorer l'Agonie **cannot** be used after an injury is already applied (e.g., it cannot cancel an active Hémorragie countdown or prevent death when the countdown reaches 0).

### Special Injuries
[v8.9] Notation: `[severity die, row die]` — see SPEC-23 §23.1 for the complete roll procedure and severity-to-column mapping.
[1,1] = Column 1 (Catastrophic), Row 1: MORT: Instant permadeath. Remove from roster.
[1,5] = Column 1 (Catastrophic), Row 5: Hémorragie Interne: See §Hémorragie System below for full specification.
[4,1] = Column 3 (Moderate, severity die 4), Row 1: Colonne Vertébrale Brisée: [v8] Movement costs 2 actions (any type). Cannot Sprint. See SPEC-23 §23.3.
[6,2] = Column 4 (Light, severity die 6), Row 2: Jambe Mutilée: Movement costs doubled. Cannot use movement-dependent abilities.

### [v8.9] Hémorragie System (Complete Specification)

Hémorragie (internal hemorrhage) is the only injury with an active countdown mechanic. It is triggered by injury roll `[1,5]` (Catastrophic Column, Row 5). Unlike all other injuries, Hémorragie is **not permanent** — it is removed when the knight exits Agony.

#### Trigger
When the Injury Table result is Hémorragie Interne (`[1,5]`):
1. Add Hémorragie to the knight's `activeInjuries` list with `isPermanent = false`.
2. Set `hémorragieCountdown = 3` (3 turns remaining).
3. **Countdown starts immediately** — the knight has 3 of their own turns to be saved.
4. Fire `OnHémorragieStart(knight, 3)` event (SPEC-27).
5. The knight is in **Agony** (PS = 0, no actions). The Agony state and Hémorragie are simultaneous and co-dependent.

#### Countdown Resolution
The countdown ticks at the **start of the bleeding knight's turn** in the initiative queue:

```
function onTurnStart(knight):
  if knight.hasHémorragie:
    knight.hémorragieCountdown -= 1
    fire OnHémorragieTick(knight, knight.hémorragieCountdown)

    if knight.hémorragieCountdown <= 0:
      // Countdown expired — permadeath
      knight.isDead = true
      removeFromRoster(knight)
      fire OnKnightDeath(knight, cause: HEMORRAGIE)
      return  // Turn ends immediately, knight is dead

    // Knight is still alive but in Agony — skip their turn (no actions)
```

**Turn-by-turn timeline:**

| Phase | hémorragieCountdown | What Happens |
|-------|---------------------|-------------|
| Injury occurs (ally/enemy turn) | 3 | Hémorragie applied. Knight enters Agony. |
| Knight's next turn starts | 2 (3 to 2) | Tick. Still alive. Skip turn (Agony). Allies have until next tick to heal. |
| Knight's next turn starts | 1 (2 to 1) | Tick. Still alive. **Last chance** — allies must act before the next tick. |
| Knight's next turn starts | 0 (1 to 0) | Tick. Countdown expired. **Instant permadeath.** |

**Saving the knight (before countdown reaches 0):**
An ally must restore the bleeding knight's PS to >= 1. Valid methods:
- **Nod de Soin** (ally uses on bleeding knight, Courte range) — see [SPEC-33](./13_NodSystem.md) §33.7.
- **Priest Mechanic** (SPEC-17.3) — restores PA, not PS. Does NOT save from Hémorragie directly.
- Any other effect that restores PS >= 1 (future content).

When PS is restored to >= 1:
1. Knight **exits Agony** (isInAgony = false, actions re-enabled).
2. **Hémorragie is removed** from `activeInjuries`. The countdown stops. Unlike other injuries, Hémorragie does NOT persist — it is a temporary condition tied to the Agony state.
3. Fire `OnAgonyExited(knight)` and `OnHémorragieSaved(knight)` events.
4. The knight can act normally on their next turn.

#### Key Rules & Interactions

| Rule | Details |
|------|---------|
| **Hémorragie is NOT permanent** | Exception to the general injury rule. All other injuries persist until treated at Camelot Infirmary (SPEC-25). Hémorragie is removed when the knight exits Agony (PS restored to >= 1). |
| **Knight remains in Agony** | Throughout the entire Hémorragie countdown, the knight is in Agony (PS = 0, no actions). They cannot help themselves. |
| **Ignorer l'Agonie prevents Hémorragie** | If the knight spends 1 Héroïsme when PS reaches 0, the entire Agony trigger sequence is skipped — no injury roll occurs, so Hémorragie cannot be rolled. PS stays at 1. |
| **Ignorer l'Agonie CANNOT cancel active Hémorragie** | Once the injury is applied and the countdown is running, Ignorer l'Agonie cannot be used. It only prevents the initial Agony trigger, not a countdown already in progress. There is no "second chance" Heroism spend to cheat death at countdown 0. |
| **Countdown = 0 means instant permadeath** | When the countdown reaches 0, the knight dies immediately. No additional injury roll, no Agony re-check. `isDead = true`, remove from roster. Same finality as rolling `[1,1] MORT`. |
| **Trompe la Mort (Tarot XIII)** | Trompe la Mort triggers on rolling MORT (`[1,1]`), not on Hémorragie (`[1,5]`). It does NOT prevent Hémorragie death. It does NOT trigger when the Hémorragie countdown expires. |
| **VOEU_OURS (L'Ours Blason)** | If an ally heals the bleeding knight out of Agony via Nod de Soin, the VOEU_OURS trigger fires for the healing knight (see SPEC-12, [SPEC-33](./13_NodSystem.md) §33.8). |
| **Between-node transition** | If combat ends while a knight has active Hémorragie (countdown > 0), the knight may receive Nods freely between nodes (SPEC-33 §33.4). If healed to PS >= 1, Hémorragie is removed. If NOT healed between nodes and the next combat starts, the countdown **continues from where it left off** at the knight's first turn in the new encounter. |
| **Despair + Hémorragie** | Agony takes priority over Despair (SPEC-07). If both trigger simultaneously, Agony suppresses Despair. The knight is incapacitated (Agony), not hostile (Despair). If healed out of Agony but PEs is still 0, Despair activates after Hémorragie is removed. |

#### Data Model

```csharp
// Add to KnightBase combat state:
bool hasHémorragie;        // true while bleeding
int hémorragieCountdown;   // 3 then 2 then 1 then 0 (dead)

// InjuryResult for Hémorragie:
struct InjuryResult {
  int severityDie;    // 1
  int rowDie;         // 5
  string injuryId;    // "HEMORRAGIE_INTERNE"
  bool isPermanent;   // FALSE — exception, removed on Agony exit
}
```

### Acceptance Criteria
PS 1 takes 5: PS=0, Agony, Injury roll.
Ignorer l'Agonie: PS=1, no Agony, no injury roll. Can't reuse this combat.
[v8.9] Hémorragie rolled `[1,5]`: countdown = 3. Knight in Agony. Knight's turn 1: tick to 2. Knight's turn 2: tick to 1. Ally uses Nod de Soin between turn 2 and turn 3: PS restored to 7, Agony ends, Hémorragie removed from injuries. Knight acts normally.
[v8.9] Hémorragie countdown reaches 0: instant permadeath. Knight removed from roster. Ignorer l'Agonie cannot prevent this.
[v8.9] Knight uses Ignorer l'Agonie when PS reaches 0: PS stays at 1, no Agony, no injury roll. Hémorragie impossible (never rolled).
[v8.9] Hémorragie active, combat ends with countdown = 2. Between nodes: ally uses Nod de Soin. PS restored. Hémorragie removed. Knight enters next combat healthy.
[v8.9] Hémorragie active, combat ends with countdown = 2. No Nod used between nodes. Next combat starts: knight's first turn ticks countdown to 1.

---

## SPEC-10: Fold State
### Purpose
When PA reaches 0, the meta-armor folds into a minimal Guardian Suit. [v3] Fully specified (C4 resolved).

### [v3] Trigger
PA reaches 0 from any damage source. Immediate — no action, no roll.

### [v3] Guardian Suit Stats

| Stat | Value |
| --- | --- |
| PA | 5 (fixed, does not scale) |
| CdF | 5 (fixed, replaces armor base CdF only). [v8.9] **External CdF bonuses still apply:** Shrine +6 (SPEC-17.2) stacks additively on top of the Guardian Suit's CdF 5. A Folded knight at a Shrine rank has total CdF = 5 + 6 = 11. The Fold replaces only the armor's base CdF, not external buffs. IGNORE_CDF still bypasses the total (including Shrine). |
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
Nods: All Nod types usable (Soin, Armure) — Nod d'Énergie blocked (PE frozen). Nod d'Armure is the primary exit path. See [SPEC-33](./13_NodSystem.md) §33.5.
Free actions: Weapon swap, communication, style switch (but forced Standard, so no effect).
Movement: Normal movement rules. Position system unchanged.
Combo Rolls: Base + Combo (no OD auto-successes). PEs penalty still applies.
Heroism spending: All Héroïsme abilities remain available (Ignorer Agonie, Relancer, Dégâts Maximum, Mode Héroïque).

### [v3] Exit Conditions
Nod d'Armure: Restores PA. If restored PA > 5, armor reactivates. All systems come back online. See [SPEC-33](./13_NodSystem.md).
Priest Mechanic: Restores 3D6+6 or 2D6+6 PA. Same exit logic.
Any PA restoration: The moment currentPA exceeds Guardian Suit's 5, armor unfolds.
On exit: CdF returns to armor base. ODs reactivate. Modules/abilities available. PE spending unlocked.

### [v3] Edge Cases
Damage while folded: If Guardian Suit PA (5) and CdF (5) are overwhelmed, remaining damage hits PS. PS = 0 triggers Agony (SPEC-08).
PA passthrough in fold: Applies normally per SPEC-06 §Step 2b (canonical). floor(paAbsorbed/5) = PS lost. Infatigable still overrides.
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

