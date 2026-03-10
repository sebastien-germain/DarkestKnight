# KNIGHT: INTO THE DARKNESS — Part 3: Issue Tracking & Changelog
> **Archive** — All 81 issues resolved across 8 review passes. This file has no implementation value.
> See [00_INDEX.md](../split/00_INDEX.md) for the active specification.
---
  OnDespairExited(knight, reason) — Despair ends (stabilized / PEs restored / defeated)
  OnDespairPermanent(knight) — duration expires, Despair becomes permanent hostile AI
Agent Note: All events are fire-and-forget. No gameplay logic depends on event listeners. Events exist solely for presentation layer (audio, VFX, screen shake, UI animations).
End of Technical Specification v8.8. 32 SPEC sections + SPEC-02.5 (Blason) + SPEC-02.8 (Haut Fait) + SPEC-18.12 (Hybrid AI) + Issue Tracking. ALL v7 through v8.8 findings RESOLVED. Document is agent-ready.
CHANGELOG
v8.8 (2026-02-26) — Pass 8 Deep-Dive Validation + Remediation: HIGH: H-1 Complete injury table digital effects filled for all 24 entries (19 were empty). Tabletop-accurate characteristic reductions with specific Aspect/characteristic targets, derived value recalculation triggers. All effects from Livre de Base p.91. MEDIUM: M-1 PEs dice penalty canonical formula added: pesPenalty = max(0, 10 − currentPEs), stated in SPEC-04 and SPEC-07. M-2 Style gate corrections: Puissant requires Lourd Contact weapon, Pilonnage requires Deux Mains Ranged weapon. SPEC-20 Lourd/Deux Mains definitions updated. M-3 Demo weapon availability note: Puissant scaffolded (no Lourd weapons in demo), Pilonnage active (Fusil de Précision, Fusil d'Assaut, Shotgun Escamotable). M-4 Cybernetic implant PEs penalty: −3 maxPEs per implant, max 6, floor 10. M-5 Hémorragie Interne linked to standard 3-turn countdown (SPEC-08). LOW: L-1 Cerveau Touché clarified (12 chars = Bête+Machine+Dame+Masque, action loss is knight's choice). L-2 HautFaitData struct standalone with ConditionType enum. L-3 Mise à couvert prose (Dispersion/Débordement/Cover interactions). L-4 Phase 2 AI change verified in SDT (false positive). Total: 81 issues across 8 reviews, ALL RESOLVED.

v8.7 (2026-02-26) — Pass 7 Deep-Dive Validation + Remediation: CRITICAL: C-1 maxPS formula corrected from flat 10 to canonical tabletop formula (10 + 6 × Max(Force,Endurance,Déplacement) [OD excluded]). HIGH: H-1 IsSquadDefeated canonical code fixed — temp Despair (isDespairPermanent==false) no longer triggers squad defeat. H-2 MN templates verified — all 10 present in SDT tags (false positive). MEDIUM: M-1 Injury recalculation AC corrected (Chair→maxPS, Bête→Defense, Machine→Reaction, Masque→Initiative). M-2 Stray } and empty paragraphs removed near IsSquadDefeated. M-3 OnCriticalHit removed (duplicate of OnExploit). M-4 TriggerEvent ev removed from EncounterTrigger (belongs in TriggerCondition only). M-5 Combat style table filled (Agressif/Défensif/Puissant/Pilonnage modifiers + restrictions). LOW: L-1 Haut Fait table completed (14 entries with conditions, callsigns, weighted dual-Aspect resolution per Armor Class primary aspects). L-2 isInDespair standardized to isDespair across all schemas. L-3 despairDuration field added to CombatantState. L-4 Ghost activation cost clarified. Total: 71 issues across 7 reviews, ALL RESOLVED.

v8.6 (2026-02-25) — Structural Remediation + Expert Review Pass 6: CRITICAL: C-1 Duplicate SPEC-01 merged (original runtime schema retained, enhanced with derived value pseudocode, Aspect/Characteristic reference, SPEC-01-AC1→AC4). HIGH: H-2 IsSquadDefeated stub removed (canonical implementation only). MEDIUM: M-1 Hash Algorithm block moved outside IRng interface. M-2 TriggerCondition/TriggerAction schemas moved outside EncounterTrigger struct with cross-reference note. M-3 SPEC-30-AC3 filled (Despair temporary vs permanent defeat evaluation). LOW: L-1 Part 3 intro corrected from "OPEN" to audit trail. Total: 59 issues across 6 reviews, ALL RESOLVED.

v8.5 (2026-02-24) — Agent Handoff Addendum + v8.5 Expert Review: Part 4 Addendum (SPEC-28→32). Expert Review 3H+5M+4L resolved: H-1 SPEC-01 KnightBase standalone section. H-2 xxHash32 mandatory hash, Xoshiro128** PRNG, uint seed types. H-3 canonical IsSquadDefeated(). M-1 debordementScore aligned. M-2 countdown timers. M-3 Nocte vs Shrine worked example. M-4 state machine updated. M-5 version header. L-1 TriggerCondition/TriggerAction schemas. L-2 Combat State Isolation. L-3 hémorragieCountdown. L-4 SPEC-22/29 cross-ref. Total: 56 issues across 5 reviews, ALL RESOLVED.

v8.4 (2026-02-24) — v8.4 Expert Review: Edge Cases, Parameters, Polish

HIGH: H-1 Le Cerf tracking contradiction fixed (neverMovedFlag tracking replaces ambiguous final-rank comparison). MEDIUM: M-1 Prisonnier PEs freeze scope clarified (minor motivations only, other sources function normally). M-2 Le Soleil disadvantage corrected from Egoiste to Egoiste (cannot be Adjuvant, cannot Assist allies). M-3 Node 4 Point Faible discovery probability set (1D6, triggers on 6 only — intentionally rare). M-4 Phase 2 Nocte bande respawn fully specified (fresh spawn if destroyed, AddCohesion if alive). M-5 MJ-06 post-liberation flow corrected (Prisonnier disadvantage removed, new major motivation drawn at Camelot). LOW: L-1 Le Dragon removed from demo Blason table (roll scope ambiguous, deferred to full game; 9 demo Blasons remain). L-2 Node 2 branch choice UX specified (title + narrative + star rating, no enemy preview). L-3 Le Pendu commandment/Blason Voeu overlap confirmed intentional with design note. L-4 Le Fou (+6 free char pts via full 15-char weighted distribution) and La Maison-Dieu (+2 Aspect pts only, no char distribution) documented.

v8.3 (2026-02-24) — v8.3 Expert Review: Blason System, Tarot Special Cases, Hybrid AI
HIGH: H-1 Complete Blason system with 10 demo Blasons, Voeu event tags, uniqueness rule, data model (SPEC-02.5). Le Cheval and Le Corbeau deferred to full game. H-2 Tarot special cases resolved: Le Pendu advantage “Code Moral” with 6 feasible Code d’Honneur commandments (1 random per mission); Le Pendu disadvantage “Sacrifice Total” clarified (no major motivation ever); Le Jugement disadvantage “Prisonnier” with micro-bomb (PEs < 10 = permadeath), frozen PEs gain, MJ-06 “Retrouver la Liberté” (2+1D3 missions, visible progress); Le Monde advantage “Créateur-né” as Art system stub (VOEU_MONDE_ART, never triggers in demo).

LOW: L-1 OnExploit event description corrected (“ALL dice even” not “6+ excess successes”). L-2 Version header updated to v8.3. L-3 Changelog updated with v8.2 + v8.3 entries.
v8.2 (2026-02-24) — v8.2 Expert Review: Medium + Low Priority Fixes
CRITICAL: CR-1 PEs restoration at Camelot contradiction resolved (PS/PA/PE restored, PEs unchanged). CR-2 Meta-Armor Base Stats Table added to SPEC-02.7 with full armor data (PA, PE, CdF, Gen, Slots, Base Overdrives).

MEDIUM: M-1 Module vs innate ability separation clarified. M-2 Complete Haut Fait table (14 entries from tabletop source). M-3 Motivation Template Pool with 10 minor + 5 major templates (SPEC-12.1). M-4 Ghost detection terminology standardized (automatic successes, not extra dice). M-5 Points de Contact future design note.
LOW: L-1 Agent-Readiness Scorecard refreshed to v8.2 values (9.5/10). L-2 Firing beyond range simplified (out-of-range = greyed out, no penalty system). L-3 Ranger column preserved as next planned expansion class.
v8.1 (2026-02-23) — v8 Expert Review Remediation
CRITICAL: CR-1 Pénétrant CdF double-subtraction bug (SPEC-06). CR-2 Priest weight column added, Ranger kept as future class (SPEC-02). CR-3 Exploit added as SPEC-05 excess successes exception. CR-4 Global Math Rules added (round down, divide-then-subtract, successive doubling=tripling). CR-5 OnFoldTriggered corrected to PA=0 (SPEC-27).
HIGH: H-1 Bande attack procedure added, violence-only damage (SPEC-11). H-2 ANTI_VEHICULE future-proofing note. H-3 Despair events added to SPEC-27. H-4 Barrage targeting constraints. H-5 Charge Brutale routes through ArmourLayerResolver. H-6 Anathème to-hit procedure clarified.

LOW: L-1 AC numbering convention note. L-2 Flashbang 0-damage clarified (effects only). L-3 XIII+XVI incompatibility check added to SPEC-02 generation. L-4 PEs do NOT recover at Camelot in demo. L-5 Enemy Exploit = bonus damage dice (same as knight Exploit).
v8.0 — Despair, Weighted Distribution, Grenade Resolution, Barrage Rewrite
Added SPEC-07 Despair system (trigger, hostile AI, stabilization, exit, permanence). Armor Class Weighted Distribution Table (SPEC-02). Grenade resolution procedure. Barrage effect rewritten as suppress-mode alternative. Knight Defense vs Anathème chain. PEs Recovery on Anathème Kill. Global Duration Definition.
v7.0 — v7 Expert Review: 20 findings resolved
5 Critical fixes (Hit Threshold, Destructeur/Meurtrier Pseudocode, Faune Majeur auto-successes, Nocte Bande targeting, NanoC Cover). 15 additional findings (H/M/L) resolved. Full issue tracking in Part 3.
v1.0–v6.0 — Initial specification + iterative development
v1: Initial 27 SPEC framework. v2: DD positioning, Rogue Ghost, enemy AI. v3: MDD findings integration. v4: Nod/Grenade, Combo Roll, DD rank system. v5: Expert review integration (SPEC 18–25). v6: Tarot 22-card pool, 5-card draw, Haut Fait correction.

# PART 3 — v7 EXPERT REVIEW: ISSUE TRACKING
Independent expert review of Technical Specification v6. 20 issues identified across 25 SPEC sections. Organized by severity using the same scale as Part 1. All 20 issues were identified and resolved in v7. This section serves as an audit trail.

## 3.1 Issue Summary
20 findings (5C + 5H + 5M + 5L). ALL 20 RESOLVED in v7. 0 remain OPEN. Document is agent-ready. Critical and High findings must be addressed before development begins. Medium findings should be resolved during Phase 1-2 implementation. Low findings can be deferred to polish.


| Severity | Count | Resolution Requirement |
| --- | --- | --- |
| CRITICAL | 5 | ALL 5 RESOLVED in v7. No longer blocking. |
| HIGH | 5 | Cross-system ambiguity. Will cause subtle bugs if unresolved. |
| MEDIUM | 5 | Requires agent assumptions. May differ from design intent. |
| LOW | 5 | Polish, edge cases, missing hooks. Non-blocking. |




| v7-C1 | Affected: SPEC-04 (§5.1 step 8), SPEC-18.9 (Step 6). Impacts every combat roll in the game. Recommendation: |
| --- | --- |



| v7-C2 | Affected: SPEC-06 (Armour Layer Resolver), SPEC-20.1 (Destructeur/Meurtrier definitions), SPEC-05 (Damage Resolver step 8). Recommendation: Rewrite SPEC-06 as an explicit 8-step pseudocode function: (1) Apply CdF, (2) Apply Bouclier, (3) CHECK Destructeur trigger — if remaining > 0 AND target has PA: roll +2D6, add to remaining, (4) Apply PA absorption, (5) CHECK Meurtrier trigger — if remaining > 0 after PA: roll +2D6, add to remaining, (6) Apply PS damage. Include worked example with both effects on same hit. |
| --- | --- |



| v7-C3 | SPEC-18.3 defines Majeur (N) as ‘N automatic successes’ (flat bonus to success count, not extra dice). SPEC-21.3 Note states: ‘Bête Majeur (6) means +6 dice offensively.’ These are mechanically different — +6 auto-successes is guaranteed, while +6 dice yields ~3 successes on average. The tabletop uses automatic successes. This error would halve the Faune’s effective offensive power if an agent follows the Note instead of SPEC-18.3. Affected: SPEC-21.3 (Faune stat block note), SPEC-18.3 (Aspects Exceptionnels rules). Recommendation: Correct the Note in SPEC-21.3 to read: ‘Bête Majeur (6) means +6 automatic successes offensively AND incoming attacks using Bête-based characteristics deal half damage.’ Remove the word ‘dice’. |
| --- | --- |



| v7-C4 | SPEC-21.1 (Nocte stat block) lists ‘Weapon (AoE): Anti-véhicule damage’ and includes a full AI profile (targeting: NEAREST_RANK, aggression: 3). SPEC-11 states unambiguously: ‘Bandes do not attack and never roll to hit. Débordement is the Bande’s ONLY offensive mechanic.’ An agent encountering these contradicting instructions will either implement attack logic for Nocte (violating SPEC-11) or ignore the weapon entry (losing the Anti-Véhicule trait). The key question: does Anti-Véhicule apply to Débordement damage? Affected: SPEC-21.1 (Nocte stat block), SPEC-11 (Bande Controller). Recommendation: Remove the ‘Weapon’ and ‘AI’ fields from the Nocte stat block. If Anti-Véhicule is intended to apply to Débordement damage, add an explicit ‘Débordement Tags’ field (e.g., debordementTags: [ANTI_VEHICULE, IGNORE_CDF]) and update SPEC-11 to check for Bande-level tags when applying Débordement. |
| --- | --- |



| v7-C5 | Affected: SPEC-17.3 (NanoC Menu), SPEC-03 (Cover System), SPEC-20.4 (Tir en Sécurité, Artillerie). Recommendation: |
| --- | --- |




| v7-H1 | SPEC-01 Derived Value Formulas contains an unresolved decision marker: ‘DECISION REQUIRED: OD included for Defense/Reaction/Initiative means add OD of only the highest Characteristic (not all Aspect ODs). Recommended default: single highest + its OD.’ This is the most fundamental combat formula in the game. Every attack roll, every defense check, every initiative calculation depends on it. An agent cannot implement combat without this resolved. Affected: SPEC-01 (KnightBase Derived Values). Propagates to SPEC-04, SPEC-05, SPEC-15, SPEC-18. Recommendation: Resolve immediately. The tabletop confirms: Defense = highest BÊTE characteristic + that characteristic’s OD. Same pattern for Reaction (MACHINE) and Initiative (MASQUE). Remove the DECISION REQUIRED tag and state the formula definitively. |
| --- | --- |



| v7-H2 | SPEC-17.2 Shrine grants +6 CdF to ‘all inside (stacks). 6m dome.’ The DD rank system has no concept of spatial distance or ‘inside a dome.’ An agent must know: does Shrine affect all allies at the same rank as the Paladin? All allies within Courte range? All 4 allies regardless of rank? The entry also says ‘Immobile — Cannot move once placed’ but ranks are abstract positions, not physical locations. Affected: SPEC-17.2 (Shrine), SPEC-03 (Position System). Recommendation: |
| --- | --- |



| v7-H3 | Affected: SPEC-18.10 (Peur rules), SPEC-21.4 (Behemot), SPEC-21.5 (Ours Corrompu). Recommendation: |
| --- | --- |



| v7-H4 | Affected: SPEC-14 (Boss Phase Controller), SPEC-21.5 (Ours Corrompu stat blocks), SPEC-22 (Mission Structure boss encounter). Recommendation: |
| --- | --- |



| v7-H5 | SPEC-21.2 Bestian Anathème capacity says: ‘Attacker may choose to inflict damage on PEs instead of PS. CdF still applies, PA does not.’ The tabletop has the MJ (attacker/GM) decide. In a digital game with no GM, the AI must have a deterministic decision rule. Should the AI always target PEs? Only when PEs < threshold? Target whichever is lower? Without this rule, an agent will either hardcode an arbitrary choice or leave it undefined. Affected: SPEC-21.2 (Bestian), SPEC-18.7 (EnemyAI profile), SPEC-21.5 (Ours Corrompu also has Anathème). Recommendation: Add an ‘anathemeTarget’ field to EnemyAI: enum {ALWAYS_PES, ALWAYS_PS, PREFER_LOWER, SMART}. Default Bestian to PREFER_LOWER (target whichever pool is lower to maximize pressure). Ours Corrompu Souffle ténébreux: ALWAYS_PES (narrative: darkness attacks hope). |
| --- | --- |




| v7-M1 | SPEC-24.1 states that only 12 of 22 Major Arcana cards have complete mechanical definitions. The remaining 10 ‘provide stat bonuses only with no advantage/disadvantage.’ SPEC-02 step 4 requires selecting 2 advantages and 1 disadvantage from 5 drawn cards. If a knight draws 3+ undefined cards, they cannot fill all required slots. No fallback rule exists. Affected: SPEC-24 (Tarot), SPEC-02 (Generation Pipeline step 4). Recommendation: Add fallback rule: ‘If fewer than 2 defined advantages or 1 defined disadvantage are available from drawn cards, redraw from the 12-card defined pool until slots are filled. Stat bonuses from all 5 original draws still apply.’ Or: complete the remaining 10 card definitions before demo ship. |
| --- | --- |



| v7-M2 | SPEC-18.9 states enemies defend with ‘Aspect/2 (round down) + Exceptionnel bonus’ and that this is the ‘pre-calculated Defense/Reaction in the stat block.’ Bestian has Bête 8, Bête Mineur (2), listed Defense 4. If Defense = Bête/2 + Mineur = 4+2 = 6, but the listed value is 4 (= Bête/2 only). Either the listed values exclude Exceptionnel (and auto-successes are added at roll time), or the formula description is wrong. An agent will double-count if the stat block already includes the bonus. Affected: SPEC-18.9 (Enemy Attack Procedure Steps 5-6), SPEC-21 (all enemy stat blocks). Recommendation: Clarify that listed Defense/Reaction values in stat blocks represent Aspect/2 only. Exceptionnel auto-successes are added at attack roll time (Step 5), NOT baked into the stat block threshold. Add a note: ‘Stat block Defense/Reaction = base opposition score. Exceptionnel bonuses add to attacker’s success count, not to the stat block value.’ |
| --- | --- |



| v7-M3 | The spec covers game mechanics exhaustively but provides no screen flow diagram or UI state machine. An agent building UI needs to know: deployment screen layout, initiative timeline display format, injury/status indicator placement, branch selection (Node 2) presentation, combat HUD layout, Camelot Hub screen organization. Without this, the UI team will make assumptions that may conflict with gameplay requirements. Affected: All SPECs (UI layer), SPEC-25 (Camelot Hub), SPEC-03 (Deployment Phase), SPEC-15 (Turn Queue), SPEC-22 (Mission Structure). Recommendation: |
| --- | --- |



| v7-M4 | Affected: SPEC-04 (Action Economy Table), SPEC-15 (Turn Structure). Recommendation: Add an explicit design note: ‘Assist range (any) vs Nod range (Courte) is intentional per tabletop rules. Assistance is a morale/mental action; Nods require physical proximity to administer.’ This prevents agent ‘correction’ and informs UI tooltip text. |
| --- | --- |



| v7-M5 | SPEC-18.7 defines TargetPriority and RankBehavior enums but provides no step-by-step decision tree. Agent needs: Does enemy move first then attack, or attack first? If ADVANCE but target already in range, does it still advance? If HOLD but no target in range, does it break HOLD? If multiple targets match priority, which wins? If FLANK, what exact behavior occurs in the 4-rank system? Without a flowchart, each agent developer will implement different AI behaviors. Affected: SPEC-18.7 (EnemyAI), SPEC-21 (all enemy stat blocks with AI profiles). Recommendation: Add Section 18.11: Enemy AI Decision Flowchart. Pseudocode: (1) Check weapon range vs all valid targets, (2) If target in range: attack (select by TargetPriority), (3) If no target in range: execute RankBehavior to get closer, (4) If now in range: attack, (5) Else: hold. Define FLANK as ‘prefer targeting Rank 3-4 knights; move to Rank 1 if needed for contact range.’ |
| --- | --- |




| v7-L1 | SPEC-08 (Injury & Agony) lists ‘1,5 Hémorragie: 3-turn countdown.’ SPEC-23 (Complete Injury Table) listed ‘Hémorragie Interne: Dead in 10 turns unless healed.’ [v7] SPEC-08 is authoritative (3 turns). SPEC-23 updated to match. Affected: SPEC-08 (Special Injuries), SPEC-23.2 (Column 1, Row 5). Recommendation: Update SPEC-23 to match SPEC-08: ‘Hémorragie: 3-turn countdown.’ SPEC-08 is authoritative. |
| --- | --- |



| v7-L2 | SPEC-05 worked example (melee attack) describes Marteau-Épieu as ‘3D6 + Force, Lesté’ and calculates Force×2. The actual weapon profile in SPEC-19.3 shows Marteau-Épieu with effects: Perce Armure 40 (contact) — no Lesté. Either the worked example should use a different weapon, or the weapon profile is missing the Lesté tag. Affected: SPEC-05 (Worked Example), SPEC-19.3 (Marteau-Épieu stat block). Recommendation: Change the SPEC-05 worked example to use a hypothetical weapon ‘Masse d’Armes (3D6 + Force, Lesté)’ or add a note: ‘This example uses a Lesté variant for illustration. Default Marteau-Épieu does NOT have Lesté.’ |
| --- | --- |



| v7-L3 | Several numbered lists in the document have incorrect starting indices. For example, SPEC-02 Algorithm steps are numbered 5-14 instead of 1-10. Acceptance criteria across SPECs use inconsistent numbering ranges. While not functionally blocking, this creates confusion when referencing specific acceptance criteria by number during development and QA. Affected: Multiple SPECs (SPEC-02, SPEC-03, SPEC-04, SPEC-05 and others). Recommendation: Re-number all acceptance criteria sequentially across the entire document (AC-001 through AC-175) or restart numbering within each SPEC (e.g., SPEC-02-AC1 through SPEC-02-AC3). Use a consistent prefix to avoid ambiguity. |
| --- | --- |



| v7-L4 | The spec defines complete mechanical chains for combat, damage, injury, and state transitions, but includes no audio or visual effect trigger points. For combat feel (hit impacts, Exploit celebrations, Fold state alarm, Agony warning, Débordement rumble, Phase transition cinematic), the agent team needs at least placeholder event hooks in the damage chain and state transitions. Affected: SPEC-05 (Damage Chain), SPEC-06 (Armour Resolver), SPEC-08 (Agony), SPEC-10 (Fold), SPEC-14 (Boss Phase). Recommendation: Add a SPEC-27: Audio/VFX Event Hooks, listing all trigger points with placeholder event names (e.g., OnHitConfirmed, OnExploit, OnFoldTriggered, OnAgonyEntered, OnPhaseTransition, OnPermadeath). This allows the agent to fire events even if actual assets are not yet available. |
| --- | --- |



| v7-L5 | L1 (resolved) notes the tabletop difficulty gap at 7-8 is intentional, but no mapping exists for how digital encounters use difficulty thresholds. If enemies or narrative events trigger tests at specific difficulty levels (e.g., Peur tests, Perception checks to discover Point Faible), the agent needs to know which numeric threshold each difficulty label maps to in the Combo Roll system. Affected: SPEC-04 (skill tests), SPEC-18.10 (Peur), SPEC-22.3 (narrative events). Recommendation: Add a difficulty-to-threshold mapping table: Facile(1)=1 success, Normale(2)=2, Ardue(3)=3, Difficile(4)=4, Éprouvante(5)=5, Délicate(6)=6, Herculéen(9)=9. Include a note that 7-8 are unused in the demo. |
| --- | --- |



## 3.6 Agent-Readiness Scorecard

| Category | Score | Notes |
| --- | --- | --- |
| Completeness | 9 / 10 | [v8.8] 32 SPECs (single merged SPEC-01 KnightBase, SPEC-28–32 Agent Handoff Addendum). All 81 review findings across 8 review passes resolved. Injury table 100% complete (all 24 digital effects filled from tabletop source). Haut Fait table complete with standalone data struct.ighting. All 10 MN templates verified. Combat style table fully populated. |
| Consistency | 9.5 / 10 | [v8.8] All consistency issues resolved across 8 review passes. PEs penalty formula canonical (max(0, 10−PEs)). Style gate restrictions corrected (Puissant=Lourd, Pilonnage=Deux Mains). Implant PEs penalty documented. No stale references.CriticalHit removed). No redundant fields (TriggerEvent ev removed). |
| Implementability | 9.5 / 10 | [v8.8] Worked examples, pseudocode, and 195+ acceptance criteria. Every injury has deterministic digital effect. All combat styles have prose + edge cases. HautFaitData struct extractable. Demo weapon availability for each style documented.ambiguous. Combat styles fully specified with restrictions. All parameters specified. |
| Tabletop Fidelity | 9 / 10 | [v8.4] Faithful adaptation. 9 of 12 tabletop Blasons adapted for demo (Le Dragon deferred for scoping). Code d'Honneur mapped to 6 feasible commandments. Le Cheval/Le Corbeau/Le Dragon deferred pending dependent systems. |
| Agent-Readiness | 9.5 / 10 | [v8.7] 71 total issues across 7 review passes, ALL RESOLVED. No formula contradictions, no logic bugs in canonical code, no missing content. Combat styles fully specified. Haut Fait generation complete with weighted mechanics. Agent team can scaffold from Phase 1 dependency map with full confidence. No known blockers. |



End of Part 3 — v8.7 Expert Review. 20 original v7 issues (5C + 5H + 5M + 5L): ALL RESOLVED. v8.1 review (2C + 4H + 5M + 3L): ALL RESOLVED in v8.2. v8.3 review (2H + 5M + 3L): ALL RESOLVED in v8.3. v8.4 review (1H + 5M + 4L): ALL RESOLVED in v8.4. v8.5 review (3H + 5M + 4L): ALL RESOLVED in v8.5. v8.6 review (1C + 2H + 3M + 3L): ALL RESOLVED in v8.6. v8.7 review (1C + 1H + 5M + 4L): ALL RESOLVED in v8.7. v8.8 review (1H + 5M + 4L, 1 false positive): ALL RESOLVED in v8.8. Total issues tracked across all reviews: 81. All 81 RESOLVED. Document is agent-ready for development handoff.
Below is a drop-in “Agent Handoff Addendum” written in the same style as your SPEC sections. It’s designed to remove implementation drift for an AI agent team, especially around determinism, authoring formats, state modeling, UI “must show,” and Unity architecture.

