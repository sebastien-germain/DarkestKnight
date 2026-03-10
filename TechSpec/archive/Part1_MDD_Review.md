# KNIGHT: INTO THE DARKNESS — Part 1: MDD Review Findings
> **Archive** — All 20 findings resolved. This file has no implementation value.
> See [00_INDEX.md](../split/00_INDEX.md) for the active specification.
---
# PART 1 — MDD REVIEW FINDINGS
Line-by-line review of MDD v1.3. All 20 findings (5C + 5H + 6M + 4L) have been resolved. Every finding shows a green RESOLVED box. This document is complete.

## 1.1 Severity Legend

| Severity | Definition | Impact |
| --- | --- | --- |
| CRITICAL | Blocks implementation. Agent cannot produce correct code. | Build fails or wrong behavior. |
| HIGH | Ambiguity across systems. Agent may implement inconsistently. | Subtle bugs, cross-system mismatches. |
| MEDIUM | Requires agent assumptions that may differ from intent. | Non-fatal but potentially wrong. |
| LOW | Polish, completeness, or edge cases. | Minor, no functional impact. |


## 1.2 Critical Findings (ALL RESOLVED)


| C1+C2 | Damage = SUM all weapon dice face values + flat bonuses (Force, OD×3, effects). This is separate from the attack roll (count evens). Excess successes do nothing by default. Only Assistance à l'Attaque converts excess successes to +1 flat damage each. Dégâts Maximum (1 Héroïsme) maximizes all weapon dice to 6. See SPEC-05 for full chain. |
| --- | --- |



| C3 | Before first turn: player places each knight on P1–P4. Enemy positions from encounter data (EnemySpawn SO). Default suggestion: melee-oriented (Warrior/Barbarian) at P1–P2, ranged-oriented (Paladin/Priest/Rogue) at P3–P4. See SPEC-03. |
| --- | --- |



| C4 | PA=0: Guardian Suit (PA 5, CdF 5). OFFLINE: ODs, modules, signature abilities, PE spending. ACTIVE: weapons, Nods, free actions. PE pool frozen (not lost). Exit: any PA restoration (Nod d'Armure, Priest Mechanic) restores armor and reactivates all systems. See SPEC-10. |
| --- | --- |



| C5 | 2 per class × 4 classes = 8 knights. Warrior ×2, Paladin ×2, Priest ×2, Rogue ×2. Remove ambiguous 'remaining' language from §3.1.1. See SPEC-02. |
| --- | --- |


## 1.3 High Findings (ALL RESOLVED)


| H1 | Violence damages Bande Cohesion only. Never reduces knight PEs. Remove all MDD references to 'Violence damages PEs for individual knights.' |
| --- | --- |



| H2 | Added to §5.6 Step 2b: floor(paAbsorbed/5) = PS lost. Disabled by Infatigable. Mirrored in §10.3. |
| --- | --- |



| H3 | Bandes do not attack and never roll to hit. Débordement hits ALL enemies automatically each turn. No position restriction. Knights target Bande from any position with Courte+ range. |
| --- | --- |



| H4 | Added to §5.1: Combo chosen per-roll, may change each action, must differ from Base Characteristic. |
| --- | --- |



| H5 | Warrior Types, Paladin Shrine/Watchtower, Priest NanoC/Mechanic, Rogue Ghost — all with full stat blocks. |
| --- | --- |


## 1.4 Medium Findings (ALL RESOLVED)


| M1 | Tabletop confirms: 'Les résultats sont conservés durant toute la phase de conflit.' Initiative = 3D6 + Initiative score, rolled once at combat start. Results persist until combat ends. Trop prudent: 2D6 instead. Bandes always initiative 1. See SPEC-15. |
| --- | --- |



| M2 |  |
| --- | --- |



| M3 | Tabletop entraide has no positioning constraint. For the demo: assistance works from any position. Each assisting knight spends 1 Movement Action and rolls 1 Characteristic (not used by others). Successes added to the aided knight's roll. Max 3 assistants. |
| --- | --- |



| M4 | [v4] No automatic healing between combat nodes. PS, PA, PE, and PEs carry over exactly as-is. Knights may use Nods during node transitions (costs no action outside combat). This creates maximum attrition pressure and makes resource management critical. Matches roguelite design philosophy. |
| --- | --- |



| M5 |  |
| --- | --- |



| M6 | [v4] MAJOR CHANGE: Exactly 1 combatant per position. 4 positions per side = max 4 knights, max 4 individual enemies. Movement = swap with adjacent ally or shift into empty slot. Forced displacement chain-swaps. Dead combatant leaves empty slot. Squad: 4 knights selected from 8-knight roster at mission start, no changes mid-mission. Reinforcements only spawn into empty enemy slots. See SPEC-03 rewrite. |
| --- | --- |


## 1.5 Low Findings (ALL RESOLVED)


| L1 | Tabletop scale: Facile(1), Normale(2), Ardue(3), Difficile(4), Éprouvante(5), Délicate(6), Herculéen(9). Gap at 7–8 is by design. Add tooltip/note in UI confirming. [v7] NOTE ON NUMBERING: Numbered lists throughout this document (acceptance criteria, algorithm steps) may show non-sequential indices due to DOCX conversion artifacts. Agents should treat list items by ordinal position, not by displayed number. E.g., if steps show 5-14, read as steps 1-10. [v7] Difficulty-to-Threshold Mapping (for agent implementation):   Facile       = 1 success required   Normale      = 2 successes required   Ardue        = 3 successes required   Difficile    = 4 successes required   Éprouvante   = 5 successes required   Délicate     = 6 successes required   (7-8)        = UNUSED in demo — reserved for future content   Herculéen    = 9 successes required These thresholds apply to all test types: Peur resistance, Savoir checks, social tests. When a mechanic says 'difficulty reduced by 1 level', subtract 1 from the required successes (minimum 1). |
| --- | --- |



| L2 | Push past Rank 4 (back): clamped to Rank 4. Push past Rank 1 (front): clamped to Rank 1. Cannot push across center line. Already reflected in SPEC-03 v4. |
| --- | --- |



| L3 |  |
| --- | --- |



| L4 | Chargeur X: X uses per mission. No mid-mission reload. Auto-refill at Camelot. Grenades: 5/mission, same rule. Added to SPEC-04 notes. |
| --- | --- |


