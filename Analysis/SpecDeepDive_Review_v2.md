# DarkestKnight — Specification Deep-Dive Review v2
**Date:** March 10, 2026  
**Reviewer:** Game Design Expert (20+ yrs 2D RPG / Unity)  
**Scope:** All 15 specification files (`00_INDEX.md` → `14_Constants.md`)  
**Goal:** Identify every ambiguity, gap, duplication, or contradiction that would block an autonomous agent team from implementing this spec without human intervention.

---

## Executive Summary

The specification suite is **remarkably thorough** for a tabletop-to-digital adaptation — far above average. The modular split, dependency map, agent task boundaries, and acceptance criteria are all strong. However, I've identified **62 issues** across 7 severity categories that would cause agent confusion, conflicting implementations, or silent bugs.

| Severity | Count | Description |
|----------|-------|-------------|
| 🔴 **BLOCKER** | 8 | Agent cannot proceed without human decision. |
| 🟠 **CONTRADICTION** | 9 | Two specs say different things about the same mechanic. |
| 🟡 **AMBIGUITY** | 16 | Spec is unclear — two agents could implement it differently. |
| 🔵 **MISSING** | 14 | Mechanic referenced but never defined. |
| 🟣 **DUPLICATION** | 8 | Same info in multiple places risks desync. |
| ⚪ **MINOR** | 5 | Cosmetic/naming inconsistencies. |
| 💡 **SUGGESTION** | 2 | Non-blocking design recommendations. |

---

## 🔴 BLOCKER Issues (Agent Cannot Proceed)

### B-01: ✅ RESOLVED — Lointaine Max Gap — Infinity vs 6
**Files:** `14_Constants.md` §7 Global Constants, `02_PositionAndTurns.md` SPEC-03  
**Problem:** The Constants file (§7) lists Lointaine Max Gap as `∞`, but the Gap Matrix in SPEC-03 shows the maximum possible gap is **6** (K4→E4). If the max gap is ∞, the gap check `(attackerRank-1)+(targetRank-1) <= maxGap` would need a special sentinel value. If it's 6, just use 6.  
**Impact:** Data agent and Combat Core agent will implement different values for `WeaponRange.LOINTAINE`.  
**Fix:** In `14_Constants.md`, change `∞` to `6` (the actual max gap on the track). Add a note: "Effectively unlimited on the 4-rank track."

---

### B-02: ✅ RESOLVED — Ours Corrompu Phase 2 — AI Change Undefined
**Files:** `08_ContentData.md` SPEC-21.5 (Phase 2 table)  
**Problem:** The Phase 2 stat change table has a row `AI change` with **empty value**. Does the boss keep Phase 1 AI (`HIGHEST_BETE / HIGHEST_COMBAT, ADVANCE, Aggression 2`)? Does it change targeting? This is critical for the Enemy AI Agent.  
**Impact:** Enemy AI Agent must guess boss Phase 2 targeting behavior.  
**Fix:** Explicitly state the Phase 2 AI profile. Suggestion: "Same as Phase 1" or define a new priority (e.g., `LOWEST_PEs` to leverage Ombre dévorante's Anathème potential).

---

### B-03: ✅ RESOLVED — Ours Corrompu Phase 2 — Ombre Dévorante Attack Aspect Undefined
**Files:** `08_ContentData.md` SPEC-21.5  
**Problem:** Phase 2 introduces "Ombre dévorante: 6D6, Longue, Ignore CdF + Dégâts Continus 3" but does not specify the **attack Aspect**. Per SPEC-18.9 Step 2, ranged weapons default to "highest of Machine or Masque." Ours has Machine 4, Masque 8 — so it would be Masque (8D6 pool). But this is a darkness-based weapon, thematically closer to Bête. The boss stat block must be unambiguous.  
**Impact:** Enemy AI Agent uses wrong dice pool.  
**Fix:** Add `Attack Aspect: Masque (8)` or `Attack Aspect: Bête (14)` to the Ombre dévorante weapon entry. Given it's a shadow attack, Bête seems appropriate, but this is a design call.

---

### B-04: ✅ RESOLVED — Ours Corrompu — Is Ombre Dévorante an Anathème Attack?
**Files:** `08_ContentData.md` SPEC-21.5, `04_ArmorAndState.md` SPEC-06  
**Problem:** The Ours has "Souffle ténébreux: Anathème" in Phase 1, and Phase 2 adds "Ombre dévorante: Ignore CdF + Dégâts Continus 3." But Ombre dévorante does NOT carry the Anathème tag. Is it Anathème (damages PEs, bypasses PA per SPEC-06 §Knight Defense vs Anathème)? Or is it a standard attack that just happens to Ignore CdF? This drastically changes the damage pathway.  
**Impact:** If Anathème → damage hits PEs directly. If not → damage goes CdF(skipped)→PA→PS. Completely different resolution path.  
**Fix:** Explicitly tag Ombre dévorante as `Anathème` or `Non-Anathème`. Given thematic consistency ("shadow devouring"), Anathème seems likely.

---

### B-05: ✅ RESOLVED — Bestian Flot de Ténèbres — Anathème Attack Aspect Undefined
**Files:** `08_ContentData.md` SPEC-21.2  
**Problem:** Bestian's "Flot de ténèbres: 3D6, Moyenne, Choc 1 + Anathème" doesn't specify the attack Aspect. The AI note says "AI always targets PEs" — but that's targeting priority, not which Aspect the attack roll uses. Default per SPEC-18.9: ranged → highest of Machine(2) or Masque(5) = Masque 5. But the Bestian's Bête Exceptionnel Mineur (2) only applies to Bête rolls. If the attack uses Masque, the Exceptionnel auto-successes don't apply. If it uses Bête (8D6 + 2 auto), it's much deadlier.  
**Impact:** Wrong dice pool and auto-successes for the most common T2 enemy's ranged attack.  
**Fix:** Add explicit `Attack Aspect: Masque (5)` or `Attack Aspect: Bête (8)` to the Flot de ténèbres weapon entry.

---

### B-06: ✅ RESOLVED — Peur Test — Is It a Combo Roll or Not?
**Files:** `06_EnemySystem.md` SPEC-18.10  
**Problem:** Peur says "Each knight tests base Sang-Froid or Hargne (digital: highest of the two) vs opposition." SPEC-04 defines the Combo Roll as always requiring a Base + Combo (two different Characteristics). The Peur test mentions only one Characteristic. Is the Peur test a standard Combo Roll (meaning the knight picks a second Characteristic and adds ODs)? Or is it a single-Characteristic roll (just that score as the pool, no combo, no ODs)? The spec also states "PEs penalty applies to this roll (SPEC-07 — all Combo Rolls are affected)" — implying it IS a Combo Roll.  
**Impact:** A Combo Roll with Sang-Froid 5 + Hargne 4 + ODs could be 11+ dice. A single-Characteristic roll would be 5 dice. Massive difference.  
**Fix:** Clarify: "Peur test is a Combo Roll with Base = highest of (Sang-Froid, Hargne). Player selects Combo Characteristic per standard rules (any Characteristic except Base). OD auto-successes apply." Or: "Peur test uses a single dice pool = highest of (Sang-Froid, Hargne). No Combo. No OD auto-successes. PEs penalty still applies."

---

### B-07: ✅ RESOLVED — Points de Contact — Computed But "NOT Mechanically Used"
**Files:** `01_KnightDataModel.md` SPEC-01 Derived Value Formulas  
**Problem:** "Points de Contact = Max(aura, parole, sangFroid). Computed and stored but NOT mechanically used in demo." This is confusing for the Data Agent: should they implement the computation or skip it? The text then says "Future full-game design: Points de Contact will grant pre-mission bonuses" — but this creates dead code.  
**Impact:** Data agent spends time implementing unused derivation; or worse, skips it and future agent expects it.  
**Fix:** Remove from the Derived Value Formulas table (move to a "Future" appendix), OR explicitly state: "Data Agent: compute and store this value. No consumer exists in demo. This is scaffolding only."

---

### B-08: ✅ RESOLVED — Cadence X — Target Adjacency vs Attacker Range
**Files:** `07_WeaponsAndEffects.md` SPEC-20.4  
**Problem:** The Cadence X effect says "all Cadence targets must be on adjacent enemy ranks (within 1 rank of each other)." But the example in Acceptance Criteria references "E2 at enemy Rank 2" and "E3 at enemy Rank 4" as being "2 ranks from E1 → invalid, too far." Wait — E2 is at enemy Rank 2 and E1 is at enemy Rank 2? That's the same rank. And no demo weapon actually has Cadence X. Is this mechanic even used in demo scope?  
**Impact:** The acceptance criteria example appears to have an inconsistency (both E1 and E2 at Rank 2?). Combat Core Agent may waste time implementing Cadence for no demo weapon.  
**Fix:** Either mark Cadence as "scaffolded, not in demo" (no demo weapon has it), or fix the acceptance criteria example to use sensible rank values. Clarify: does any demo weapon or enemy use Cadence?

---

## 🟠 CONTRADICTION Issues

### C-01: ✅ RESOLVED — Mise à Couvert — Requires Cover or Not?
**Files:** `03_CombatRolls.md` SPEC-04 Style Modifiers table vs `02_PositionAndTurns.md` SPEC-03 Cover System  
**Problem:** The Style Modifiers table describes Mise à Couvert as a generic style with "+2 Reaction (ranged only)" and doesn't mention requiring a Cover rank. The SPEC-03 Cover Removal Procedure says "combatants using Mise à Couvert forced to Standard" when cover is removed — implying the style IS linked to cover. But can a knight use Mise à Couvert at a rank without cover?  
**Impact:** Two agents may disagree on whether `hasCover` is a prerequisite for the style.  
**Fix:** Add to the Style Modifiers table: "Restriction: Rank must have `hasCover = true`. If cover is removed, forced to Standard (SPEC-03)." OR: "Mise à Couvert can be used at any rank for the +2 Reaction bonus. Cover rank adds an additional +3 (stacking to +5)."

---

### C-02: ✅ RESOLVED — Action Economy — Combat Action Flexibility
**Files:** `03_CombatRolls.md` SPEC-04 Action Economy  
**Problem:** "Combat Action can be used for any action requiring a Combat Action OR a Movement Action (it is the more flexible action type)." But several places (SPEC-15 Turn Structure) say "May forfeit Combat Action for 2nd Movement Action. Cannot reverse." These two descriptions disagree: is forfeiting the Combat Action for movement a special decision declared at turn start, or can the knight freely use their Combat Action as a Movement Action at any time?  
**Impact:** If it's free substitution, a knight can always use both actions as movement without explicit forfeit. If it requires a forfeit declaration, the UI and combat manager must handle it differently.  
**Fix:** Clarify: "A Combat Action may be spent on any Movement-cost action at any time during the turn, without explicit forfeit. 'Forfeiting Combat for 2nd Movement' is the natural consequence, not a separate mechanic."

---

### C-03: ✅ RESOLVED — Nod on Ally in Agony — SPEC-04 vs SPEC-33
**Files:** `02_PositionAndTurns.md` SPEC-15, `13_NodSystem.md` SPEC-33  
**Problem:** SPEC-15 says "Nod on ally: Target must be a living ally (not in Agony — exception: Nod de Soin, see SPEC-33)." But SPEC-33 §33.3 says "The 'not in Agony' restriction in the SPEC-04 action economy table is overridden for Nod de Soin specifically." The SPEC-04 action economy table itself doesn't mention the Agony restriction at all — the reference is to SPEC-15. This is a three-way reference loop.  
**Impact:** Agents see different rules in different files.  
**Fix:** Make SPEC-33 the single canonical source for Nod targeting. In SPEC-15 and SPEC-04, add: "For Nod targeting rules including Agony exceptions, see SPEC-33 §33.3 (canonical)."

---

### C-04: ✅ RESOLVED — Bande Defense/Reaction — Derived or Hand-Authored?
**Files:** `06_EnemySystem.md` SPEC-11 vs SPEC-18.3  
**Problem:** SPEC-18.3 states "Defense/Reaction values are authoritative per stat block... Do NOT derive them from a formula." SPEC-11 §Knight vs Bande Attack Procedure says "Compare total successes vs the Bande's Defense (contact attacks) or Reaction (ranged attacks)" and references "Nocte: Defense 5, Reaction 1." But the Nocte stat block in SPEC-21.1 has `Défense 5 / Réaction 1 / Initiative 1` listed in a single-line format that could be confused with Aspect-derived values.  
**Impact:** Minor — values are present, but the formatting in SPEC-21.1 doesn't clearly separate them from Aspects as "hand-authored."  
**Fix:** In SPEC-21.1, format Defense/Reaction as a dedicated table row (like T2+ enemies) rather than a single inline `Défense 5 / Réaction 1 / Initiative 1` string.

---

### C-05: ✅ RESOLVED — Ghost Stealth Roll — Is Combo Characteristic Player-Chosen?
**Files:** `05_ArmorAbilities.md` SPEC-17.4  
**Problem:** Ghost Activation says "Combo Roll with Discrétion as Base. Store result as stealthThreshold." A Combo Roll requires a second Characteristic. But the spec doesn't say which Combo Characteristic the Rogue uses. Does the player choose any non-Discrétion Characteristic? Does it default to Dextérité? The stealth threshold depends heavily on this choice.  
**Impact:** Rogue with Discrétion 5 + Perception 4 = 9D6 is very different from Discrétion 5 + Combat 2 = 7D6.  
**Fix:** Add: "Player chooses Combo Characteristic per standard Combo Roll rules (any Characteristic except Discrétion). +3 auto-successes added to the stealth roll (not the Combo Roll's ODs)."

---

### C-06: ✅ RESOLVED — Guérison Rapide — Nod Soin +3 vs "Healing Recovery Doubled"
**Files:** `09_TarotSystem.md` SPEC-24.2, `13_NodSystem.md` SPEC-33.6  
**Problem:** SPEC-24.2 says Guérison Rapide = "Healing recovery doubled. Nod Soin: +3 extra PS recovered." SPEC-33.6 says "Nod de Soin restores 3D6 + 3." But "doubled" and "+3 flat" are contradictory. If a Nod de Soin rolls 12, is the result 24 (doubled) or 15 (+3)?  
**Impact:** Two agents implement different formulas.  
**Fix:** Pick one. SPEC-33 says "+3 flat bonus" and is more specific. Amend SPEC-24.2 to: "Nod Soin: +3 flat PS bonus on top of the 3D6 roll." Remove or rephrase "Healing recovery doubled" to avoid confusion — or clarify that "doubled" applies to other healing sources (Camelot rest, future psychologist) and Nod has the specific "+3" rule.

---

### C-07: ✅ RESOLVED — Dispersion — Primary Target Auto-Hit or Separate Roll?
**Files:** `07_WeaponsAndEffects.md` SPEC-20.5  
**Problem:** The Dispersion Resolution pseudocode skips the primary target (`if combatant == primaryTarget: continue`) and only resolves splash targets. But it says "One attack roll for the primary; compare vs each splash target's Defense/Reaction separately." This implies the primary was already resolved by the normal attack, and splash targets reuse that same roll. However, the pseudocode compares `attackSuccesses > threshold` for splash targets — are splash targets each compared against their own Defense/Reaction using the **original** attack roll's successes? Or do they get separate rolls?  
**Impact:** One roll for primary + reuse for splash (efficient but different thresholds) vs separate rolls per splash target (more rolls, more variance).  
**Fix:** Clarify: "The primary target is resolved by the standard attack roll. Splash targets reuse the primary attack roll's total successes but compare against each splash target's individual Defense (contact) or Reaction (ranged) threshold. No separate roll for splash targets." This appears to be the intent based on the pseudocode.

---

### C-08: ✅ RESOLVED — Dispersion — Friendly Fire Damage Amount
**Files:** `07_WeaponsAndEffects.md` SPEC-20.5  
**Problem:** "All hit targets take full damage." Does "full damage" mean the same damage roll result as the primary? Or a separate damage roll? The text "One attack roll for the primary" and "compare vs each splash target" suggests one roll, one damage. But it's ambiguous whether damage is re-rolled per target.  
**Impact:** Significant balance difference.  
**Fix:** Add: "One damage roll is performed. All hit targets (primary and splash) take the same damage amount, routed through each target's individual ArmourLayerResolver."

---

### C-09: ✅ RESOLVED — Bande Débordement Turn Counter — When Does It Increment?
**Files:** `06_EnemySystem.md` SPEC-11 Data Model  
**Problem:** The `BandeController` class has `OnTurnEnd() { debordementTurn++; }`. But "Turn End" is ambiguous — is it at the end of the Bande's own turn (initiative 1, always last), or at the end of the entire round? The Débordement damage formula is `débordementTurn × débordementScore`. If the counter increments at the Bande's turn end, then on Turn 1 the Bande deals `1 × score`, then increments to 2. On Turn 2 it deals `2 × score`. This matches the acceptance criteria. But if it increments at round end, the timing could differ.  
**Impact:** Correct timing matters for cumulative damage calculation.  
**Fix:** Clarify: "`debordementTurn` starts at 1. At the Bande's initiative (always last in the round), it deals `debordementTurn × debordementScore` to all opposing-faction combatants, THEN increments `debordementTurn` by 1." The acceptance criteria confirms this: "Turn 1=3, Turn 2=6, Turn 3=9" with score 3.

---

## 🟡 AMBIGUITY Issues

### A-01: ✅ RESOLVED — Enemy Failure Critique — "+2 Auto-Successes" for Next Knight Attack
**Files:** `06_EnemySystem.md` SPEC-18.9 Step 6  
**Problem:** "Attack auto-misses. Enemy is exposed — next knight attack vs this enemy gets +2 auto-successes." How long does this last? Just the very next knight attack, or until the enemy's next turn? If multiple knights act before the exposed enemy, do they all get the bonus?  
**Fix:** Clarify duration: "The +2 auto-successes apply to the **first** knight attack against this enemy after the Failure Critique. Once consumed by one attack, the bonus is removed."

---

### A-02: ✅ RESOLVED — Warrior Type Selection at Generation — "Default 3 = Soldier + Hunter + 1 random"
**Files:** `05_ArmorAbilities.md` SPEC-17.1  
**Problem:** The spec says "Default 3 Types = Soldier + Hunter + 1 random." But it also says "3 chosen from 5." Are both Warriors in the 8-knight roster given the same 3 types? Or is the 3rd type random per Warrior? Can the random type roll Soldier or Hunter again (duplicate)?  
**Fix:** Clarify: "Each Warrior knight's 3rd Type is randomly selected from {Scholar, Herald, Scout} (excluding the 2 defaults: Soldier, Hunter). Each Warrior rolls independently."

---

### A-03: ✅ RESOLVED — Paladin Watchtower — "+1 Ranged Combat Action per Turn (Starting Next Turn)"
**Files:** `05_ArmorAbilities.md` SPEC-17.2  
**Problem:** "Starting next turn" — does this mean the bonus shot is only available from the turn AFTER activation? Or from this turn onwards? Also, is this +1 Combat Action total (like Actions Multiples) or +1 RANGED attack specifically?  
**Fix:** Clarify: "On the turn Watchtower is activated, the Paladin gains no extra action (they spent 1 Movement Action to activate). Starting on the Paladin's next turn and every subsequent turn while Watchtower is active: the Paladin gains +1 Combat Action that can ONLY be used for a ranged attack."

---

### A-04: ✅ RESOLVED — Régénération Capacity (Ours P2) — When Does It Trigger?
**Files:** `06_EnemySystem.md` SPEC-14, `08_ContentData.md` SPEC-21.5  
**Problem:** SPEC-14 says "Régénération — once per turn, at start of boss turn." But is this before or after the boss's other start-of-turn effects (like Peur, which is at encounter start)? Also, if the Nocte bande has < 50 Cohésion, does it consume all remaining Cohésion (healing less) or fail entirely?  
**Fix:** Add: "Régénération triggers at the start of the boss's turn, before any attacks. Requires Nocte Cohésion ≥ 50. If Cohésion < 50: Régénération does not fire (it does not partially consume)."

---

### A-05: ✅ RESOLVED (SCAFFOLDED) — En Chaîne — "One Free Attack on Another Target at Same Range, Same Turn"
**Files:** `07_WeaponsAndEffects.md` SPEC-20.1  
**Problem:** No demo weapon has En Chaîne. Is this scaffolded only? Also, "same range" — does this mean the same range band, or within the weapon's max gap? The chain attack — does it use the same Combo Roll, or a fresh roll?  
**Fix:** Mark as "scaffolded, no demo weapon." If keeping: "Same weapon, new Combo Roll, target must be within the weapon's range band. 3 PE cost."

---

### A-06: ✅ RESOLVED (SCAFFOLDED) — Assassin X — "Surprise Attacks Only"
**Files:** `07_WeaponsAndEffects.md` SPEC-20.4  
**Problem:** "isSurpriseAttack flag on combat init." No specification defines surprise attacks, ambush mechanics, or when this flag is set. No demo encounter uses it.  
**Fix:** Mark as "scaffolded, no demo implementation." Or define surprise: "Surprise attack = first attack on an unaware target (Ghost stealth attack counts)."

---

### A-07: ✅ RESOLVED (SCAFFOLDED) — Désignation — "Mark One Target per Turn" and Scope
**Files:** `07_WeaponsAndEffects.md` SPEC-20.4  
**Problem:** "Mark one target (not bande) per turn. Allies get +1 auto-success vs target. No action cost." Only the Fusil de Précision has Désignation in demo. Questions: (1) Does marking persist until a new target is marked, or just until end of turn? (2) If the Fusil holder attacks a target, do they auto-Désignate it? Or is Désignation a separate free action?  
**Fix:** Clarify: "Désignation is a Free Action (once per turn). The knight declares a target — that target gains the 'Designated' status until the designating knight marks a new target or dies. +1 auto-success applies to ALL attacks from ALL allies against the Designated target."

---

### A-08: ✅ RESOLVED (SCAFFOLDED) — Démoralisant — "Reduces Bande Débordement by 2 for Current Turn Only"
**Files:** `07_WeaponsAndEffects.md` SPEC-20.3  
**Problem:** No demo weapon or grenade has the Démoralisant effect. Is it scaffolded?  
**Fix:** Mark as "scaffolded, no demo weapon." Remove from demo scope or keep as reference-only.

---

### A-09: ✅ RESOLVED (SCAFFOLDED) — Mode Héroïque — "Declare Objective" and End Conditions
**Files:** `03_CombatRolls.md` SPEC-09  
**Problem:** "Declare objective." What constitutes a valid objective? Kill an enemy? Clear the encounter? This is vague for digital implementation. Also "Ends on: objective complete OR hero at 0 PS/PEs." How does the system determine "objective complete"?  
**Fix:** Define digital objectives: "Objective = kill a specific target enemy OR survive N turns OR clear the encounter. The player selects from a menu. The system checks the objective condition at end of each turn."

---

### A-10: ✅ RESOLVED — Multiple Exploits from Same Roll — Re-Roll Chain?
**Files:** `03_CombatRolls.md` SPEC-04  
**Problem:** "On Exploit, re-roll the entire dice pool. New successes from the re-roll are ADDED." But what if the re-roll is also all-even? Is there a chain? The tabletop typically allows chaining. The spec doesn't mention this case.  
**Fix:** Add: "If the re-roll is also an Exploit (all even), do NOT chain — only one re-roll per Combo Roll. The re-roll Exploit still grants +1 Héroïsme and +1 PEs." Or if chaining: "Exploit chains until a re-roll is not all-even. Each chain grants another +1 Héroïsme and +1 PEs."

---

### A-11: ✅ RESOLVED — Failure Critique on Enemy — "+2 Auto-Successes" Stacking
**Files:** `06_EnemySystem.md` SPEC-18.9  
**Problem:** If two enemies both Failure Critique in the same round, does the next knight get +4 auto-successes vs them? Or +2 each independently?  
**Fix:** Clarify: "+2 applies per-enemy. If E1 and E2 both Failure Critique, a knight attacking E1 gets +2, and a knight attacking E2 gets +2. Not additive across enemies."

---

### A-12: ✅ RESOLVED — Shrine Range — "Rank ±1"
**Files:** `05_ArmorAbilities.md` SPEC-17.2  
**Problem:** "+6 CdF to all allies at Shrine rank ±1." If the Shrine is at Rank 2, does it cover Rank 1, 2, and 3? Or just Rank 1 and 3 (excluding the Shrine's own rank)? "Rank ±1" typically means the Shrine's rank AND adjacent ranks (3 ranks total).  
**Fix:** Clarify: "+6 CdF to all allied combatants at the Shrine's rank, rank-1, and rank+1 (up to 3 ranks total). The Shrine's rank is included."

---

### A-13: ✅ RESOLVED (SCAFFOLDED) — Paladin Shrine — Can Enemies Enter Shrine Rank?
**Files:** `05_ArmorAbilities.md` SPEC-17.2  
**Problem:** "Entry restriction: Enemies: Force<5 or Chair<10 can't enter." This implies enemies CAN enter if they meet the threshold. But the Shrine is on the knight side of the track. Enemies occupy enemy ranks 1–4. How does an enemy "enter" a knight rank? Forced displacement? Charge Brutale? Or does this restriction apply to the Shrine's *area of effect* rather than physical entry?  
**Fix:** Clarify that this restriction is about Bande Débordement specifically: "Bande Débordement CdF still applies" is already mentioned. Rewrite: "Entry restriction governs which enemies can physically attack combatants in the Shrine zone. Enemies below Force 5 or Chair 10 cannot target combatants within Shrine ±1 ranks with contact attacks. Ranged attacks are unaffected."  
Or, if this is truly about physical entry in the linear track, explain how enemies would reach knight-side ranks.

---

### A-14: ✅ RESOLVED — Tarot Cards — Aspect +1 Exceeding Cap?
**Files:** `09_TarotSystem.md` SPEC-24.4  
**Problem:** Each card grants "+1 Aspect." With 5 cards, a knight could gain up to +5 to various Aspects (base 2 + up to +5 = 7). With Haut Fait (+1 more Aspect), that's potentially 8. The normal Aspect cap is 9. The Vétéran disadvantage caps at 7. But the spec never states an Aspect cap enforcement during generation — only that Characteristics are capped at parent Aspect.  
**Fix:** Confirm: "Aspect scores during generation are uncapped (normal max 9 applies to final value). The Vétéran disadvantage (Tarot XIII) hard caps at 7 — checked as a post-generation validation step."

---

### A-15: ✅ RESOLVED — Le Pendu — Code Moral as 4th Minor Motivation
**Files:** `09_TarotSystem.md` SPEC-24.2  
**Problem:** Code Moral adds a 4th minor Motivation. But SPEC-01 KnightBase has `Motivation[] minorMotivations; // 3 entries`. The array size is hardcoded at 3. Similarly, Le Monde (XXI) adds a dormant 4th minor motivation. If a knight draws BOTH Le Pendu AND Le Monde, they'd have 5 minor motivations.  
**Fix:** Change `minorMotivations` to a dynamic list with no hardcoded size cap, or explicitly state "max 4 minor motivations per knight (3 base + 1 from Tarot). Le Pendu and Le Monde cannot both apply (drawing both is handled by the 5-card selection logic)."

---

### A-16: ✅ RESOLVED — Esprit de Contradiction (L'Impératrice) — Ally's "Next Combo Roll This Turn"
**Files:** `09_TarotSystem.md` SPEC-24.3  
**Problem:** "That ally's next Combo Roll **this turn** has +1 difficulty." But the proc triggers "at the start of each of this knight's combat turns." If the contrarian knight acts before the target ally in initiative order, "this turn" = this round, and the ally hasn't acted yet. If the contrarian knight acts AFTER the ally in initiative... the ally already acted this round. Does "this turn" mean "this round" or "the ally's next turn whenever it happens"?  
**Fix:** Clarify: "The +1 difficulty applies to the target ally's next Combo Roll, regardless of when it occurs (this round or next if the ally already acted). The debuff lasts until consumed by one Combo Roll."

---

## 🔵 MISSING Definitions

### M-01: ✅ RESOLVED — No Specification for Priest Mechanic Range Validation
**Files:** `05_ArmorAbilities.md` SPEC-17.3  
**Problem:** "Contact (same position): 3D6+6 PA. Long (any position): 2D6+6 PA." But "same position" isn't a valid range in the DD rank system. Does "Contact" mean same rank (gap = 0), or gap ≤ 1 (Contact range)? Does "Long" mean Longue range (gap ≤ 5) or literally any position? Given the 4-rank system, "any position" effectively means unlimited.  
**Fix:** Redefine: "Contact: target must be at the Priest's rank (gap = 0). Long: any rank on the knight side (gap ≤ 6, effectively unlimited)."

---

### M-02: ✅ RESOLVED (SCAFFOLDED via A-06) — No Definition of Surprise/Ambush Mechanics
**Files:** `07_WeaponsAndEffects.md` SPEC-20.4 (Assassin X)  
**Problem:** Assassin X references `isSurpriseAttack` but surprise is never defined.  
**Fix:** Mark Assassin X as scaffolded, or define surprise: "First combat turn where one side was undetected."

---

### M-03: ✅ RESOLVED — Choc Auto-Apply — Chair Threshold for Standard vs Auto Choc
**Files:** `07_WeaponsAndEffects.md` SPEC-20.3  
**Problem:** Choc X says "If attack roll exceeds target's Chair/2 (round down): target loses X actions." But Flashbang grenade says "Choc 1 (auto)" — implying no Chair check. The difference between standard Choc and "auto-Choc" isn't formally defined.  
**Fix:** Add to Choc definition: "Standard Choc: requires successes > floor(target.Chair / 2). Auto-Choc: always applies on hit (skip Chair comparison). Weapons specify which variant via a boolean `isAutoChoc` flag on the effect."

---

### M-04: ✅ RESOLVED — Enemy Weapon — Ranged Attack Flat Bonus Pre-Baking Rules
**Files:** `06_EnemySystem.md` SPEC-18.9 Step 7  
**Problem:** "PNJ weapon stat blocks include ALL flat bonuses pre-baked." For contact weapons, the pre-baking is clearly documented (Bête Aspect + Exceptionnel). For ranged weapons (e.g., Bestian's Flot de ténèbres 3D6), there's no flat bonus — but is that because ranged enemies don't get flat bonuses, or because the Bestian specifically doesn't? The Ours Corrompu's Souffle ténébreux is also 3D6 flat. If Précision were on an enemy ranged weapon, would Machine/2 be pre-baked?  
**Fix:** Add a note: "Ranged enemy weapons: if no flat bonus is listed, there is none. The +Aspect/Exceptionnel pre-baking only applies to contact weapons via Bête Exceptionnel. Ranged flat bonuses (e.g., from Précision) would be pre-baked if present — currently no demo enemy has them."

---

### M-05: ✅ RESOLVED — Behemot — Immune to Forced Displacement, but How?
**Files:** `08_ContentData.md` SPEC-21.4  
**Problem:** "Immune to forced displacement" is stated but there's no general mechanic for displacement immunity. Is this a tag on EnemyBase? A capacity? How does the Combat Core Agent check it?  
**Fix:** Add a boolean `immuneToForcedDisplacement` to EnemyBase, or define it as a named capacity. Reference in SPEC-18.

---

### M-06: ✅ RESOLVED — Ours Corrompu Phase Transition — Does the Boss Get a Turn Immediately?
**Files:** `06_EnemySystem.md` SPEC-14  
**Problem:** Phase transition happens when Phase 1 PS = 0. But at what point in the turn? Mid-attack? End of attacker's turn? Does the boss get to act with Phase 2 stats this round, or does Phase 2 start on the boss's next initiative?  
**Fix:** Add: "Phase transition is immediate when PS reaches 0. The current attacker's turn resolves completely. The boss's next turn (at its normal initiative) uses Phase 2 stats. If the boss already acted this round, Phase 2 does not grant an additional turn — the boss must wait for the next round."

---

### M-07: ✅ RESOLVED — Ours Corrompu — Charge Brutale "Once Per Phase" Clarification
**Files:** `06_EnemySystem.md` SPEC-18.13  
**Problem:** "Once per combat" for Faune/Behemot, "Once per phase" for Ours. Does "once per phase" mean the Charge resets when Phase 2 starts? If the boss used Charge in Phase 1, can it Charge again in Phase 2?  
**Fix:** Confirm: "Yes — the Charge Brutale cooldown resets on phase transition. The boss may Charge once in Phase 1 AND once in Phase 2."

---

### M-08: ✅ RESOLVED — Choc/Parasitage — What Exactly Is "Loses X Actions"?
**Files:** `07_WeaponsAndEffects.md` SPEC-20.3  
**Problem:** "Target loses X next actions." Does this mean X Combat Actions, X Movement Actions, or X total actions (Combat + Movement combined)? For a standard knight turn (1 Combat + 1 Movement = 2 actions), Choc 2 would mean the knight loses their entire next turn. For enemies with Actions Multiples, Choc 2 out of 3 Combat Actions would still leave 1 Combat Action.  
**Fix:** Clarify: "X actions = X total actions of any type. The target skips the next X actions they would normally take. Choc 1 = skip 1 action (e.g., lose Combat Action, keep Movement). Choc 2 = skip 2 actions (e.g., lose both Combat and Movement = full turn lost for a standard combatant)."

---

### M-09: ✅ RESOLVED — Parasitage — "Machine-Based Targets" Scope
**Files:** `07_WeaponsAndEffects.md` SPEC-20.3  
**Problem:** "Requires Machine-based targets or IEM sensitivity." Are knights in meta-armor "Machine-based"? All knight armor is technological. If Parasitage can affect knights, IEM grenades become extremely powerful (Parasitage 2 on all knights in range).  
**Fix:** Clarify: "Parasitage affects enemies with the Machine seigneur tag OR any combatant with active technological systems (meta-armor qualifies). Knights are vulnerable to Parasitage via IEM grenades."

---

### M-10: ✅ RESOLVED — Narrative Event Triggers — Who Determines PEs Effects?
**Files:** `10_MissionAndHub.md` SPEC-22.3  
**Problem:** SPEC-07 Loss Source #2 says "Narrative event choices: Variable per event (designer-authored)." But the Mission Structure only defines 4 specific narrative events. If an agent needs to add narrative events, there's no template or schema for authoring them.  
**Fix:** Add a simple schema: `struct NarrativeEvent { string eventId; string description; PEsEffect[] effects; PlayerChoice[] choices; }` to SPEC-22 or SPEC-29.

---

### M-11: ✅ RESOLVED — Flashbang Grenade — "No Damage/Violence Dealt" But Hit Roll Required
**Files:** `07_WeaponsAndEffects.md` SPEC-19.4  
**Problem:** Flashbang says "No damage/violence dealt." But it still requires a Combo Roll (hit vs Reaction). On miss, no effects apply. On hit... Choc 1 (auto) and Barrage 2 apply, but Dispersion 5 hits splash targets too. Do the Choc/Barrage effects also apply to splash targets on hit? Or only the primary?  
**Fix:** Clarify: "Flashbang on hit: apply Choc 1 (auto) and Barrage 2 to the primary target. Dispersion 5: splash targets are checked vs their own Reaction. On hit: apply Choc 1 (auto) and Barrage 2 to each splash target independently."

---

### M-12: ✅ RESOLVED — IEM Grenade — Parasitage 2 + Dispersion 5
**Files:** `07_WeaponsAndEffects.md` SPEC-19.4  
**Problem:** "No damage/violence dealt. Disables machines." Same friendly fire question as Flashbang — does Parasitage 2 hit allied knights in the Dispersion zone?  
**Fix:** Clarify: "Yes — Dispersion is faction-agnostic. Allied knights in the blast zone are also tested. On hit, Parasitage 2 applies to them (they lose 2 actions). IEM is extremely dangerous to use near allies."

---

### M-13: ✅ RESOLVED (INTENTIONAL) — Ours Corrompu — Point Faible Discovery Only by Narrative Event
**Files:** `08_ContentData.md` SPEC-21, `06_EnemySystem.md` SPEC-18.6  
**Problem:** "Discovery in demo: only random narrative events during mission nodes can reveal it." Node 4 has a 1/6 chance. If the player takes branch 2A instead of 2B, they have exactly ONE chance at Node 4 (1/6 = 16.7%). If they take 2B, they have the Node 4 chance plus any Perception-based clues (but "In-combat Perception-based discovery is NOT available in the demo"). This means the Point Faible is functionally hidden 83% of the time. Is this intended? The boss fight is already extremely hard.  
**Fix:** This is a design call, not a spec issue — but flag it for the team. Consider adding a second discovery opportunity (e.g., Node 3 post-combat, or branch 2B event).

---

### M-14: ✅ RESOLVED (via A-14) — Characteristics Floor at 0 — But What About Aspect Floor?
**Files:** `01_KnightDataModel.md` SPEC-01  
**Problem:** "Characteristics floor at 0." But what about Aspects? Can Aspects go below 2 (the starting value)? Injuries reduce Characteristics but never directly reduce Aspects. However, if all 3 Characteristics in an Aspect reach 0, is the Aspect still > 0? The spec says "Characteristic ≤ parent Aspect" but never addresses Aspect reduction.  
**Fix:** Clarify: "Aspect scores are NOT reduced by injuries (only Characteristics are). Aspects can only change via Camelot training or specific future mechanics. The Characteristic ≤ Aspect constraint only applies during generation and leveling, not retroactively — an injury can reduce a Characteristic below its Aspect score without issue."

---

## 🟣 DUPLICATION Issues (Risk of Desync)

### D-01: ✅ RESOLVED — Squad Selection — Defined in Three Places
**Files:** `01_KnightDataModel.md` SPEC-02, `02_PositionAndTurns.md` SPEC-03, `00_INDEX.md`  
**Problem:** Mission squad selection (choosing 4 of 8 knights) is described in SPEC-02 §Mission Squad Selection, SPEC-03 §Squad Selection (Mission Start), and SPEC-03 §Core Rank Rules. Each version adds slightly different details (UI hints, dead knight handling).  
**Fix:** Designate SPEC-03 as the canonical source (it has the most detail). Add redirect notes in SPEC-02: "See SPEC-03 §Squad Selection for canonical rules."

---

### D-02: ✅ RESOLVED — Hémorragie — Defined in Three Places
**Files:** `04_ArmorAndState.md` SPEC-08, `08_ContentData.md` SPEC-23, `13_NodSystem.md` SPEC-33  
**Problem:** Hémorragie mechanics are described in SPEC-08 §Hémorragie System (full specification), SPEC-23 §23.2 Column 1 Row 5 (injury table entry), and SPEC-33 §33.7 (Nod interaction). While cross-references exist, each file restates key rules (countdown 3→0, permadeath, non-permanent). Any update risks desync.  
**Fix:** SPEC-08 is the canonical source. SPEC-23 and SPEC-33 should say "See SPEC-08 §Hémorragie System for complete rules" and only state the locally-relevant interaction (injury roll triggers it; Nod Soin saves from it).

---

### D-03: ✅ RESOLVED — Fold State — Systems Offline/Active Listed in Two Places
**Files:** `04_ArmorAndState.md` SPEC-10, `13_NodSystem.md` SPEC-33.5  
**Problem:** SPEC-10 says "Nod d'Énergie blocked (PE frozen)." SPEC-33.5 has a dedicated table confirming this. If SPEC-10 is updated, SPEC-33 might not be.  
**Fix:** SPEC-10 is canonical for Fold rules. SPEC-33 should reference SPEC-10 and only add the Nod-specific interaction.

---

### D-04: ✅ RESOLVED — Global Rules — Stated in INDEX and SPEC-03
**Files:** `00_INDEX.md`, `02_PositionAndTurns.md` SPEC-03  
**Problem:** "Duration: 1 turn", "All divisions round down", "Hit threshold: strictly exceed Defense" — these appear in both files.  
**Fix:** INDEX already has them. SPEC-03 should reference INDEX: "See 00_INDEX.md Global Rules."

---

### D-05: ✅ RESOLVED — PA Passthrough — Stated in SPEC-06 and SPEC-10
**Files:** `04_ArmorAndState.md` SPEC-06, SPEC-10  
**Problem:** PA passthrough (1 PS per 5 PA absorbed) is defined in SPEC-06 and re-stated in SPEC-10 (Fold edge cases).  
**Fix:** Single source in SPEC-06. SPEC-10 references: "PA passthrough applies normally per SPEC-06."

---

### D-06: ✅ RESOLVED — Tarot Drawing Rules — Two Locations
**Files:** `01_KnightDataModel.md` SPEC-02 Step 4, `09_TarotSystem.md` SPEC-24.5  
**Problem:** The 5-card draw procedure and advantage/disadvantage selection are described in both files.  
**Fix:** SPEC-24.5 is canonical for Tarot generation integration. SPEC-02 Step 4 should say: "Tarot: See SPEC-24.5 for complete draw and selection procedure."

---

### D-07: ✅ RESOLVED — Action Economy Table — Multiple Versions
**Files:** `03_CombatRolls.md` SPEC-04, `02_PositionAndTurns.md` SPEC-15  
**Problem:** Action costs are listed in both the SPEC-04 Action Economy Reference table and SPEC-15 Turn Structure. The SPEC-04 table is more complete, but SPEC-15 adds Nod-on-ally rules.  
**Fix:** SPEC-04 is canonical for all action costs. SPEC-15 should reference it.

---

### D-08: ✅ RESOLVED — Ours Corrompu Phase 2 — Described in SPEC-14 and SPEC-21.5
**Files:** `06_EnemySystem.md` SPEC-14, `08_ContentData.md` SPEC-21.5  
**Problem:** Phase transition stats and procedure are in both files. If one is updated, the other risks desync.  
**Fix:** SPEC-21.5 is canonical for stat blocks. SPEC-14 is canonical for the transition procedure. Each should reference the other for its non-canonical content.

---

## ⚪ MINOR Issues (Naming / Formatting)

### N-01: ✅ RESOLVED — Inconsistent French/English Naming
**Problem:** Some fields use French (hémorragieCountdown, Héroïsme) while others use English (isDead, activeStyle). The `CombatStyle` enum uses French for some values (AGRESSIF, DEFENSIF, MISE_A_COUVERT) and English for others (STANDARD).  
**Fix:** Establish a convention in SPEC-32: "Enum values use French names from the tabletop (AGRESSIF, not AGGRESSIVE). Field names use English (isInAgony, not estEnAgonie). Display names use French."

---

### N-02: ✅ INTENTIONAL — "Colonne Vertébrale Brisée" — Inconsistent Location References
**Problem:** Referenced in SPEC-03, SPEC-08, and SPEC-23. SPEC-03 says "See SPEC-23 Column 3, Row 1." But Column 3 is Moderate (D6 = 4-5). Row 1 of Column 3 is indeed Colonne Vertébrale Brisée. Correct, but the injury is Severity "Moderate" yet mechanically devastating (movement doubled). Worth flagging for balance review.  
**Fix:** No spec fix needed, but note for design review: a "Moderate" injury that doubles movement cost is arguably Severe.

---

### N-03: ✅ RESOLVED — SPEC-21.1 Nocte — Aspects Listed But Never Used for Attacks
**Problem:** Nocte has "Chair 4 / Bête 9 / Machine 1 / Dame 0 / Masque 4" but "No attacks, no hit rolls. Débordement only." The aspects are purely for the data model. No agent needs them for gameplay. Could confuse the Enemy AI Agent.  
**Fix:** Add note to Nocte: "Aspects are included for data completeness. Bandes do not use aspects for attacks or defense — see SPEC-11."

---

### N-04: ✅ RESOLVED — SPEC-18.8 Actions Multiples Table — Inconsistent "None" vs "(1)"
**Problem:** The table uses "None (standard)" for the default case. The EnemyBase schema uses `combatActionsRemaining` as an int. It should be clearer that "None" = 0 extra Combat Actions, "(1)" = 1 extra, etc.  
**Fix:** Change the table header to "Extra Combat Actions" and use 0/1/2 instead of None/(1)/(2).

---

### N-05: ✅ RESOLVED (via B-02) — SPEC-21.5 Ours Phase 2 Table — Missing AI Primary/Secondary
**Problem:** Phase 2 stat table has `AI change` row with empty value. Related to B-02 but also a formatting issue — the table should have `AI: Primary/Secondary` rows like other stat blocks.  
**Fix:** Add `AI: Primary/Secondary` rows to the Phase 2 table with explicit values.

---

## 💡 SUGGESTIONS (Non-Blocking)

### S-01: ✅ RESOLVED — Add a "Demo Scope" Flag to Every Weapon Effect
**Recommendation:** Several effects (En Chaîne, Fureur, Assassin, Démoralisant, Cadence, Lourd → Puissant style) are defined but have no demo weapon using them. Without a clear "IN_DEMO: yes/no" flag, agents will implement all 30+ effects, wasting significant effort.  
**Implementation:** Add a column to the SPEC-20 effects tables: `Demo Scope: YES / SCAFFOLDED`. Agents implement YES effects fully and create stubs for SCAFFOLDED ones.

---

### S-02: ✅ RESOLVED — Add Explicit "Test Scenario" Specifications for Integration Testing
**Recommendation:** The acceptance criteria are excellent for unit tests but lack end-to-end scenarios. For example: "Full 3-turn combat: K1 attacks E1, E1 uses Charge Brutale, K2 uses Nod on K1, K3 enters Ghost..." This would catch interaction bugs that isolated AC tests miss.  
**Implementation:** Add a `15_IntegrationTests.md` file with 3–5 full combat scenarios covering the most complex interactions (Fold+Despair+Hémorragie, Ghost+Ambidextre+Exploit, Barrage+Puissant, Boss Phase transition mid-turn).

---

## Summary Action Items

| Priority | Count | Status |
|----------|-------|--------|
| 🔴 BLOCKERs | 8 | ✅ ALL RESOLVED |
| 🟠 CONTRADICTIONs | 9 | ✅ ALL RESOLVED |
| 🟡 AMBIGUITIEs | 16 | ✅ ALL RESOLVED (6 scaffolded for post-demo) |
| 🔵 MISSINGs | 14 | ✅ ALL RESOLVED (1 intentional, 1 via A-06, 1 via A-14) |
| 🟣 DUPLICATIONs | 8 | ✅ ALL RESOLVED |
| ⚪ MINORs | 5 | ✅ ALL RESOLVED (1 intentional) |
| 💡 SUGGESTIONs | 2 | ✅ ALL RESOLVED |

**All 62 issues resolved. Specification is agent-ready.**

**Scaffolded items requiring future design passes:**
- Mode Héroïque objective system (A-09)
- En Chaîne effect (A-05)
- Assassin X / surprise mechanics (A-06 / M-02)
- Désignation full mechanic (A-07)
- Démoralisant effect (A-08)
- Cadence X effect (B-08)
- Shrine entry restriction (A-13)
- Point Faible discovery paths (M-13 — intentional, future expansion planned)

---

*End of Deep-Dive Review v2. All issues reference specific file paths and section headers for traceability.*

