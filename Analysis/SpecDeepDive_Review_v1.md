# DarkestKnight — Specification Deep-Dive Analysis
**Reviewer:** Expert Game Designer (20+ years, 2D RPG / Unity)  
**Date:** 2026-03-04 (last updated: 2026-03-09)  
**Scope:** All 14 specification files (`00_INDEX` → `13_NodSystem`)  
**Goal:** Identify missing, duplicated, contradictory, or ambiguous information that would block autonomous agent implementation.

---

## Executive Summary

The specification suite is impressively detailed — far above average for an indie project. The tabletop-to-digital translation is handled carefully, and the modular split is well-structured for parallel agent work. Initial analysis found **52 issues** across 6 severity categories. After collaborative review sessions, **all 52 issues are now resolved** with spec changes applied.

| Severity | Count | Resolved | Remaining | Description |
|----------|-------|----------|-----------|-------------|
| 🔴 BLOCKER | 8 | ✅ 8 | 0 | Agent cannot implement without guessing; will cause divergent builds |
| 🟠 HIGH | 14 | ✅ 14 | 0 | Ambiguity likely to produce bugs or contradictory implementations |
| 🟡 MEDIUM | 16 | ✅ 16 | 0 | Missing detail that an experienced dev could infer, but agents may not |
| 🔵 LOW | 9 | ✅ 9 | 0 | Minor inconsistencies, duplications, or cosmetic issues |
| ⚪ DUPLICATE | 3 | ✅ 3 | 0 | Redundant definitions that risk desync across files |
| 💡 SUGGESTION | 2 | ✅ 2 | 0 | Quality-of-life improvements for agent consumption |

**Status: ✅ ALL 52 ISSUES RESOLVED. Specifications are fully ready for autonomous agent handoff.**

---

## 🔴 BLOCKER Issues (8)

### B-01: ✅ RESOLVED — Nod System — Missing Core Mechanic Definition
**Resolution:** Created [SPEC-33: Nod System](../TechSpec/specifications/13_NodSystem.md) — full specification covering all 3 Nod types, usage rules (combat/outside), Fold/Agony/Hémorragie interactions, Tarot modifiers, edge cases, data model, pseudocode, and 10 acceptance criteria. Cross-references added to 00_INDEX, 01_KnightDataModel, 02_PositionAndTurns, 03_CombatRolls, 04_ArmorAndState, 10_MissionAndHub, 11_UIAndPresentation, 12_ArchitectureGuardrails.

**Key design decisions made:**
- Nod d'Énergie → PE only (NOT PEs). PEs recovery remains Motivation/Exploit/narrative only.
- Nod de Soin CAN target Agony allies (exception to general "not in Agony" rule). This is required for Hémorragie saves and VOEU_OURS.
- Nod d'Énergie BLOCKED in Fold state (PE frozen). Soin and Armure usable.
- Guérison Rapide bonus (+3) belongs to the USER, not the target.
- No Combo Roll — Nod use is automatic.

**Files:** `03_CombatRolls.md` (SPEC-04), `04_ArmorAndState.md` (SPEC-10), `10_MissionAndHub.md` (SPEC-25)  
**Problem:** Nods are referenced **extensively** (Nod Soin, Nod d'Armure, Nod d'Énergie) as a core healing/recovery mechanic across the entire game, but there is **no SPEC defining the Nod system itself**. The only details are scattered fragments:
- Action economy table says "Restores 3D6 of PS/PA/PE" (SPEC-04)
- Fold state says "Nod d'Armure is the primary exit path" (SPEC-10)
- Camelot says "3 Nods of each type per knight" (SPEC-25)
- Guérison Rapide advantage says "Nod Soin: +3 extra PS recovered" (SPEC-24)

**Missing info an agent needs:**
1. What exactly does each Nod type restore? (Soin → PS, Armure → PA, Énergie → PE/PEs?)
2. Is the 3D6 restore the same for all three types?
3. Does Nod d'Énergie restore PE or PEs or both?
4. Is there a Combo Roll to use a Nod or is it automatic?
5. Can a Nod fail?
6. Can Nods be used on self AND allies, or only self?
7. What is the range for ally Nod use? (SPEC-04 says Courte, but only for "Nod on ally" — is self-use always valid?)
8. Can a Nod be used while in Agony? (Would break "no actions" rule)
9. Can a Nod be used while in Fold state? (SPEC-10 says yes, but what about Nod d'Énergie if PE is frozen?)

**Recommendation:** Create a dedicated SPEC (e.g., SPEC-33: Nod System) or add a full subsection to `03_CombatRolls.md` or `07_WeaponsAndEffects.md`.

---

### B-02: ✅ RESOLVED — Espoir Penalty vs Dice Pool — Ambiguous Floor Behavior
**Resolution:** Added explicit pool floor rule to SPEC-04 Roll Procedure (v8.9): "Dice pool minimum is 0. If modifiers reduce pool below 0, set pool = 0." **Pool = 0 is automatic failure** — no dice, no OD auto-successes, no Exploit/Failure Critique. OD represents technique built on an actual attempt; no attempt means no technique. Also added: modifier application order (Base+Combo → style → PEs penalty → injury penalties → floor), acceptance criterion for the floor case. SPEC-18.9 (enemy attacks) aligned: Exceptionnel auto-successes also don't apply at pool 0. SPEC-07 cross-reference updated.

**Files changed:** `03_CombatRolls.md` (SPEC-04 Roll Procedure, Edge Cases, Acceptance Criteria), `04_ArmorAndState.md` (SPEC-07 Purpose line), `06_EnemySystem.md` (SPEC-18.9 minimum pool).

---

### B-03: ✅ RESOLVED — Enemy Derived Values — Defense/Reaction Calculation Incomplete
**Resolution:** Verified all 5 demo enemies against both competing formulas — **neither formula reproduces all stat block values**. Nocte Defense (5), Bestian Reaction (3), Faune Reaction (2), and Ours Corrompu Defense (8) all fail to match any derivation. The tabletop bestiary hand-authors these values and the digital version must use them as-is.

**Changes made:**
- SPEC-18 §18.3: Removed the misleading `Aspect/2 + Exceptionnel bonus` formula. Replaced with authoritative stat block rule: "Do NOT derive them from a formula — no single formula reproduces all enemy values." Clarified Exceptionnels are offensive-only.
- SPEC-18.9 IMPORTANT DISTINCTION: Removed `Aspect/2 (round down)` + Exceptionnel mapping. Replaced with: "Enemy uses the `defense` and `reaction` values from the SPEC-21 stat block directly."
- SPEC-18.9 Step 7: Removed `Aspect/2 (round down) only for opposition scores` reference, replaced with §18.3 pointer.
- SPEC-21 header: Added general note that all stat block values (Defense, Reaction, Initiative) are authoritative.

**Files changed:** `06_EnemySystem.md` (SPEC-18 §18.3, §18.9), `08_ContentData.md` (SPEC-21 header).

**Verification matrix (all stat blocks checked):**
| Enemy | Defense (stat) | Reaction (stat) | Formula match? |
|-------|---------------|-----------------|---------------|
| Nocte | 5 | 1 | ❌ No formula works |
| Bestian | 4 | 3 | ❌ Reaction fails |
| Faune | 6 | 2 | ❌ Reaction fails |
| Behemot | 8 | 1 | ✅ Matches max(Chair,Bête)/2 |
| Ours P1 | 8 | 4 | ❌ Defense fails |

---

### B-04: ✅ RESOLVED — Bande Defense/Reaction — Never Defined
**Resolution:** Fixed as part of B-03. Added explicit note to SPEC-11 Knight vs Bande Attack Procedure: "Bande Defense/Reaction are fixed values from the SPEC-21 stat block (e.g., Nocte: Defense 5, Reaction 1) — they are NOT derived from aspects." The authoritative stat block note in SPEC-21 header also covers Bandes.

**Files changed:** `06_EnemySystem.md` (SPEC-11 Knight vs Bande Attack Procedure).

---

### B-05: ✅ RESOLVED — Injury Table Roll Procedure — Ambiguous Mapping
**Resolution:** Rewrote SPEC-23 §23.1 with explicit 2-step procedure: Step 1 = roll severity D6 (with probability table mapping D6 result → Column), Step 2 = roll row D6 (1–6 within column). Defined notation convention: `[severity die, row die]`. Added worked examples (`[3,5]` = Severe Column 2, Row 5). Specified RNG stream (SPEC-28 Combat). Updated acceptance criteria in SPEC-23 with full notation explanations. Fixed SPEC-08 Special Injuries: added notation legend, fixed truncated Hémorragie description, corrected Colonne Brisée from wrong `1,4` to correct `[4,1]`, and replaced terse "1 position = 2 Movement Actions" with clearer text.

**Files changed:** `08_ContentData.md` (SPEC-23 §23.1, §23 Acceptance Criteria), `04_ArmorAndState.md` (SPEC-08 Special Injuries).

**Bonus fix:** Corrected a latent bug in SPEC-08: old notation `1,4 Colonne Brisée` mapped to Column 1 (Catastrophic), Row 4 = "Colonne Vertébrale Touchée" (a *different*, more severe spinal injury). The actual Colonne Vertébrale Brisée is Column 3 (Moderate), Row 1 = `[4,1]`.

---

### B-06: ✅ RESOLVED — PEs Penalty — Applies to ALL Combo Rolls
**Resolution:** Design decision confirmed: PEs penalty applies to **every** Combo Roll a knight performs, with no exceptions. This includes attack rolls, Ghost activation/re-stealth, Stabilization rolls, Assist rolls, and Peur tests. Added explicit scope statement to SPEC-07 (canonical definition) listing all affected roll types. Added `[v8.9]` PEs penalty reminders to each non-attack Combo Roll location so agents implementing individual systems in isolation cannot miss it.

**Files changed:** `04_ArmorAndState.md` (SPEC-07 Purpose, Stabilization roll), `03_CombatRolls.md` (SPEC-04 PEs penalty line, Assistance special outcome), `05_ArmorAbilities.md` (SPEC-17.4 Ghost activation, Ghost re-stealth), `06_EnemySystem.md` (SPEC-18.10 Peur test roll).

**Design rationale:** PEs represents overall morale/mental state. When a knight's resolve crumbles, everything degrades — not just combat ability. A Rogue with PEs 3 (penalty 7) will have a drastically weaker stealth threshold, making Ghost nearly useless. This creates meaningful gameplay pressure: protect your Rogue's morale or lose stealth capability entirely.

---

### B-07: ✅ RESOLVED — Reconstruction Therapy vs Implants — Full Specification
**Resolution:** Design decision confirmed: Reconstruction Therapy (100 PG) is a full-body restoration — removes ALL injuries, resets implant count to 0, AND **restores maxPEs** to its pre-implant value. Lore: therapy rebuilds the knight's organic body with cloned tissue, restoring their humanity. Both Implant and Therapy are performed at the Camelot **Infirmary** (medical center).

**Changes made across 6 files:**
- SPEC-25 §25.2 (Hub Actions table): Rewrote both Implant and Therapy descriptions. Added `Location` column with Infirmary designation. Therapy now explicitly states maxPEs restoration with lore justification.
- SPEC-25 §25.3 (Spending PG table): Updated Therapy notes to include "Restores maxPEs to pre-implant value."
- SPEC-25 §25.5 (State Persistence): Added Implant persistence line. Clarified therapy resets both injuries and implants.
- SPEC-25 (Acceptance Criteria): Added 3 test cases — Implant purchase (maxPEs −3), Implant blocked (floor 10), Therapy (full reset, currentPEs unchanged).
- SPEC-01 (KnightBase schema): Added `implantCount` field (0–6) with SPEC-25 cross-reference.
- SPEC-01 (maxPEs formula): Updated derived value table and pseudocode to show full formula: `50 + tarotBonus − (implantCount × 3)`.
- SPEC-07 (Loss Sources #4): Updated implant entry to note reversibility via Therapy.
- SPEC-07 (Recovery Sources): Added entry #8: Reconstruction Therapy restores maxPEs.
- SPEC-23 (injury persistence): Updated to distinguish implant vs therapy with costs and effects.
- SPEC-26 (Camelot Hub UI): Added Infirmary to the Hub screen list.

**Files changed:** `10_MissionAndHub.md`, `01_KnightDataModel.md`, `04_ArmorAndState.md`, `08_ContentData.md`, `11_UIAndPresentation.md`.

---

### B-08: ✅ RESOLVED — Hémorragie (Hemorrhage) — Complete Specification
**Resolution:** Added complete Hémorragie subsection to SPEC-08 with all design decisions documented:

**Key design decisions:**
- **Countdown starts immediately** on injury (set to 3).
- **Ticks at start of bleeding knight's turn** (3→2→1→0).
- **Knight stays in Agony** throughout — cannot act, cannot self-heal.
- **Hémorragie is NOT permanent** — exception to the general injury rule. Exiting Agony (PS restored >= 1) removes Hémorragie from the injury list entirely.
- **Ignorer l'Agonie prevents Hémorragie** — it skips the entire Agony trigger sequence (no injury roll). But it **cannot** cancel an active Hémorragie countdown. No "second chance" Heroism spend at countdown 0.
- **Countdown = 0 → instant permadeath** — same finality as rolling `[1,1] MORT`. No additional roll.
- **Between-node persistence** — if combat ends with active Hémorragie, countdown continues from where it left off in the next encounter. Nods can be used freely between nodes to save the knight.

**Changes made across 4 files:**
- SPEC-08 (04_ArmorAndState.md): Complete rewrite — expanded Agony rules, Ignorer l'Agonie clarification, new §Hémorragie System with trigger sequence, countdown pseudocode, turn-by-turn timeline, save methods, 10-row interaction rules table, data model (hasHémorragie, hémorragieCountdown, InjuryResult.isPermanent), and 7 acceptance criteria.
- SPEC-23 (08_ContentData.md): Updated Hémorragie Interne row in injury matrix — cross-references SPEC-08, marked NOT permanent, added key rules summary.
- SPEC-33 (13_NodSystem.md): Fixed §33.7 — Hémorragie is now correctly described as non-permanent (was incorrectly stated as permanent). Updated AC5 to say "removed" instead of "stopped."
- SPEC-01 (01_KnightDataModel.md): Added `hasHémorragie` and `hémorragieCountdown` fields to KnightBase combat state. Added `InjuryResult` struct definition with `isPermanent` field.

**Files changed:** `04_ArmorAndState.md`, `08_ContentData.md`, `13_NodSystem.md`, `01_KnightDataModel.md`.

---

## 🟠 HIGH Issues (14)

### H-01: ✅ RESOLVED — Warrior Type OD Stacking — Hard Cap Defined
**Resolution:** Design decision confirmed: Normal OD hard cap = **5** from all sources (generation, modules, future). Warrior Type +1 is the **only** exception in demo, pushing effective OD to **6**. Future Evolution (+2 OD, 250 PG) would push to 7 but is out of demo scope.

The Type bonus is a **runtime-only** modifier — it is not persisted to `odLevels`. Formula: `effectiveOD = min(baseOD, 5) + (typeActive && typeMatchesAspect ? 1 : 0)`.

**Changes made across 3 files:**
- SPEC-01 (01_KnightDataModel.md): Updated `odLevels` comment with hard cap rule and Type exception. Added AC3b (OD 6 with Type) and AC3c (non-Warrior cap enforced).
- SPEC-17.1 (05_ArmorAbilities.md): Rewrote OD stacking row with explicit cap (5 normal, 6 with Type), formula, runtime-only note. Updated Evolutions note. Added 4 acceptance criteria covering cap, activation, deactivation, and non-Warrior enforcement.
- SPEC-04 (03_CombatRolls.md): Added OD cap note to the auto-successes line.

**Files changed:** `01_KnightDataModel.md`, `05_ArmorAbilities.md`, `03_CombatRolls.md`.

---

### H-02: ✅ RESOLVED — Grenade Hit Resolution — Always Reaction
**Resolution:** Design decision confirmed: Grenades are ranged weapons — always compare vs **Reaction**, never Defense, regardless of range or rank. Fixed the hit resolution text in SPEC-19.4 (removed incorrect "Defense if adjacent rank" clause). Added acceptance criterion testing same-rank grenade still uses Reaction.

**Files changed:** `07_WeaponsAndEffects.md` (SPEC-19.4 hit resolution, acceptance criteria).

---

### H-03: ✅ RESOLVED — Assist Action — Solitaire Difficulty Modifier Defined
**Resolution:** Design decision confirmed: "difficulty +1 level" means the **main Combo Roll's threshold** (Defense, Reaction, or narrative required successes) is increased by +1 when a knight with the Solitaire disadvantage acts as an assistant. The assistant's successes still contribute — but the bar is higher. Stacks per Solitaire assistant. Does NOT apply when the Solitaire knight is the primary roller being assisted.

**Changes made across 2 files:**
- SPEC-24 (09_TarotSystem.md): Rewrote Solitaire mechanic row with explicit digital implementation (threshold +1, combat and narrative, assistant-only direction, no effect on primary roller). Added 2 acceptance criteria (single and double Solitaire stacking).
- SPEC-04 (03_CombatRolls.md): Added Solitaire interaction note to the Assistance special outcome. Added 2 acceptance criteria with worked examples (miss due to threshold increase, hit despite threshold increase).

**Files changed:** `09_TarotSystem.md`, `03_CombatRolls.md`.

---

### H-04: ✅ RESOLVED — Cover System — Destruction/Removal Fully Specified
**Resolution:** Four design decisions documented:

1. **Cover destruction: NOT in demo.** Encounter cover is indestructible. Destructible cover is future content requiring a dedicated design pass (HP, thresholds, debris). Scaffolded only.
2. **Artillerie bypasses cover entirely** — fires over/around it, hits the combatant directly. Does NOT damage or destroy the cover object. This rule persists even when destructible cover is implemented in future content.
3. **Enemy cover destruction: NOT in demo.** No enemy capacity targets cover. Future content may add siege-type enemies.
4. **NanoC Cover Wall removal:** On expiry (10 turns), Priest Fold, or Priest death: `hasCover = false` at the rank. Combatants using Mise à Couvert are forced to Standard. Combatants using any other style are unaffected.

Added a **Cover Removal Procedure** (4-step) to SPEC-03 for any cover removal source.

**Changes made across 4 files:**
- SPEC-03 (02_PositionAndTurns.md): Added §Cover Persistence & Destruction (encounter vs NanoC rules, future scope notes), §Cover Removal Procedure (4-step), updated Artillerie note.
- SPEC-17.3 (05_ArmorAbilities.md): Expanded Cover Wall effect with expiry/Fold/death behavior and Cover Removal Procedure reference. Added 3 acceptance criteria.
- SPEC-20 (07_WeaponsAndEffects.md): Updated Artillerie effect with "does NOT damage cover" clarification.
- SPEC-04 (03_CombatRolls.md): Added cover dependency note to Mise à couvert style.

**Files changed:** `02_PositionAndTurns.md`, `05_ArmorAbilities.md`, `07_WeaponsAndEffects.md`, `03_CombatRolls.md`.

---

### H-05: ✅ RESOLVED — Weapon & Profile Switching — Full Cost System Defined
**Resolution:** Three-tier cost system based on physical manipulation:

1. **Switch equipped weapon** (Marteau → Fusil): always **1 Movement Action** (physically swapping what's in hand). Changed from previous "Free Action."
2. **Switch weapon profile (Free)**: weapon adapts automatically — swing vs trigger, stab vs shoot. No manual manipulation. Examples: Marteau-Épieu, Pistolet de Service, Grenades (type chosen on throw).
3. **Switch weapon profile (1 Movement Action)**: character must physically manipulate the weapon — swap magazine, change grip, reload bolt. Examples: Lance-Grenade Léger (ammo swap), future Épée Bâtarde (1H↔2H), future Arbalète Magnétique (bolt type).

**Additional fix:** Lance-Grenade ammo switch reduced from "1 full turn" to "1 Movement Action" (less punitive).

**Design pattern for new weapons:** `profileSwitchCost` is a per-weapon enum (FREE or MOVEMENT_ACTION) stored on WeaponProfile. Rule of thumb: if the character must do something with their hands → MOVEMENT_ACTION. If the weapon adapts to the action → FREE.

**Changes made across 2 files:**
- SPEC-04 (03_CombatRolls.md): Fixed weapon switch to 1 Movement Action. Rewrote profile switch as per-weapon cost. Added §Profile Switch Cost Rules (design pattern, demo weapon table, future weapon table). Added 4 acceptance criteria (weapon switch, free profile, movement profile, grenade type).
- SPEC-19 (07_WeaponsAndEffects.md): Added `profileSwitchCost` field to WeaponProfile schema. Added `profileSwitchCost` annotations to Marteau-Épieu (FREE), Pistolet de Service (FREE), Lance-Grenade Léger (MOVEMENT_ACTION, changed from "1 full turn").

**Files changed:** `03_CombatRolls.md`, `07_WeaponsAndEffects.md`.

---

### H-06: ✅ RESOLVED — Faune Weapon & Charge Brutale Damage Clarified
**Resolution:** Two issues fixed:

1. **Weapon name:** Renamed "Arme lourde" to "Griffes corrompues" (corrupted claws) — lore-appropriate, removes false implication of Lourd effect. No "(no Lourd effect)" disclaimer needed.

2. **Charge Brutale formula clarified:** Pre-baked weapon stat (4D6+16) already includes all normal Exceptionnel bonuses (+10 Bête score + +6 Exc score). Charge Brutale adds 2× Bête score as an ADDITIONAL flat bonus ON TOP of the pre-baked value. This means Bête contributes 3× total during a Charge (1× in pre-baked, 2× from Charge) — intentional, makes Charge the scariest single hit in the game. Balance check: Faune Charge = avg 50 dmg → heavy but survivable for Paladin (PA 120). Added explicit "do NOT strip out the +16" warning and canonical formula to SPEC-18.13 Step 4 and SPEC-18.9 Step 7 (exception note).

Also removed the incorrect "Bête Exceptionnelle is purely offensive" claim (this is H-07 territory — left for separate resolution).

**Changes made across 2 files:**
- SPEC-21.3 (08_ContentData.md): Renamed weapon to "Griffes corrompues." Replaced old note with explicit flat bonus breakdown and Charge formula walkthrough.
- SPEC-18.13 (06_EnemySystem.md): Clarified Step 4 damage formula with `chargeDamage = weaponDamageDice + weaponFlatBonus + (2 × enemy.bête)`. Added exception note to Step 7 about Charge being the only capacity that adds beyond pre-baked stat block.

**Files changed:** `08_ContentData.md`, `06_EnemySystem.md`.

---

### H-07: ✅ RESOLVED — Aspects Exceptionnels — Complete Per-Aspect Mechanics from Tabletop
**Resolution:** Complete rewrite based on tabletop source material. Each of the 5 aspects has **unique** Mineur and Majeur effects. Majeur always includes Mineur. The old spec incorrectly treated all aspects the same (generic auto-successes + damage bonus). The reality is far more complex:

| Aspect | Mineur | Majeur (includes Mineur) |
|--------|--------|--------------------------|
| **Chair** | −N flat damage/violence reduction on ALL incoming hits | Immune to Barrage X, Choc X, Meurtrier |
| **Bête** | +N contact damage (pre-baked) | +Bête score contact damage (pre-baked) |
| **Machine** | +N Reaction (pre-baked in stat block) | Immune to sensory abilities (Ghost, invisibility) |
| **Dame** | Redirect damage to allied PNJ at Courte range | Attacker must pass Hargne test (diff = N) or can't attack |
| **Masque** | +N Defense (pre-baked in stat block) | Initiative = 30 (always acts first) |

**Key discovery:** Chair Mineur/Majeur are significant DEFENSIVE mechanics that were completely missing from the spec. Behemot (Chair Majeur 5) is immune to Barrage, Choc, and Meurtrier AND reduces ALL incoming damage by 5. Ours (Chair Mineur 3) reduces all incoming damage by 3.

**Pre-baked vs runtime:** Bête damage, Machine Reaction, and Masque Defense are pre-baked in stat blocks. Chair reduction, Chair effect immunity, Machine sensory immunity, Dame redirect/test, and Masque initiative override are runtime mechanics that must be implemented.

**Changes made across 3 files:**
- SPEC-18.3 (06_EnemySystem.md): Complete rewrite with per-aspect tables, pre-baked vs runtime guide, demo enemy summary with runtime effects needed. Updated EnemyBase struct comment.
- SPEC-06 (04_ArmorAndState.md): Clarified Chair Mineur tag override description. Added Chair Majeur immunity to Meurtrier check (Step 7) in ArmourLayerResolver pseudocode.
- SPEC-20 (07_WeaponsAndEffects.md): Added Chair Majeur immunity notes to Barrage X, Choc X, and Meurtrier effect definitions.
- SPEC-18.9 Acceptance Criteria: Added 4 tests (Bête pre-baked, Chair Mineur reduction, Chair Majeur immunity, Masque Mineur pre-baked).

**Files changed:** `06_EnemySystem.md`, `04_ArmorAndState.md`, `07_WeaponsAndEffects.md`.

---

### H-08: ✅ RESOLVED — Actions Multiples — Combat Actions Only, Movement Unchanged
**Resolution:** Design decision confirmed: Actions Multiples (N) adds N extra **Combat Actions** only. Movement Actions are always exactly 1, unaffected by Actions Multiples. Since Combat Actions can be used as Movement Actions (general rule now explicitly stated in SPEC-04), extra Combat Actions provide flexibility but do NOT double movement capacity.

**Action economy:**

| Actions Multiples | Combat Actions | Movement Actions | Total |
|-------------------|---------------|-----------------|-------|
| None | 1 | 1 | 2 |
| (1) | 2 | 1 | 3 |
| (2) | 3 | 1 | 4 |

**Also fixed:** The general rule "Combat Action can be used as Movement Action" was only implied (Move 2 ranks = Forfeit Combat Action). Now explicitly stated as a top-level rule in SPEC-04 Action Economy.

**Changes made across 2 files:**
- SPEC-04 (03_CombatRolls.md): Added explicit "Combat Action can be used as Movement Action" general rule to Action Economy header. Applies to both knights and enemies.
- SPEC-18.8 (06_EnemySystem.md): Rewrote Actions Multiples with combat-only rule, action economy table (None/1/2), flexibility note, single initiative slot clarification. Updated EnemyBase schema (`actionsRemaining` → `combatActionsRemaining` + `movementActionsRemaining`). Updated AI flowchart loop. Updated 2 acceptance criteria.

**Files changed:** `03_CombatRolls.md`, `06_EnemySystem.md`.

---

### H-09: ✅ RESOLVED — Cadence X — Targeting Constraint Clarified
**Resolution:** Clarified: Cadence targets must be on **adjacent enemy ranks** (within 1 rank of each other: `abs(targetA.rank - targetB.rank) <= 1`). This is adjacency between **targets**, NOT gap from the attacker. Each target must also independently be within the weapon's range from the attacker (standard gap/position rules apply).

**Changes made across 2 files:**
- SPEC-20.4 (07_WeaponsAndEffects.md): Rewrote Cadence targeting constraint with explicit formula and clarification that it's inter-target adjacency, not attacker gap. Added acceptance criterion with valid/invalid target example.
- SPEC-18.11 (06_EnemySystem.md): Updated AI flowchart Cadence handling with adjacency-to-primary-target selection.

**Files changed:** `07_WeaponsAndEffects.md`, `06_EnemySystem.md`.

---

### H-10: ✅ RESOLVED — La Maison-Dieu (XVI) — Amnésique Algorithm Defined
**Resolution:** Precise algorithm defined: At character generation (SPEC-02 step 4), after 2 advantages and 1 disadvantage are selected, if the disadvantage is Amnésique: pick one of the 2 active advantages at random, replace it with a random advantage from the **full 22-card advantage table** (not limited to the knight's drawn cards). Replacement must differ from the remaining advantage (redraw up to 10 times). Happens once at generation, not at runtime. Original advantage is permanently lost. Stat bonuses from the original card still apply (only the advantage is replaced).

**Changes made:**
- SPEC-24.3 (09_TarotSystem.md): Rewrote Amnésique mechanic with complete algorithm (when, what's replaced, source pool, duplicate guard).
- SPEC-24.5 (09_TarotSystem.md): Added Amnésique post-processing step to generation integration.
- SPEC-24 Acceptance Criteria: Added worked example (Infatigable replaced by Chanceux).

**Files changed:** `09_TarotSystem.md`.

---

### H-11: ✅ RESOLVED — Barrage X — Suppress Action & Stacking Rules Defined
**Resolution:** Complete Suppress mechanics defined:

1. **Suppress = 1 Combat Action** (alternative to Attack, listed in Action Economy table).
2. **Source definition:** One source = one weapon instance from one character. Same source cannot Suppress the same target twice.
3. **Stacking:** Different sources stack additively. C1 (Barrage 2) + C2 (Barrage 2) on same target = −4 Def/React.
4. **Ambidextre Suppress:** 1 Combat Action fires 2 Suppress actions (one per weapon). Each weapon is a separate source — both can target the same enemy and stack. No −3 dice penalty (no attack roll). Example: main hand Barrage 2 + off-hand Barrage 4 → −6 Def/React on same target.
5. **Suppress + Attack combo:** Character with multiple Combat Actions can Suppress then Attack the same target to benefit from the debuff.
6. **Duration:** Barrage effect lasts until the start of the **suppressor's** next turn (not the target's). Tracked per source character. This means allies acting between the suppressor and the suppressor's next turn benefit from the debuff.

**Changes made across 2 files:**
- SPEC-20 (07_WeaponsAndEffects.md): Complete Barrage X rewrite with source definition, same-source restriction, stacking rules, Ambidextre Suppress, suppressor-based duration, tracking data model (`expiryOnSourceTurnStart`). Added 3 acceptance criteria (same-source rejection, Ambidextre stacking, Suppress+Attack combo).
- SPEC-04 (03_CombatRolls.md): Added Suppress (Barrage) to Action Economy table with suppressor-based duration. Added Ambidextre Suppress note to Dual-Wielding Rules.

**Files changed:** `07_WeaponsAndEffects.md`, `03_CombatRolls.md`.

---

### H-12: ✅ RESOLVED — Ghost Attack — First Attack Only, Ambidextre Clarified
**Resolution:** Ghost bonus applies to the **first attack only** after activation. Ghost deactivates immediately after the first attack resolves — before any subsequent attack in the same Combat Action. For Ambidextre (2 attacks in 1 Combat Action): first attack gets +Discrétion dice and +Discrétion damage, Ghost deactivates, second attack proceeds without Ghost (no bonus, knight is visible). This is a per-activation limit, not per-turn.

**Changes made:**
- SPEC-17.4 (05_ArmorAbilities.md): Rewrote "Ghost deactivation" row (immediate after first attack, before subsequent). Rewrote "Scope" row with explicit Ambidextre interaction and per-activation clarification. Added acceptance criterion (Rogue Ambidextre: 1st attack +7/+7, Ghost drops, 2nd attack 0/0).

**Files changed:** `05_ArmorAbilities.md`.

---

### H-13: ✅ RESOLVED — Prisonnier Disadvantage — Fulfillment Deferred to Future Content
**Resolution:** Prisonnier fulfillment ("complete 3 missions") is out of demo scope (single mission). Tagged as **FUTURE CONTENT**. In demo: the PEs blocking mechanic is permanently active with no resolution — the knight cannot recover PEs for the entire run. The fulfillment/unlock system is deferred. Agents must implement the PEs blocking (block all PEs gains when `MJ_PRISONNIER` is active and unfulfilled), but do NOT need to implement mission completion tracking or the unlock trigger.

**Changes made:**
- SPEC-24.3 (09_TarotSystem.md): Rewrote Prisonnier with "FUTURE CONTENT" tag on fulfillment. Clarified demo behavior (permanently active, no resolution). Separated what must be implemented in demo (PEs blocking) from what's deferred (fulfillment tracking, unlock).

**Files changed:** `09_TarotSystem.md`.

---

### H-14: ✅ RESOLVED — Despair — Forced Agressif Style Explicitly Stated
**Resolution:** Added forced Agressif style to the Despair Behavior section. The Despaired knight loses control and fights recklessly against former allies — Agressif is the natural fit (+3 dice, −2 Defense, −2 Reaction). Style is locked during Despair (cannot switch). Exception: Fold overrides to Standard (already documented in edge cases). Allies attacking the Despaired knight benefit from the −2 Def/React penalty.

**Changes made:**
- SPEC-07 (04_ArmorAndState.md): Added "forced Agressif" rule to Despair Behavior section with lore justification, style lock, and Fold override reference. Added 2 acceptance criteria (Despair Agressif with penalties, Despair + Fold = Standard override).

**Files changed:** `04_ArmorAndState.md`.

---

## 🟡 MEDIUM Issues (16)

### M-01: ✅ RESOLVED — Lance-Grenade Léger — Already Fixed in H-05
**Resolution:** Already addressed during H-05 fix. Lance-Grenade ammo switch was changed from "1 full turn" to "1 Movement Action" and `profileSwitchCost = MOVEMENT_ACTION` was added to the weapon description and schema. See H-05 resolution for details.

**Files changed:** (see H-05).

---

### M-02: ✅ RESOLVED — Trompe la Mort — Second Death Is Final
**Resolution:** Confirmed: Trompe la Mort ignores Death once per mission (reroll Injury Table, take new result). If Death is rolled a second time during the same mission, the knight dies permanently — no second reroll. Added `trompeLaMortUsed: boolean` tracking and explicit "second Mort is final" wording.

**Files changed:** `09_TarotSystem.md`.

---

### M-03: ✅ RESOLVED — Shrine Placement — Consistent Free Action, Maintenance, Disable Conditions
**Resolution:** Shrine placement is ALWAYS Free Action (no action cost), both initial and re-placement. Cost: 2 PE to place, +1 PE/turn to maintain. Shrine stays active until disabled. Disable conditions: PEs reaches 0, Fold, Agony, Despair, death, or voluntary disable. Re-placement after any disable = same as initial (Free Action, 2 PE). Removed contradictory "combat action to re-place after Fold" wording. Updated Action Economy table.

**Files changed:** `05_ArmorAbilities.md`, `03_CombatRolls.md`.

---

### M-04: ✅ RESOLVED — Weapon Violence Column Standardized to XD6 + Y Format
**Resolution:** Confirmed: bare "1" in Violence columns means flat 1 (0D6 + 1), not 1D6. Standardized all stat blocks to explicit `XD6 + Y` format. Changed: Pistolet de Service Contact (1 → 0D6 + 1), Lance-Grenade Antiblindage (1 → 0D6 + 1), Unarmed Strike (1 → 0D6 + 1). Added formatting convention note to §19.3 header: "never use bare numbers without D6 prefix."

**Files changed:** `07_WeaponsAndEffects.md`.

---

### M-05: ✅ RESOLVED — Enemy "Module" Renamed to "Capacity" for Demo
**Resolution:** Renamed "Module (Saut nv1)" and "Module (Saut nv2)" to "Capacity (Saut nv1/nv2)" in Bestian and Faune stat blocks. Added note: future human-type enemies may have actual modules; for demo, all enemy special abilities are capacities.

**Files changed:** `08_ContentData.md`.

---

### M-06: ✅ RESOLVED — Point Faible — No Tier Restriction
**Resolution:** Point Faible is not limited to T5 Patrons. Any enemy can have a Point Faible, defined per-enemy at design time. Removed "(T5 Patrons)" from section title and "Each patron" wording. Updated EnemyBase schema: `pointFaible` moved out of "T5 Patron special" section, made nullable (not all enemies have one). Behemot (T4) having Point Faible: Endurance is now fully consistent.

**Files changed:** `06_EnemySystem.md`.

---

### M-07: ✅ RESOLVED — Dispersion — Radius-Based Explosion Model
**Resolution:** Complete redesign. Dispersion X = total positions covered (target + collateral). X is always odd. Radius = (X-1)/2 ranks outward from target in each direction. The blast zone covers a symmetric area on the linear 8-position track (KR4–KR1–ER1–ER4). All combatants (any faction) within the blast zone are hit — friendly fire is real and predictable.

**Key design decisions:**
- **X = total positions affected**, radius = (X-1)/2. X is always odd (1, 3, 5, 7).
- **Tabletop conversion:** even numbers round up to nearest odd, capped at 5 for balance.
- **Grenades reworked:** Dispersion 6 → Dispersion 5 (radius 2). Explosive grenade stays at Dispersion 3.
- **Ours Corrompu:** Souffle ténébreux Dispersion 2 → Dispersion 3 (radius 1).
- **Friendly fire** is a core tactical mechanic: targeting enemy frontline pulls the blast across the center line.

| Dispersion X | Radius | Pattern | Friendly fire risk |
|---|---|---|---|
| 1 | 0 | Target only | None |
| 3 | 1 | Target ± 1 rank | Low (only if targeting frontline from frontline) |
| 5 | 2 | Target ± 2 ranks | Medium-High (crosses center line at mid-range) |
| 7 | 3 | Target ± 3 ranks | Very High (covers most of the track) |

**Changes made across 2 files:**
- SPEC-20.3 + §20.5 (07_WeaponsAndEffects.md): Complete rewrite — radius model, linear track, resolution pseudocode, tabletop conversion table, friendly fire rules, bande exception, 5 worked examples, tactical takeaways. Updated Dispersion X definition in Crowd Control table. Changed grenade Dispersion 6→5. Updated grenade splash description. Updated acceptance criteria.
- SPEC-21 (08_ContentData.md): Converted Ours Souffle ténébreux Dispersion 2→3.

**Files changed:** `07_WeaponsAndEffects.md`, `08_ContentData.md`.

---

### M-08: ✅ RESOLVED — MJ-02 — Boss-Target Motivation, Assigned at Generation
**Resolution:** Reworked MJ-02 entirely. At character creation, one boss enemy is randomly selected from the campaign's boss pool and assigned as this knight's target (`targetBossId`). The motivation triggers when the party kills the assigned boss during any mission. Removed the incorrect "Hostile-tier or higher (tier 4+)" wording. In demo: the only boss is Ours Corrompu, so the assignment is trivially that one boss. The full random-from-pool mechanic becomes meaningful in the full game with multiple patrons across missions.

**Files changed:** `08_ContentData.md`.

---

### M-09: ✅ RESOLVED — Paladin Watchtower — Reaction Halving Clarified
**Resolution:** Watchtower halves the Paladin's **base Reaction** score using `floor(baseReaction / 2)`. Rank and cover modifiers are applied AFTER halving. The halving persists while Watchtower is active. On deactivation (Free Action), base Reaction is fully restored. Added worked example acceptance criterion (base 5 → floor 2, at Rank 4 with −1 modifier → effective 1).

**Files changed:** `05_ArmorAbilities.md`.

---

### M-10: ✅ RESOLVED — NanoC Barricade — Blocks All Involuntary Movement
**Resolution:** Barricade blocks **all involuntary (forced) displacement** through, into, or out of the Barricade rank. Voluntary movement is NOT blocked — combatants can freely walk through on their own turn. Added explicit examples of blocked movement (Choc push, charge knockback, forced rank change capacities) and allowed movement (voluntary move/advance).

**Files changed:** `05_ArmorAbilities.md`.

---

### M-11: ✅ RESOLVED — Blason Collision Check — String-Tag Only, No Semantic Check
**Resolution:** Design decision confirmed: Blason Voeu collision checking is **string-tag-only** (`voeuEventTag != mn.eventTag`). No semantic overlap check is needed. If VOEU_OURS and MN-02 have different tag strings, they are considered non-colliding regardless of any lore/narrative overlap. This keeps the algorithm deterministic and simple — semantic deduplication would require subjective judgment that automated generation cannot reliably perform.

No spec changes needed — existing wording is correct as-is.

---

### M-12: ✅ RESOLVED — Choc X — New Choc Ignored While Active
**Resolution:** Design decision: if a target already has an active Choc effect (actionsToLose > 0), any new Choc application is **ignored entirely** — no refresh, no replacement, no upgrade. The original Choc runs its course. Same rule applies to Parasitage (mutual exclusion already specified). This is the simplest, most predictable behavior and matches the tabletop "does not stack" intent.

**Changes made:**
- SPEC-20 (07_WeaponsAndEffects.md): Updated Choc X implementation notes with explicit "ignored entirely" wording and edge case example (Choc 1 active, Choc 2 incoming = ignored).

**Files changed:** `07_WeaponsAndEffects.md`.

---

### M-13: ✅ RESOLVED — Ambidextre — Same or Different Weapons Allowed
**Resolution:** Design decision confirmed: Ambidextre does NOT require two different weapons. A knight can use Ambidextre with two identical weapons (e.g., 2× Couteau de Combat) or two different weapons (e.g., 1 Couteau + 1 Pistolet). The key distinction between Ambidextre and Akimbo is the **attack resolution style**, not the weapon identity:
- **Ambidextre:** 2 separate attacks (2 separate Combo Rolls), each weapon resolves independently. Can target same or different enemies.
- **Akimbo:** 1 combined attack (1 Combo Roll), damage dice pooled. Must target same enemy.

The Rogue default loadout (2× Couteau de Combat) now works correctly with Ambidextre style — 2 separate attacks with independent Combo Rolls, each benefiting from Jumelé (Ambidextrie) penalty reduction.

**Changes made:**
- SPEC-04 (03_CombatRolls.md): Updated Ambidextre description — removed "two different" restriction. Updated Akimbo description — clarified the distinction is resolution style, not weapon identity. Added note: "Two identical weapons can use either Ambidextre or Akimbo at the knight's choice."

**Files changed:** `03_CombatRolls.md`.

---

### M-14: ✅ RESOLVED — Archetype Table — choicePool Column Added
**Resolution:** Added a `choicePool` column to the SPEC-02.9 archetype table. Each row now explicitly shows whether the bonus is fixed (`—`) or randomly selected from a pool (`[Combat, Tir]`). For archetypes #7 and #8, the choice applies to **bonus1** (not bonus2), which was previously ambiguous in the table layout. Updated `ArchetypeData` struct to include `choiceBonusSlot` field (0 = no choice, 1 = bonus1 from pool, 2 = bonus2 from pool) so agents know exactly which slot to randomize.

**Changes made:**
- SPEC-02.9 (01_KnightDataModel.md): Added `choicePool` column to archetype table (17 rows). Updated `ArchetypeData` struct with `choiceBonusSlot` field. Added column rules explanation.

**Files changed:** `01_KnightDataModel.md`.

---

### M-15: ✅ RESOLVED — Esprit de Contradiction — Trigger & Proc Chance Defined
**Resolution:** Complete trigger system defined:
- **When:** At the start of each of this knight's combat turns, roll a **30% proc chance** (`rng.Chance01(0.30f)` using Combat RNG stream).
- **Target:** Nearest allied knight by rank distance (ties broken randomly by Combat RNG).
- **Effect:** Target ally's next Combo Roll this turn gets +1 difficulty level (Defense/Reaction +1 for combat, required successes +1 for narrative).
- **Frequency:** At most **once per combat encounter** (not per mission — multiple encounters in a mission can each trigger it). Track `espritDeContradictionUsed: boolean`, reset per encounter.
- **Edge case:** If no ally is reachable (solo knight alive), the disadvantage does not proc.
- **UI:** Brief dialogue popup showing the contrarian knight's objection.

The 30% chance makes it impactful but not guaranteed, creating interesting tension without being overly punitive.

**Changes made:**
- SPEC-24.3 (09_TarotSystem.md): Rewrote Esprit de Contradiction with proc chance, target selection, frequency cap, edge case, and tracking field.

**Files changed:** `09_TarotSystem.md`.

---

### M-16: ✅ RESOLVED — CombatSession Snapshot — Explicit Field Boundary Defined
**Resolution:** Added a complete **RunState vs CombatSession field boundary** table to SPEC-32. Every KnightBase field is now explicitly categorized:

**RunState (persistent):** Identity, generation data, Aspects/Characteristics, Meta-Armor stats, all resource values (PS/PA/PE/PEs/CdF/Héroïsme), injuries, implant count, equipment, Nods, isDead. Also Hémorragie (exception — carries across encounters).

**CombatSession-only (ephemeral):** position, activeStyle (reset to Standard), all combat boolean states (Agony/Fold/Despair/Ghost), activeWarriorTypeIndex, ALL status effects (Barrage, Choc, Parasitage, etc.), turn queue, initiative order, Shrine active state, per-encounter disadvantage tracking.

**ApplyCombatResults:** Added canonical pseudocode showing the atomic commit — the ONLY place where CombatSession writes back to RunState. Key rules: combat never modifies Aspects/Chars/OD/equipment directly; crash = no state corruption; Hémorragie is the one combat state that persists across encounters.

**Changes made:**
- SPEC-32 (12_ArchitectureGuardrails.md): Added §RunState vs CombatSession — Explicit Field Boundary with 2 tables (RunState fields, CombatSession-only fields), ApplyCombatResults pseudocode, and 3 key rules.

**Files changed:** `12_ArchitectureGuardrails.md`.

---

## 🔵 LOW Issues (9)

### L-01: ✅ RESOLVED — Index File Line Counts Removed
**Resolution:** Removed the `Lines` column from the 00_INDEX File Manifest table entirely. Line counts go stale with every spec edit — better to not track them at all than maintain inaccurate data.

**Files changed:** `00_INDEX.md`.

---

### L-02: ✅ RESOLVED — SPEC-02.9 Added to Header
**Resolution:** Added `02.9` to the SPECs header line in `01_KnightDataModel.md`. Header now reads: `SPECs: 01, 02, 02.5, 02.7, 02.8, 02.9`.

**Files changed:** `01_KnightDataModel.md`.

---

### L-03: ✅ RESOLVED — Ranger Column Marked as FUTURE
**Resolution:** Renamed the Ranger column header to `Ranger ⚠️FUTURE` in the Weighted Distribution Table. The existing paragraph below the table already says "Ranger is excluded from demo generation pool — do not select during SPEC-02 step 1." The column header now provides an immediate visual warning.

**Files changed:** `01_KnightDataModel.md`.

---

### L-04: ✅ RESOLVED — PEs Penalty Line Already Fixed in B-06
**Resolution:** Already addressed during B-06 fix. The PEs penalty line in SPEC-04 now contains the complete formula (`pesPenalty = max(0, 10 − currentPEs)`), cross-reference to SPEC-07, and explicit note that it applies to every Combo Roll.

**Files changed:** (see B-06).

---

### L-05: ✅ RESOLVED — Blason Count Header Fixed
**Resolution:** Changed Blason table header from "Demo Blason Table (10 Blasons)" to "Demo Blason Table (10 Blasons listed, 9 active in demo)". Makes it immediately clear that Le Dragon (#4) is deferred and not rollable.

**Files changed:** `01_KnightDataModel.md`.

---

### L-06: ✅ RESOLVED — Shotgun Escamotable — Unlimited Ammo Confirmed
**Resolution:** Design decision confirmed: Shotgun Escamotable intentionally has unlimited ammo (no Chargeur effect). This is consistent with the Pistolet de Service, which also has unlimited ammo. No spec changes needed.

---

### L-07: ✅ RESOLVED — SPEC-06 Worked Example Completed
**Resolution:** Rewrote the Destructeur+Meurtrier worked example with all 8 steps fully detailed. Shows CdF absorption (Step 2), Destructeur +2D6 trigger at Step 5 (remaining > 0, PA > 0), Perce Armure 20 reducing effective PA at Step 6, Meurtrier +2D6 trigger at Step 7 (remaining > 0 after PA), and final PS damage at Step 8. Added an alternate outcome showing the same scenario without Meurtrier for comparison.

**Files changed:** `04_ArmorAndState.md`.

---

### L-08: ✅ RESOLVED — Code Moral "Friendly Fire" Defined
**Resolution:** Defined "friendly fire" for Code Moral tracking: any Dispersion X splash damage from this knight's attack that hits an allied knight counts as a code breach. This includes grenades, Lance-Grenade, Marteau-Épieu charge, and any other weapon with a Dispersion effect. Implementation: set `codeMoralBroken = true` whenever `applyDamage(target)` is called with an allied target and this knight as the source. Also clarified the "fleeing" condition as moving from Rank 1 to Rank 2+ while any allied knight at Rank 1 is in Agony.

**Files changed:** `09_TarotSystem.md`.

---

### L-09: ✅ RESOLVED — Bande Débordement — Faction-Based Targeting
**Resolution:** Rewrote Débordement description from "ALL enemies" (ambiguous — enemies of whom?) to "ALL combatants of the **opposing faction**." Added implementation note: use `bande.faction != target.faction` — never hardcode "damages knights." Added FUTURE NOTE for allied Bandes: when allied Bande reinforcements are implemented, their Débordement would damage all enemy on-track combatants using the same faction-based rule. Also fixed the Targeting Rules subsection with matching faction-neutral language.

**Files changed:** `06_EnemySystem.md`.

---

## ⚪ DUPLICATE Issues (3)

### D-01: ✅ RESOLVED — Squad Selection — Consolidated to SPEC-03
**Resolution:** SPEC-02 (01_KnightDataModel.md) squad selection section replaced with a cross-reference to SPEC-03 (02_PositionAndTurns.md) as the canonical deployment definition. SPEC-02 retains a brief summary and the UI roster view details (since UI is SPEC-02's domain), but the core deployment rules ("4 from 8, no swaps mid-mission") point to SPEC-03.

**Files changed:** `01_KnightDataModel.md`.

---

### D-02: ✅ RESOLVED — Colonne Vertébrale Brisée — Consolidated to SPEC-23
**Resolution:** Made SPEC-23 Column 3, Row 1 the **canonical definition** for Colonne Vertébrale Brisée (full detail: 2 actions per rank shift, cannot Sprint, implement as doubled action cost, forced displacement still applies). The two duplicates were replaced:
- SPEC-03 (02_PositionAndTurns.md) §Forced Displacement: replaced full description with cross-reference to SPEC-23.
- SPEC-23.3 (08_ContentData.md) §Digital Adaptations: replaced full repeat with "See Column 3, Row 1 above."
- SPEC-03 Acceptance Criteria: updated Colonne Vertébrale line with SPEC-23 cross-reference.

**Files changed:** `02_PositionAndTurns.md`, `08_ContentData.md`.

---

### D-03: ✅ RESOLVED — Dependency Map — Consolidated to SPEC-16
**Resolution:** Removed the full dependency map table from 00_INDEX.md and replaced it with a cross-reference to SPEC-16 (12_ArchitectureGuardrails.md) as the canonical source. INDEX retains a compact "quick reference" summary of build phases (1-line per phase) for navigation convenience, but agents must read SPEC-16 for the full map with version history and exact dependencies.

**Files changed:** `00_INDEX.md`.

---

## 💡 SUGGESTIONS (2)

### S-01: ✅ RESOLVED — Constants & Enumerations Reference File Created
**Resolution:** Created `14_Constants.md` — a consolidated reference file containing every enum, constant, and magic value used across all 13 spec files. Organized into 7 sections:
1. **Identity & Aspect Enums** — AspectId, CharacteristicId (with Aspect→Char mapping table), ArmorClass
2. **Combat Enums** — CombatStyle (9 values with modifier summaries), NodType
3. **Weapon Enums** — WeaponCategory, WeaponSlot, ProfileType, ProfileSwitchCost, WeaponRange (with Gap mapping table), ForceMode, WeaponEffectId (complete 30+ values by category)
4. **Enemy Enums** — EnemyTier, TargetPriority, RankBehavior (including Hybrid patterns)
5. **Architecture Enums** — LifeState, ControlState, ActionState, ArmorState, RNGPolicy, RngStreamId, EncounterType, TriggerEvent, TriggerActionType
6. **Generation Enums** — ConditionType
7. **Global Constants** — 30+ game constants (max values, base values, thresholds, ratios) with source SPECs

Each entry lists its canonical source SPEC. The file is read-only reference — no new logic.

**Files created:** `14_Constants.md`.  
**Files changed:** `00_INDEX.md` (added to manifest + cross-reference convention).

---

### S-02: ✅ RESOLVED — Agent Task Boundary Definitions Added
**Resolution:** Added a complete "Agent Task Boundaries" section to `00_INDEX.md`. Maps every SPEC to exactly one owning agent role across 9 agent roles:
- **Data Agent:** SPEC-01, 02 family, 24 → KnightBase, Generation, Tarot
- **Weapon & Content Agent:** SPEC-19, 20, 21, 23 → Weapons, Effects, Enemy roster, Injury table
- **Combat Core Agent:** SPEC-03, 04, 05, 06, 15 → Position, Rolls, Damage, Armour, Turns
- **Combat State Agent:** SPEC-07, 08, 09, 10, 33 → Espoir, Injury, Heroism, Fold, Nods
- **Armor Abilities Agent:** SPEC-17 → Warrior/Paladin/Priest/Rogue class abilities
- **Enemy AI Agent:** SPEC-11, 14, 18 → Enemy model, Bandes, Bosses, AI
- **Mission & Hub Agent:** SPEC-12, 13, 22, 25 → Motivations, Save, Mission, Camelot
- **Architecture Agent:** SPEC-16, 28, 29, 30, 32 → RNG, Encounters, State, Conventions
- **UI Agent:** SPEC-26, 27, 31 → Screens, Audio/VFX, Info contract

Includes 5 boundary rules: single ownership, read-only cross-deps, flag contradictions, Phase 1 must complete first, and parallelism guidance.

**Files changed:** `00_INDEX.md`.

---

## Cross-File Consistency Matrix

Quick reference for where the same concept is defined across multiple files. Green = consistent, Yellow = minor wording difference, Red = contradiction.

| Concept | Files | Status |
|---------|-------|--------|
| Gap Formula | SPEC-03, SPEC-19, SPEC-20 | 🟢 Consistent |
| Defense/Reaction (Knights) | SPEC-01, SPEC-04, SPEC-06 | 🟢 Consistent |
| Defense/Reaction (Enemies) | SPEC-18.3, SPEC-18.9, SPEC-21 | 🟢 See B-03 (resolved — stat block authoritative) |
| PEs Penalty | SPEC-04, SPEC-07, SPEC-17.4, SPEC-18.10 | 🟢 See B-02 + B-06 (resolved — pool floor + all Combo Rolls) |
| Fold State restrictions | SPEC-10, SPEC-17 | 🟢 Consistent |
| Débordement | SPEC-11, SPEC-21 | 🟢 Consistent |
| Nod mechanics | SPEC-04, SPEC-10, SPEC-25, SPEC-33 | 🟢 See B-01 (resolved — SPEC-33 created) |
| Hémorragie | SPEC-08, SPEC-23, SPEC-33 | 🟢 See B-08 (resolved — full spec in SPEC-08) |
| Squad Selection | SPEC-02, SPEC-03 | 🟢 See D-01 (resolved — canonical in SPEC-03, cross-ref from SPEC-02) |
| Colonne Vertébrale Brisée | SPEC-03, SPEC-23, SPEC-23.3 | 🟢 See D-02 (resolved — canonical in SPEC-23, cross-ref elsewhere) |
| Dependency Map | INDEX, SPEC-16 | 🟢 See D-03 (resolved — canonical in SPEC-16, INDEX has summary only) |
| Charge Brutale | SPEC-18.13, SPEC-21 | 🟢 Consistent |
| Cover System | SPEC-03, SPEC-17 (NanoC), SPEC-20 | 🟢 Consistent |
| Heroism sources/spending | SPEC-09, SPEC-04, SPEC-07 | 🟢 Consistent |
| Ghost detection | SPEC-17.4 | 🟢 Self-contained |
| Damage chain (8 steps) | SPEC-05, SPEC-06 | 🟢 Consistent |
| Dual-Wielding (Ambidextre/Akimbo) | SPEC-04, SPEC-19, SPEC-20 | 🟢 See M-13 (resolved — same/different allowed) |
| Choc/Parasitage stacking | SPEC-20 | 🟢 See M-12 (resolved — ignored while active) |
| Barrage stacking & duration | SPEC-04, SPEC-20 | 🟢 See H-11 (resolved — suppressor-based duration) |
| Dispersion radius model | SPEC-19, SPEC-20, SPEC-21 | 🟢 See M-07 (resolved — odd-number radius) |
| RunState/CombatSession boundary | SPEC-01, SPEC-32 | 🟢 See M-16 (resolved — explicit field tables) |
| Weapon/Profile switch costs | SPEC-04, SPEC-19 | 🟢 See H-05 (resolved — 3-tier cost system) |
| Aspects Exceptionnels (enemies) | SPEC-18.3, SPEC-20, SPEC-21 | 🟢 See H-07 (resolved — per-aspect tables) |

---

## Priority Fix Order

~~All BLOCKER, HIGH, and MEDIUM issues have been resolved.~~ ✅ **COMPLETE.**

Original priority order (preserved for reference — all items resolved):

1. ✅ **B-01 (Nod System)** — SPEC-33 created
2. ✅ **B-05 (Injury Roll Procedure)** — SPEC-23 rewritten
3. ✅ **B-08 (Hémorragie)** — SPEC-08 expanded
4. ✅ **B-03 (Enemy Defense/Reaction)** — Stat block authoritative
5. ✅ **B-02 (Pool Floor)** — Pool = 0 = auto-fail
6. ✅ **B-04 (Bande Defense)** — Fixed with B-03
7. ✅ **H-06 (Faune Charge Brutale)** — Formula clarified
8. ✅ **H-07 (Aspects Exceptionnels)** — Per-aspect tables from tabletop
9. ✅ **B-06 (PEs on utility rolls)** — All Combo Rolls affected
10. ✅ **B-07 (Reconstruction Therapy)** — Full Infirmary spec

**Remaining (non-blocking):** None. All 52 issues resolved. 🎉

---

*End of analysis. 52 issues identified, **52 resolved (100%)**. Specifications are fully ready for autonomous agent handoff. No open items remain.*

