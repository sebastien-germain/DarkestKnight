# Integration Test Scenarios
> **Purpose:** End-to-end combat scenarios that exercise cross-system interactions. These complement the per-SPEC acceptance criteria (unit-level) with full multi-turn, multi-system integration tests.
> **Dependencies:** ALL combat SPECs
> **Depended on by:** None (test reference only)
---

## Scenario 1: Fold → Despair → Hémorragie Cascade

**Setup:** Knight K1 (Warrior, PS 40/40, PA 60/60, PE 30/30, PEs 8/50, CdF 10). One Faune (T3) at Enemy Rank 1. K1 at Knight Rank 1.

**Turn 1 — Faune attacks K1:**
1. Faune uses Griffes corrompues: 4D6+16, Contact. Attack Aspect: Bête (10).
2. Faune rolls 10D6 + 6 auto-successes (Bête Majeur). Total: e.g., 4 evens + 6 = 10 successes.
3. K1 Defense = 6. 10 > 6 → hit.
4. Damage roll: 4D6 = [3,4,5,6] = 18 + 16 = 34.
5. ArmourLayerResolver: CdF absorbs 10 → 24 remaining. PA absorbs 24 → PA drops from 60 to 36. Passthrough: floor(24/5) = 4 PS lost → PS = 36.
6. **Verify:** PS = 36, PA = 36, PEs = 8 (unchanged — not Anathème).

**Turn 2 — Faune attacks K1 again (Actions Multiples 1 = 2 attacks):**
1. Second attack: same weapon. Hit confirmed.
2. Damage: 4D6 = [2,5,6,6] = 19 + 16 = 35.
3. CdF 10 → 25 remaining. PA absorbs 25 → PA drops from 36 to 11. Passthrough: floor(25/5) = 5 → PS = 31.
4. **Faune Charge Brutale (2nd Combat Action):** 4D6+16 + 2×10 (Bête) = 4D6+36.
5. Damage: 4D6 = [4,5,5,6] = 20 + 36 = 56.
6. CdF 10 → 46. PA absorbs 11 → PA = 0. **Fold triggers** (SPEC-10). Remaining: 46 - 11 = 35 → PS. Passthrough: floor(11/5) = 2 → PS loses 2 more. Total PS loss: 35 + 2 = 37. PS = 31 - 37 = **-6 → clamped to 0**.
7. **Agony triggers** (PS = 0, SPEC-08). Injury roll: D6 column = let's say roll 5 → Hémorragie Interne.
8. **Hémorragie countdown = 3.**
9. K1 is now in **Fold** (PA = 0, Guardian Suit active: PA 5, CdF 5) AND **Agony** (PS = 0) AND **Hémorragie** (3 turns).
10. PEs check: PEs = 8. PEs penalty = max(0, 10 - 8) = 2 penalty dice.

**Expected state after Turn 2:**
- K1: PS = 0, PA = 5 (Guardian), CdF = 5, PEs = 8, Agony = true, Fold = true, Hémorragie = 3.
- K1 cannot act (Incapacitated by Agony).

**Turn 3 — Ally K2 (Priest) uses Nod de Soin on K1:**
1. K2 at Rank 2. Gap to K1 at Rank 1 = (2-1)+(1-1) = 1 ≤ 2 (Courte) → valid.
2. Nod de Soin on Agony ally → permitted (SPEC-33 §33.3 exception).
3. K2 has Guérison Rapide (Tarot XIV) → bonus +3.
4. Roll: 3D6 = [3,4,5] = 12 + 3 = 15.
5. K1 PS = 0 + 15 = 15. PS > 0 → **Agony exits.** **Hémorragie removed** (not permanent).
6. K1 is still in **Fold** (PA = 5, Guardian active).
7. Fire: OnAgonyExited(K1), OnNodHealFromAgony(K2, K1), check VOEU_OURS for K2.

**Expected state after Turn 3:**
- K1: PS = 15, PA = 5 (Guardian), CdF = 5, PEs = 8, Agony = false, Fold = true, Hémorragie = false.
- K1 can now act (limited by Fold — no ODs, no modules, no PE spending).

**Turn 4 — K2 uses Nod d'Armure on K1:**
1. Same range check → valid.
2. Roll: 3D6 = [4,5,6] = 15.
3. K1 PA = 5 + 15 = 20. 20 > 5 (Guardian threshold) → **Fold exits.**
4. K1 CdF returns to base (10). ODs, modules, PE spending all restored.

**Expected final state:**
- K1: PS = 15/40, PA = 20/60, CdF = 10, PEs = 8/50. Fully operational. No Agony, no Fold, no Hémorragie.

**Systems tested:** SPEC-05 (damage), SPEC-06 (armour layers + passthrough), SPEC-08 (agony + injury + hémorragie), SPEC-10 (fold + guardian + exit), SPEC-33 (nod soin on agony, nod armure fold exit, guérison rapide), SPEC-18.13 (charge brutale), SPEC-30 (combatant state transitions).

---

## Scenario 2: Ghost Alpha-Strike → Exploit → Dual-Wield

**Setup:** Knight K3 (Rogue, Discrétion 6, Dextérité 5, Perception 4, Masque 7). Equipped: 2× Couteau de Combat. Style: Ambidextre. One Bestian (T2) at Enemy Rank 1. K3 at Knight Rank 1.

**Turn 1 — K3 activates Ghost:**
1. Free Action: Ghost activation (SPEC-17.4). PE cost: 2 PE/turn.
2. Stealth roll: Combo Roll, Base = Discrétion (6), Combo = Dextérité (5). Pool = 6 + 5 = 11D6. OD autos from both apply. +3 Ghost auto-successes.
3. Roll 11D6 = 5 evens + OD autos (say 2) + 3 Ghost = 10 total. stealthThreshold = 10.
4. K3 is now Ghosted. Enemies cannot target K3 unless they detect (exceed threshold 10).

**Turn 2 — K3 attacks from Ghost (Ambidextre):**
1. Ambidextre style: 2 separate attacks, each at −3 dice. Couteau has Jumelé (Ambidextrie) → penalty reduced to −1.
2. **First attack (Couteau 1):** Base Combat (say 4) + Combo Dextérité (5) - 1 = 8D6. OD autos apply.
3. Ghost damage bonus: +Discrétion score to damage (Silencieux weapon from stealth).
4. Damage: 3D6 + Force(3) + Dextérité(5, Orfèvrerie) + Discrétion(6, Ghost Silencieux) = 3D6 + 14.
5. Roll 8D6 for hit: assume ALL EVEN → **Exploit!** +1 Héroïsme, +1 PEs.
6. Re-roll 8D6: 4 evens. Combined original (8, all successes since all even) + re-roll (4) = 12 total successes.
7. Bestian Defense = 4. Excess = 12 - 4 = 8. 8 bonus D6 damage dice.
8. Damage: 3D6 = [3,5,6] = 14 + 14 flat = 28. Bonus 8D6 = [2,3,3,4,5,5,6,6] = 34. Total = 62.
9. **No Exploit chain** (A-10 rule). Second all-even would grant +1 Héroïsme/+1 PEs but no further re-roll.
10. Bestian PS = 20. 62 > 20 → Bestian dies (no PA, CdF/Bouclier on T2).
11. **Second attack (Couteau 2):** No valid target (Bestian dead). Attack is forfeited.

**Expected state:**
- K3: Héroïsme +1, PEs +1 (from Exploit). Ghost **deactivated** (immediate deactivation after first attack per SPEC-17.4 §Scope — enemies later in initiative CAN target K3).
- Bestian: Dead.

**Systems tested:** SPEC-17.4 (Ghost activation + stealth roll + damage bonus), SPEC-04 (Combo Roll + Exploit + no chain), SPEC-04 dual-wield (Ambidextre + Jumelé penalty reduction), SPEC-05 (damage with Orfèvrerie + Silencieux + Ghost), SPEC-09 (Héroïsme gain), SPEC-07 (PEs gain from Exploit).

---

## Scenario 3: Barrage + Puissant Style + Boss Phase Transition

**Setup:** Knights K1 (Warrior, Rank 1), K2 (Paladin, Rank 2, Watchtower active — 2nd turn since activation). Ours Corrompu Phase 1 (PS 15/120, Bouclier 8, Défense 8) at Enemy Rank 1. Nocte bande (Cohésion 80).

**Turn 1 — K2 uses Fusil d'Assaut to Suppress boss (Barrage 2):**
1. Suppress action: 1 Combat Action. No attack roll. No damage.
2. Boss Defense reduced by 2 → effective Defense = 8 - 2 = 6 (until K2's next turn).
3. Boss Reaction reduced by 2 → effective Reaction = 4 - 2 = 2.

**Turn 1 — K2 uses Watchtower bonus ranged action (Fusil d'Assaut attack):**
1. K2's +1 ranged-only Combat Action from Watchtower.
2. Standard attack roll vs boss. Reaction = 2 (suppressed).
3. Hit confirmed. Damage: 2D6 + 6 = [4,5] + 6 = 15 Violence. Boss takes 0 Dégâts (Violence goes to Bande via SPEC-11 if applicable, otherwise standard resolution).
4. Wait — Fusil d'Assaut vs on-track enemy: Damage = 2D6+6, Violence = 3D6+9. Damage route: CdF (none on boss... actually boss has Bouclier 8). Let me recalculate.
5. Damage: 2D6+6 = [4,5]+6 = 15. Bouclier absorbs 8 → 7 remaining. PA = boss has no PA listed... Ours has PA 0 in stat block (T5 Patron — Bouclier but no PA). 7 → PS. PS = 15 - 7 = 8.
6. Violence: 3D6+9 = [3,4,6]+9 = 22. Applied to Nocte Cohésion: 80 - 22 = 58.

**Turn 1 — K1 attacks boss (melee, Marteau-Épieu contact, Perce Armure 40):**
1. Boss Defense = 6 (suppressed by K2's Barrage).
2. K1 rolls Combo Roll. Hit confirmed (successes > 6).
3. Damage: 3D6 + Force = [3,4,6] + Force(say 5) = 18. Perce Armure 40: irrelevant (boss has 0 PA).
4. Bouclier 8 → 10 remaining after Bouclier. PS = 8 - 10 = **-2 → clamped to 0.**
5. **Phase 1 PS = 0 → Phase transition triggers!**

**Phase Transition:**
1. K1's turn resolves completely (current attacker finishes).
2. Boss does NOT enter Agony. Phase 2 stats applied:
   - PS resets to 120.
   - Bouclier → 12. Défense → 10.
   - Actions Multiples → (2) = 3 Combat Actions.
   - Ombre dévorante unlocked (6D6, Longue, Anathème, Attack Aspect Bête 14).
   - AI shifts to LOWEST_PEs / LOWEST_PS, Aggression 3.
   - Régénération unlocked.
3. Bande reinforcement: Nocte Cohésion was 58 → AddCohesion(200) → 258.
4. Darkness erupts: all knights −1D6 PEs each. Roll per knight.
5. Charge Brutale cooldown **resets** (M-07 — boss may Charge again in Phase 2).

**Boss next turn — Phase 2:**
1. Régénération check: Nocte Cohésion = 258 ≥ 50 → consumes 50, boss heals 25 PS. PS = 120 + 25 = capped at 120 (assuming max is 120).
2. AI targets LOWEST_PEs knight. If K1 has PEs 5 and K2 has PEs 12 → boss targets K1.
3. Boss has 3 Combat Actions. Can attack K1 with Griffes (contact), Ombre dévorante (Longue, Anathème), and Souffle ténébreux (Moyenne, Anathème + Dispersion 3).

**Expected state after Phase Transition:**
- Boss: PS = 120, Bouclier = 12, Défense = 10, Phase = 2.
- Nocte: Cohésion = 258 - 50 (Régénération) = 208.
- All knights: PEs reduced by 1D6 each.
- Barrage debuff from K2: still active (expires at start of K2's next turn). But Phase 2 Defense is 10, so effective = 10 - 2 = 8 while Barrage lasts.

**Systems tested:** SPEC-20 (Barrage Suppress), SPEC-17.2 (Watchtower extra ranged action), SPEC-14 (Phase transition procedure + timing), SPEC-11 (Bande AddCohesion), SPEC-18.13 (Charge cooldown reset), SPEC-21.5 (Phase 2 stat changes + Régénération), SPEC-22 (darkness erupts PEs loss), SPEC-18.8 (Actions Multiples 2).

---

## Scenario 4: Grenade Friendly Fire + Despair + Stabilization

**Setup:** K1 (Warrior, Rank 1, PEs 3/50), K2 (Paladin, Rank 2, PEs 25/50). Bestian E1 at Enemy Rank 1, Bestian E2 at Enemy Rank 2. Linear track: K2(pos3) — K1(pos4) — E1(pos5) — E2(pos6).

**Turn 1 — K2 throws Shrapnel grenade at E1:**
1. Grenade stats: 3D6 damage, 3D6 violence, Dispersion 5 (radius 2).
2. Combo Roll: Tir as Base vs E1's Reaction (3). Hit confirmed (successes > 3).
3. Damage roll: 3D6 = [3,5,6] = 14. Violence: 3D6 = [2,4,6] = 12.
4. Meurtrier: damage reaches PS (Bestian has no CdF/PA) → +2D6 = [4,5] = 9. Total damage = 23.
5. Ultraviolence on violence: reroll any 1s or 2s in [2,4,6] → 2 rerolled → [5,4,6] = 15 violence.
6. E1 PS = 20. Takes 23 damage → PS = -3 → dead.
7. **Dispersion 5 (radius 2):** Blast covers pos3 to pos7 (E1 at pos5 ± 2).
   - K1 at pos4: IN BLAST. Compare K2's attack successes vs K1's Reaction. If successes > K1 Reaction → K1 is hit. Same damage (23) routed through K1's ArmourLayerResolver.
   - E2 at pos6: IN BLAST. Compare successes vs E2 Reaction (3). If > 3 → hit. Same damage.
   - K2 at pos3: IN BLAST (pos3 is within radius!). Compare successes vs K2's Reaction. If > K2 Reaction → K2 hits self. **K2 damaged their own knight.**
   - Meurtrier and Ultraviolence effects also apply to each hit splash target independently.
8. **K1 hit by friendly fire:** Damage 23 → CdF(10) absorbs 10 → 13 remaining. PA absorbs 13. Passthrough: floor(13/5) = 2 PS lost.
9. K1 PEs = 3. PEs penalty = max(0, 10-3) = 7 penalty dice on all Combo Rolls.

**Turn 2 — Bestian E1 (dead, skip). E2 attacks K1 with Flot de ténèbres (Anathème):**
1. Attack Aspect: Bête (8) + Exceptionnel Mineur (2) = 8D6 + 2 auto-successes.
2. Hit confirmed. Damage: 3D6 = [2,4,5] = 11.
3. Anathème route: CdF applies → CdF(10) absorbs 10 → 1 remaining. PA bypassed → **PEs takes 1 damage.**
4. K1 PEs = 3 - 1 = 2. Choc 1 (standard — check successes > floor(K1.Chair/2)): if Choc threshold met → K1 loses 1 action next turn.

**Turn 3 — E2 attacks K1 again with Flot de ténèbres:**
1. Same resolution. Damage: 3D6 = [3,3,6] = 12. CdF absorbs 10 → 2 remaining → PEs loses 2.
2. K1 PEs = 2 - 2 = **0. Despair triggers** (SPEC-07).
3. K1 control = EnemyControlled. Despair duration = 1D6 turns (say roll 4).
4. Fire OnDespairEntered(K1, 4).

**Turn 4 — Ally K2 attempts Stabilization (SPEC-07):**
1. K2 uses Combat Action: Stabilization attempt.
2. Combo Roll (Aura or Parole as Base). Difficulty per SPEC-07.
3. If success: K1 PEs set to 1. Despair ends. Control returns to PlayerControlled.
4. If fail: K1 remains Despaired. Duration countdown continues.

**Expected state if stabilization succeeds:**
- K1: PEs = 1/50, Despair = false, control = PlayerControlled. PEs penalty = max(0, 10-1) = 9.
- K1 is technically alive but nearly useless (9 penalty dice on everything).

**Systems tested:** SPEC-19.4 (grenade mechanics), SPEC-20.5 (Dispersion friendly fire — self-hit possible), SPEC-20.1 (Meurtrier + Ultraviolence on splash), SPEC-06 (Anathème damage route — CdF then PEs), SPEC-07 (Despair trigger at PEs 0 + stabilization), SPEC-30 (CombatantState transitions — Despair is not defeat if temporary), SPEC-20.3 (standard Choc on Anathème weapon — Choc applies normally, Anathème only changes damage routing).

---

## Scenario 5: Full Mission Node Transition — Attrition Check

**Setup:** End of Node 2A combat. 4 knights surviving with:
- K1: PS 25/40, PA 30/60, PE 18/30, PEs 35/50. 2 Nod Soin, 3 Nod Armure, 3 Nod Énergie.
- K2: PS 40/40, PA 10/60, PE 5/30, PEs 42/50. 3/3/3 Nods.
- K3: PS 0/40 (in Agony, Hémorragie countdown = 2). 1/3/3 Nods.
- K4: PS 35/40, PA 55/60, PE 28/30, PEs 15/50. 3/3/3 Nods.

**Node Transition (SPEC-13 — No Rest):**
1. All state carries over exactly. No auto-heal.
2. K3 is in Agony with Hémorragie (carried forward per SPEC-32 field boundary — hasHémorragie is a RunState exception).
3. **Between-node Nod use (SPEC-33 §33.4):** Free, no action cost, no range restriction.
4. K1 uses Nod de Soin on K3: 3D6 = [4,5,6] = 15. K3 PS = 0 + 15 = 15. Agony exits. Hémorragie removed.
5. K2 uses Nod d'Armure on self: 3D6 = [2,3,4] = 9. K2 PA = 10 + 9 = 19.
6. K4 uses Nod d'Énergie on K2: 3D6 = [3,3,5] = 11. K2 PE = 5 + 11 = 16.
7. Decision: Save remaining Nods for Node 3 (hardest non-boss fight) and boss.

**State entering Node 3:**
- K1: PS 25, PA 30, PE 18, PEs 35. Nods: Soin 1, Armure 3, Énergie 3.
- K2: PS 40, PA 19, PE 16, PEs 42. Nods: 3/2/2.
- K3: PS 15, PA (whatever it was — carry over), PE (carry over), PEs (carry over). Nods: 1/3/3 (K1 used their own Soin on K3).
- K4: PS 35, PA 55, PE 28, PEs 15. Nods: 3/3/2.

**Verify:**
- No PS/PA/PE auto-recovery occurred.
- PEs unchanged (no motivation trigger between nodes, no rest recovery).
- Hémorragie resolved by Nod (not by auto-heal).
- Injuries persist (if K3 had any permanent injuries from the Agony roll besides Hémorragie, they carry forward).
- Dead knights (if any) would remain dead and leave empty rank slots.

**Systems tested:** SPEC-13 (no rest between nodes, full attrition), SPEC-33 §33.4 (between-node Nod use — free, no range), SPEC-33 §33.7 (Nod Soin saves from Hémorragie), SPEC-32 (RunState field boundary — Hémorragie carries across encounters), SPEC-08 (Agony + Hémorragie persistence).

---

*End of integration test scenarios. Each scenario is designed to exercise the most complex cross-system interactions that isolated acceptance criteria cannot catch. Agents should implement these as automated integration tests once the core combat systems are in place.*

