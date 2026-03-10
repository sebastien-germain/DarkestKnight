# DarkestKnight — Specification Deep Dive Review v3
**Date:** March 10, 2026  
**Reviewer:** Expert Game Designer (20+ yrs 2D RPG / Unity)  
**Scope:** Full review of `/TechSpec/specifications/` (files 00–15) for missing, duplicated, confusing, or contradictory information.  
**Goal:** Make the specification unambiguous enough for an autonomous agent team to implement without human clarification.

---

## Executive Summary

The specification suite is **remarkably thorough** — one of the most detailed tabletop-to-digital adaptations I've reviewed. The split-file architecture with dependency headers, canonical source markers, and per-SPEC acceptance criteria is excellent for agent workflows. However, I identified **47 issues** across 5 severity levels. Most are clarification gaps or minor inconsistencies, not fundamental design flaws.

| Severity | Count | Description |
|----------|-------|-------------|
| 🔴 CRITICAL | 5 | ✅ ALL FIXED — Contradictions or missing rules that WILL cause agent divergence |
| 🟠 HIGH | 12 | ✅ ALL FIXED — Ambiguities likely to cause implementation inconsistencies |
| 🟡 MEDIUM | 15 | ✅ ALL FIXED — Missing details that need clarification but have reasonable defaults |
| 🔵 LOW | 10 | ✅ ALL FIXED — Duplicated or confusing text that could slow agents down |
| ⚪ NIT | 5 | ✅ ALL FIXED — Cosmetic/organizational suggestions |

---

## 🔴 CRITICAL Issues (Will cause agent divergence)

> **STATUS: ALL 5 CRITICAL ISSUES FIXED** (2026-03-10)

### C-01: ✅ FIXED — Perce Armure — Conflicting Pseudocode vs Description
**Files:** `04_ArmorAndState.md` (SPEC-06), `07_WeaponsAndEffects.md` (SPEC-20)
**Resolution:** Perce Armure **caps PA absorption** — X points of PA are "pierced" and cannot absorb damage. `target.PA` only decreases by the amount actually absorbed (`paAbsorbed`), never by X itself. Pseudocode comments clarified, Tag Overrides table reworded, SPEC-20 acceptance criteria corrected, and two new worked examples added (Case A: X ≥ PA, Case B: X < PA) with full PA tracking.
**Problem:** The SPEC-06 pseudocode for `Perce Armure X` in the ArmourLayerResolver reduces the **effective PA** by X before absorption:
```python
effectivePA = max(target.PA - X, 0)
paAbsorbed = min(effectivePA, remaining)
```
This means Perce Armure 40 vs PA 60 → effectivePA = 20, damage absorbed up to 20, then remainder hits PS. But `target.PA` itself is NOT reduced by the X — only the absorption is capped. However, the SPEC-20 acceptance criteria says:
> "Perce Armure 40 vs PA 30: PA fully ignored, damage hits PS."

This implies the X is subtracted from PA for hit-routing purposes. But the pseudocode does `paAbsorbed = min(effectivePA, remaining)` — so if effectivePA = 0, **no PA is absorbed**, and `target.PA` remains unchanged (not reduced). The actual PA value on the target should still decrease. The Marteau-Épieu has Perce Armure 40, and several acceptance criteria assume it works — but the pseudocode would leave target.PA at 30 while routing damage to PS. **Does target.PA actually decrease?**
**Impact:** Every agent implementing the ArmourLayerResolver will interpret this differently.
**Recommendation:** Clarify: does Perce Armure reduce the PA pool permanently for this hit (i.e., `target.PA -= X` first), or does it only cap how much PA can absorb (leaving target.PA unchanged, bypassing X points to route to PS)? The pseudocode implies the latter but the text implies the former. Add a worked example with PA tracking.

### C-02: ✅ FIXED — Despair Duration — Roll Timing Not Specified
**Files:** `04_ArmorAndState.md` (SPEC-07)
**Resolution:** Added explicit line: "Immediately on Despair entry, roll 1D6 to determine despairDuration. This roll happens once — the result is stored and decremented each turn. It is NOT re-rolled per turn."
**Problem:** Despair lasts `despairDuration` turns (1D6), but when is this 1D6 rolled? The spec says:
> "Fire OnDespairEntered(knight, despairDuration) event"

This implies the duration is known at trigger time — so the 1D6 is rolled immediately on Despair entry. But it's never explicitly stated. An agent could reasonably roll it at the start of each Despaired turn to check "still Despaired?" 
**Impact:** Moderate — most agents will assume immediate roll, but without explicit wording, the event `OnDespairEntered(knight, duration)` could be ambiguous.
**Recommendation:** Add explicit line: "On Despair entry, immediately roll 1D6 to determine despairDuration (number of turns the knight acts under hostile AI)."

### C-03: ✅ FIXED — BandeController Data Model — ExecuteBandeTurn vs OnTurnEnd Duplication
**Files:** `06_EnemySystem.md` (SPEC-11)
**Resolution:** Removed `OnTurnEnd()` from BandeController. `ExecuteBandeTurn()` is the sole canonical method — it deals damage then increments the counter.
**Problem:** The BandeController code has BOTH:
- `ExecuteBandeTurn()` which calls `debordementTurn++` at the end.
- `OnTurnEnd()` which also calls `debordementTurn++`.

If both are called, the counter increments TWICE per turn. The intent seems to be that `ExecuteBandeTurn()` is the only thing called at the Bande's initiative, and `OnTurnEnd()` is dead code. But an agent wiring up events might connect both.
**Impact:** Double-increment would cause Débordement to escalate at 2× speed — game-breaking.
**Recommendation:** Remove `OnTurnEnd()` or add a comment that it's redundant with `ExecuteBandeTurn()`. The canonical pattern should be: deal damage → increment → done.

### C-04: ✅ FIXED — Ghost Deactivation Timing — "After First Attack Resolves" vs Turn Duration
**Files:** `05_ArmorAbilities.md` (SPEC-17.4), `15_IntegrationTests.md`
**Resolution:** Clarified in SPEC-17.4: the "Duration" field now explains that 1 turn is the PE maintenance billing period, NOT a guaranteed stealth window. Ghost deactivates **immediately** after the first attack resolves. Fixed Integration Test Scenario 2 to say "Ghost deactivated" instead of "Ghost still active."
**Problem:** Two conflicting duration statements:
1. §Ghost Activation: "Duration: 1 turn combat" — implies Ghost persists for the whole turn.
2. §Attack from Ghost: "Ghost deactivation: Immediate after the first attack resolves" — implies Ghost drops mid-turn.

Integration Test Scenario 2 says after the Rogue's Ambidextre attack: "Ghost still active (1 turn duration, deactivates at start of K3's next turn)." But SPEC-17.4 §Attack from Ghost §Scope says: "Ghost deactivates immediately after the first attack resolves."

These are contradictory. If Ghost deactivates immediately after the first attack, it cannot also persist until the start of the next turn. The Ambidextre AC ("2nd attack proceeds without Ghost") is consistent with immediate deactivation. Scenario 2's comment about Ghost being "still active" after the attack sequence is wrong if the Rogue attacked.
**Impact:** An agent implementing Ghost stealth-vs-detection will check Ghost status between enemy actions. If Ghost drops mid-turn (after attack), enemies acting later in the same round CAN target the Rogue. If it persists until next turn start, they can't.
**Recommendation:** Clarify: Ghost deactivates **immediately after the first attack resolves** (mid-turn), consistent with the Scope section and Ambidextre AC. Fix Scenario 2 final comment accordingly. Enemies later in the initiative can target the Rogue after their Ghost attack.

### C-05: ✅ FIXED — Bestian Flot de Ténèbres — Choc 1 on Anathème Weapon, Missing Specification
**Files:** `07_WeaponsAndEffects.md` (SPEC-20), `15_IntegrationTests.md`
**Resolution:** (1) Added explicit default rule in SPEC-20 Choc definition: "Choc is standard unless the weapon/grenade explicitly specifies `isAutoChoc = true`." Listed all demo enemy weapons as standard Choc. Only Flashbang grenade is auto-Choc. (2) Confirmed Choc applies normally on Anathème attacks — Anathème only changes damage routing, not hit effects. (3) Fixed Integration Test Scenario 4 which incorrectly said "Choc 1 (auto on Flot de ténèbres)" → now says standard Choc with threshold check.
**Problem:** The Bestian's Flot de Ténèbres has "Choc 1 + Anathème." Anathème damage bypasses PA and hits PEs directly (SPEC-06 §Knight Defense vs Anathème). But Choc 1 is applied on the **attack roll** (standard Choc: successes > floor(Chair/2)). The attack roll is against **Reaction** (ranged). The question: does Choc 1 apply to the target knight even though damage hits PEs instead of PS? The spec never addresses this. Choc is a post-hit effect (SPEC-20 §20.6 Resolution Order), and Anathème only changes the damage routing — the attack roll and hit determination are standard. So Choc 1 should apply normally as long as the attack hits. But this is never stated, and Integration Test Scenario 4 mentions "Choc 1 (auto on Flot de ténèbres)" — the stat block says Choc 1, not "auto-Choc 1." The `isAutoChoc` flag is not specified for enemy weapons.
**Impact:** Agent implementing enemy weapons won't know if Choc is standard or auto for enemy stat blocks.
**Recommendation:** 
1. Specify whether enemy Choc effects are standard or auto (add `isAutoChoc` to enemy weapon data or specify per-weapon).
2. Confirm that Choc effects apply normally on Anathème attacks (they apply on hit, regardless of damage routing).

---

## 🟠 HIGH Issues (Likely agent inconsistencies)

> **STATUS: ALL 12 HIGH ISSUES FIXED** (2026-03-10)

### H-01: ✅ FIXED — Grenade Force OD Range Extension — Undefined Mechanic
**Files:** `07_WeaponsAndEffects.md` (SPEC-19.4)
**Resolution:** Added formula: `grenadeMaxGap = 2 + forceODLevel`. Base Courte (gap 2) + 1 per Force OD level. Passive benefit, does not consume OD.

### H-02: ✅ FIXED — Warrior Type Selection at Generation — "3 chosen from 5" by Whom?
**Files:** `05_ArmorAbilities.md` (SPEC-17.1)
**Resolution:** Reworded to: "3 from 5 (demo: Soldier + Hunter + 1 random from {Scholar, Herald, Scout}). Full game: player chooses 3."

### H-03: ✅ FIXED — Watchtower Reaction Halving — Unclear Interaction with Barrage
**Files:** `05_ArmorAbilities.md` (SPEC-17.2)
**Resolution:** Added explicit modifier application order per Knight tabletop rule (Division/Multiplication always before Addition/Subtraction): (1) halve base Reaction, (2) rank modifiers, (3) cover modifiers, (4) style modifiers, (5) Barrage debuffs (last, as external subtractive effect). All results floor at 0. Added worked example with Watchtower + Barrage interaction.

### H-04: ✅ FIXED — Nod de Soin on Agony Knight — Does It Also Heal Priest Mechanic?
**Files:** `05_ArmorAbilities.md` (SPEC-17.3)
**Resolution:** Added explicit Agony targeting rule to Priest Mechanic: "Priest Mechanic cannot target a knight in Agony (PS = 0). Only Nod de Soin can target Agony knights. Mechanic CAN target Folded knights."

### H-05: ✅ FIXED — Enemy Aspect Selection for Attack — "Defaults" vs Stat Block Specification
**Files:** `06_EnemySystem.md` (SPEC-18.9 Step 2)
**Resolution:** Attack Aspect marked as REQUIRED on every enemy weapon profile. Default fallback is for validation/error recovery only. Agents authoring new enemies must always include the field.

### H-06: ✅ FIXED — Motivation MN-01 "Protéger un allié" — Ambiguous Trigger
**Files:** `08_ContentData.md` (SPEC-12.1)
**Resolution:** Reworded trigger: "Allied knight with currentPS ≤ 10% maxPS is hit AND attack deals 0 PS damage (fully absorbed by CdF/PA/Bouclier) AND motivated knight is within 1 rank of hit ally."

### H-07: ✅ FIXED — Tarot Draw Weighting — "3× weight for primary aspects" Undefined
**Files:** `01_KnightDataModel.md` (SPEC-02 Step 4)
**Resolution:** Added full pseudocode: primary aspect cards get weight 3, others get weight 1, Le Fou/La Maison-Dieu (no linked Aspect) get weight 1. Weighted random without replacement.

### H-08: ✅ FIXED — Barrage Expiry — "Start of Suppressor's Next Turn" Ambiguity
**Files:** `07_WeaponsAndEffects.md` (SPEC-20.3)
**Resolution:** Added explicit expiry rule: "If the suppressor dies or enters permanent incapacitation, all Barrage debuffs from that source expire immediately. Temporary incapacitation does NOT expire Barrage — the turn is skipped but expiry event still fires at the suppressor's initiative slot."

### H-09: ✅ FIXED — Defense/Reaction Floor — Missing Global Rule
**Files:** `00_INDEX.md`, `14_Constants.md`
**Resolution:** Added to Global Rules: "Defense and Reaction values floor at 0 after all modifiers. Negative values are clamped to 0." Added to Constants table with Math operation order (Division/Multiplication before Addition/Subtraction).

### H-10: ✅ FIXED — Cerveau Touché (Injury [1,3]) — "Lose 1 action per turn" Unclear
**Files:** `08_ContentData.md` (SPEC-23.2)
**Resolution:** Clarified: "Lose 1 standard action per turn (knight chooses which: Combat OR Movement). Bonus actions from other sources (e.g., Mode Héroïque, Watchtower extra ranged action) are unaffected."

### H-11: ✅ FIXED — Shrine CdF — Stacks with Base or Replaces?
**Files:** `04_ArmorAndState.md` (SPEC-10)
**Resolution:** Clarified in Guardian Suit Stats table: "CdF 5 replaces armor base CdF only. External CdF bonuses (Shrine +6) still apply additively. Folded knight at Shrine rank: total CdF = 5 + 6 = 11. IGNORE_CDF bypasses the total."

### H-12: ✅ FIXED — Dispersion — Primary Target Miss Still Causes Splash?
**Files:** `07_WeaponsAndEffects.md` (SPEC-20.5)
**Resolution:** Implemented Option A: "Splash fires regardless of primary hit or miss. The explosion happens at the target location whether or not the primary was hit. A damage roll is still performed on primary miss for splash resolution. Splash targets are evaluated normally against their individual thresholds."

---

## 🟡 MEDIUM Issues (Need clarification, reasonable defaults exist)

> **STATUS: ALL 15 MEDIUM ISSUES FIXED** (2026-03-10)

### M-01: ✅ FIXED — Pilonnage — Violence Bonus Too?
**Files:** `03_CombatRolls.md` (SPEC-04)
**Resolution:** Added clarification: "For each accumulated pilonnageTurn, add 1 bonus D6 to both the damage dice pool AND the violence dice pool for that attack. Bonus dice are rolled independently for each pool."

### M-02: ✅ FIXED — Enemy Spawning Initiative — "Can Act on Turn They Spawn"
**Files:** `02_PositionAndTurns.md` (SPEC-03)
**Resolution:** Added: "Reinforcements roll initiative on spawn per SPEC-15. Inserted into current round's queue at rolled value. If initiative slot already passed this round, they wait until next round."

### M-03: ✅ FIXED — Dégâts Continus — "Cannot Stack with Itself" Across Sources?
**Files:** `07_WeaponsAndEffects.md` (SPEC-20.1)
**Resolution:** Option B: New application **replaces** existing DC. Use higher X value (max of old/new), roll new 1D6 duration. Only one DC effect active per target at any time.

### M-04: ✅ FIXED — Rogue Ghost Between Encounters
**Files:** `12_ArchitectureGuardrails.md` (SPEC-32), `05_ArmorAbilities.md` (SPEC-17.4)
**Resolution:** Clarified in both SPEC-32 and Ghost Interactions: "Ghost is disabled at combat end. `isGhostActive` resets to `false` at each combat start. No stealth checks exist outside combat in demo."

### M-05: ✅ FIXED — Tarot Card Le Fou (0) — +6 Char Points "Free" Distribution
**Files:** `09_TarotSystem.md` (SPEC-24.4)
**Resolution:** Clarified: "Use the full 15-characteristic column for the knight's armor class. Normalize weights across all 15 (total = 43). Each point rolled independently (probability = weight/43). Caps at parent Aspect score still apply."

### M-06: ✅ FIXED — maxPEs Floor for Implants — What About Prisonnier?
**Files:** `10_MissionAndHub.md` (SPEC-25.2)
**Resolution:** Added: "Prisonnier disadvantage does not prevent implant installation (floor applies to maxPEs, not PEs recovery). UI displays additional warning for Prisonnier knights."

### M-07: ✅ FIXED — Charge Brutale Cooldown Reset on Phase Transition — Only Ours?
**Files:** `06_EnemySystem.md` (SPEC-18.13)
**Resolution:** Generalized: "Charge Brutale cooldown resets on any boss phase transition. This is a generic multi-phase boss rule, not specific to Ours Corrompu."

### M-08: ✅ FIXED — "Points de Contact" — Scaffolded but Computed at Generation
**Files:** `01_KnightDataModel.md` (SPEC-01)
**Resolution:** Changed instruction to: "Do NOT compute or store during generation. Formula preserved for future reference only."

### M-09: ✅ FIXED — Healing at Camelot — "PS/PA/PE Fully Restored to Max"
**Files:** `10_MissionAndHub.md` (SPEC-25.5)
**Resolution:** Added callout box: "PE (Points d'Énergie, energy) is fully restored at Camelot. PEs (Points d'Espoir, morale) is NOT restored at Camelot in demo."

### M-10: ✅ FIXED — Mode Héroïque — "All allies Adjuvants" Without Combo Details
**Files:** `03_CombatRolls.md` (SPEC-09)
**Resolution:** Added full specification: "All alive/controllable allies auto-contribute as Assistants (no action/PE cost). Each uses highest-scoring unused Characteristic. PEs penalty applies. Max 3 assistants rule applies. Egoiste knights excluded."

### M-11: ✅ FIXED — Bestian Flot de Ténèbres — Is It Ranged?
**Files:** `08_ContentData.md` (SPEC-21.2)
**Resolution:** Reworded: "AI prioritizes this weapon to deal Anathème PEs damage (CdF → PEs route). The attack roll is standard — successes must exceed target's Reaction at Moyenne range."

### M-12: ✅ FIXED — Reinforcement Spawn Priority — Tier-Based
**Files:** `02_PositionAndTurns.md` (SPEC-03)
**Resolution:** Clarified: "Spawn order follows `enemyIdsInOrder` list from EncounterSO. Tier-based priority is used ONLY when multiple reinforcements spawn simultaneously into multiple empty ranks."

### M-13: ✅ FIXED — Tarot L'Impératrice Disadvantage — "Esprit de Contradiction" vs Assist
**Files:** `09_TarotSystem.md` (SPEC-24.3)
**Resolution:** Added: "+1 difficulty applies to affected ally's next Combo Roll regardless of type. For Assist rolls: no threshold applies, so debuff is wasted — but IS consumed. Ally's next roll after is debuff-free."

### M-14: ✅ FIXED — Shrine Maintenance Failure — Timing Within Turn
**Files:** `05_ArmorAbilities.md` (SPEC-17.2)
**Resolution:** Fixed PEs→PE typo in Shrine Disable Conditions. Now reads: "Paladin's PE reaches 0: no energy to maintain → Shrine disabled." Added PE=0 maintenance-specific condition. Despair (PEs=0) listed separately as its own condition.

### M-15: ✅ FIXED — NarrativeEvent Schema — Incomplete for Demo Events
**Files:** `10_MissionAndHub.md` (SPEC-22.3)
**Resolution:** Replaced simple table with 4 explicit NarrativeEvent struct instances following the schema. Each event includes eventId, nodeIndex, triggerType, description, rollThreshold, and complete NarrativeEffect arrays. Quick reference table preserved below.

---

## 🔵 LOW Issues (Duplicated or confusing text)

> **STATUS: ALL 10 LOW ISSUES FIXED** (2026-03-10)

### L-01: ✅ FIXED — Squad Selection Defined in 3 Places
**Files:** `01_KnightDataModel.md` (SPEC-02)
**Resolution:** Replaced SPEC-02 squad selection content with cross-reference: "See SPEC-03 §Squad Selection (canonical). Do not duplicate rules here."

### L-02: ✅ FIXED — "Successive Doubling = Tripling" — No Example
**Files:** `00_INDEX.md`
**Resolution:** Added example with clarification: "No demo mechanic currently produces successive doubling — this rule exists as a global safeguard for future content where multiple multiplicative effects may stack."

### L-03: ✅ FIXED — Blason Table Says "10 Blasons listed, 9 active" but Lists 10
**Files:** `01_KnightDataModel.md` (SPEC-02.5)
**Resolution:** Reworded to: "Demo Blason Table (10 entries, 9 active — Le Dragon #4 is deferred)."

### L-04: ✅ FIXED — Injury Table — Severity Die Mapping Mismatch
**Files:** `04_ArmorAndState.md` (SPEC-08)
**Resolution:** Added explicit column references: e.g., "[4,1] = Column 3 (Moderate, severity die 4), Row 1: Colonne Vertébrale Brisée."

### L-05: ✅ FIXED — Characteristics[15] in KnightBase — Array vs Named Fields
**Files:** `01_KnightDataModel.md` (SPEC-01)
**Resolution:** Added note: "Access via CharacteristicId enum index. Named aliases (e.g., knight.Combat) are optional convenience getters. Aspects above are separate named fields because there are only 5."

### L-06: ✅ FIXED — "PG" Abbreviation Used Without Definition
**Files:** `00_INDEX.md`
**Resolution:** Added PG definition to Global Rules: "PG (Points de Gloire): Meta-currency earned through play, spent at Camelot Hub (SPEC-25)."

### L-07: ✅ FIXED — Encounter 2A Has Nocte Bande But Mission Graph Says "2 Bestians + 1 Nocte bande"
**Files:** `10_MissionAndHub.md` (SPEC-22.2)
**Resolution:** Added explicit combat end condition: "A combat node ends when ALL enemies are eliminated — including on-track individuals (PS = 0) AND off-track bandes (Cohésion = 0). A bande with Cohésion > 0 keeps combat going."

### L-08: ✅ FIXED — Ours Corrompu Phase 2 — Régénération Fires Before or After Nocte Bande Turn?
**Files:** `06_EnemySystem.md` (SPEC-14)
**Resolution:** Added safeguard: "Régénération requires Cohésion > 50 (not ≥ 50). At Cohésion = 50, does not fire — boss won't sacrifice last of its bande." Added timing note: Régénération fires at init 6 (boss turn), before Nocte Débordement at init 1.

### L-09: ✅ FIXED — Integration Test Scenario 2 — Ghost Duration Comment Error
**Files:** `15_IntegrationTests.md`
**Resolution:** Already fixed during C-04 critical fix. Text already reads "Ghost deactivated (immediate deactivation after first attack per SPEC-17.4 §Scope)."

### L-10: ✅ FIXED — SPEC-14 vs SPEC-21.5 — Dual Canonical Sources for Boss
**Files:** `06_EnemySystem.md` (SPEC-14)
**Resolution:** Removed inline stat values from SPEC-14 transition procedure. Now references SPEC-21.5: "Apply Phase 2 stat overrides from SPEC-21.5 (canonical for all stat values). Do NOT duplicate stat values here."

---

## ⚪ NIT (Cosmetic/organizational)

> **STATUS: ALL 5 NIT ISSUES FIXED** (2026-03-10)

### N-01: ✅ FIXED — 14_Constants.md — Missing MotivationType Enum
**Files:** `14_Constants.md`
**Resolution:** Added `MotivationType { MINOR, MAJOR, VOEU }` enum with descriptions for each type.

### N-02: ✅ FIXED — 14_Constants.md — Missing Seigneur Enum
**Files:** `14_Constants.md`
**Resolution:** Added `SeigneurId { LA_BETE, LA_MACHINE, LA_CHAIR, LA_DAME, LE_MASQUE }` enum. Maps to EnemyBase.seigneur field.

### N-03: ✅ FIXED — SPEC-24 Tarot Table — Cards Not in Numerical Order
**Files:** `09_TarotSystem.md` (SPEC-24.4)
**Resolution:** Sorted all 22 cards in numerical order (0, I, II, ... XXI). Removed `[v7]` version tags from the table rows.

### N-04: ✅ FIXED — Version Tags Inconsistent
**Files:** `00_INDEX.md`
**Resolution:** Added guidance note to INDEX: "Version tags are historical annotations. Agents should ignore them — treat ALL current text as authoritative. A future revision will consolidate tags into a changelog."

### N-05: ✅ FIXED — Missing Enum — DamageTag for Weapon Effect Overrides
**Files:** `14_Constants.md`
**Resolution:** Added note under WeaponEffectId: "Weapon effect tags in ArmourLayerResolver pseudocode map directly to WeaponEffectId. No separate DamageTag enum needed."

---

## Summary of Highest-Priority Fixes

| Priority | Issue | Status |
|----------|-------|--------|
| 🔴 C-01 | Perce Armure pseudocode vs description conflict | ✅ FIXED |
| 🔴 C-02 | Despair duration roll timing | ✅ FIXED |
| 🔴 C-03 | BandeController double-increment | ✅ FIXED |
| 🔴 C-04 | Ghost deactivation timing contradiction | ✅ FIXED |
| 🔴 C-05 | Choc on Anathème weapons — standard or auto? | ✅ FIXED |
| 🟠 H-01 | Grenade Force OD range extension undefined | ✅ FIXED |
| 🟠 H-02 | Warrior Type selection wording | ✅ FIXED |
| 🟠 H-03 | Watchtower Reaction + Barrage interaction | ✅ FIXED |
| 🟠 H-04 | Priest Mechanic Agony targeting | ✅ FIXED |
| 🟠 H-05 | Enemy Attack Aspect required field | ✅ FIXED |
| 🟠 H-06 | Motivation MN-01 ambiguous trigger | ✅ FIXED |
| 🟠 H-07 | Tarot draw weighting undefined | ✅ FIXED |
| 🟠 H-08 | Barrage expiry on suppressor death | ✅ FIXED |
| 🟠 H-09 | Defense/Reaction floor missing | ✅ FIXED |
| 🟠 H-10 | Cerveau Touché action loss unclear | ✅ FIXED |
| 🟠 H-11 | Shrine CdF in Fold state | ✅ FIXED |
| 🟠 H-12 | Dispersion on primary miss | ✅ FIXED |
| 🟡 M-01 | Pilonnage violence bonus clarification | ✅ FIXED |
| 🟡 M-02 | Reinforcement initiative on spawn | ✅ FIXED |
| 🟡 M-03 | Dégâts Continus stacking rule (replace) | ✅ FIXED |
| 🟡 M-04 | Ghost between encounters | ✅ FIXED |
| 🟡 M-05 | Le Fou distribution across all 15 chars | ✅ FIXED |
| 🟡 M-06 | Prisonnier + Implants interaction | ✅ FIXED |
| 🟡 M-07 | Charge Brutale phase-reset generalized | ✅ FIXED |
| 🟡 M-08 | Points de Contact — don't compute in demo | ✅ FIXED |
| 🟡 M-09 | PE vs PEs at Camelot callout | ✅ FIXED |
| 🟡 M-10 | Mode Héroïque auto-contribution details | ✅ FIXED |
| 🟡 M-11 | Bestian Flot de Ténèbres attack roll clarity | ✅ FIXED |
| 🟡 M-12 | Reinforcement spawn priority vs queue | ✅ FIXED |
| 🟡 M-13 | Esprit de Contradiction vs Assist | ✅ FIXED |
| 🟡 M-14 | Shrine PEs→PE typo + PE=0 disable | ✅ FIXED |
| 🟡 M-15 | NarrativeEvent schema instances | ✅ FIXED |
| 🔵 L-01 | Squad Selection duplicate removed | ✅ FIXED |
| 🔵 L-02 | Successive Doubling example added | ✅ FIXED |
| 🔵 L-03 | Blason table phrasing clarified | ✅ FIXED |
| 🔵 L-04 | Injury table column references added | ✅ FIXED |
| 🔵 L-05 | Characteristics array access note | ✅ FIXED |
| 🔵 L-06 | PG abbreviation defined | ✅ FIXED |
| 🔵 L-07 | Combat end requires bande killed too | ✅ FIXED |
| 🔵 L-08 | Régénération safeguard + timing | ✅ FIXED |
| 🔵 L-09 | Ghost duration comment (already fixed) | ✅ FIXED |
| 🔵 L-10 | SPEC-14 stat dedup → SPEC-21.5 ref | ✅ FIXED |
| ⚪ N-01 | MotivationType enum added | ✅ FIXED |
| ⚪ N-02 | SeigneurId enum added | ✅ FIXED |
| ⚪ N-03 | Tarot table sorted numerically | ✅ FIXED |
| ⚪ N-04 | Version tags guidance note | ✅ FIXED |
| ⚪ N-05 | WeaponEffectId = DamageTag note | ✅ FIXED |

---

## Positive Observations

1. **Canonical source markers** (e.g., "this file is canonical for X") are excellent for preventing desync — most specs use them consistently.
2. **Acceptance criteria** on every SPEC are well-written and cover edge cases. Integration Test Scenario file is a great addition.
3. **Pre-baked vs Runtime distinction** for enemy stats (SPEC-18.3) is a critical disambiguation that many specs would miss.
4. **RunState vs CombatSession boundary** (SPEC-32) is the kind of architecture documentation that prevents the most common agent divergence issue.
5. **Worked examples** throughout (especially the Barrage tactical example in SPEC-20) are extremely valuable for agent training.
6. **Demo scope flags** (YES/SCAFFOLDED) in SPEC-20 are a great prioritization tool.
7. **Agent Task Boundaries** table in the Index is well-designed for parallel agent work.

---

*End of review. Recommended next step: address all 🔴 CRITICAL issues before agent handoff, then 🟠 HIGH issues. 🟡 MEDIUM and below can be resolved during implementation with agent-flagged questions.*







