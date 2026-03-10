# KNIGHT: INTO THE DARKNESS — Technical Specification v8.8
**March 2026** — Split into modular files for AI agent development. All 31 review findings resolved.
> **How to use this index:** Read this file first. Each file below is self-contained for its domain. The "Dependencies" header in each file lists which other files you need to load alongside it. The Dependency Map (below) shows build order.
> 
> **Version tags (`[v2]`, `[v7]`, `[v8.9]`, etc.):** These are historical annotations indicating when a rule was added or last changed. **Agents should ignore version tags** — treat ALL current text as authoritative regardless of tag. The tags exist solely for the human design team to track change history. A future major revision will consolidate these into a separate changelog file.
---
## File Manifest
| File | SPECs | Domain | Description |
|------|-------|--------|-------------|
| [01_KnightDataModel.md](./01_KnightDataModel.md) | 01, 02, 02.5, 02.7, 02.8, 02.9 | `Scripts/Data` | Knight schema + procedural generation pipeline (archetypes, Haut Fait, Blason) |
| [02_PositionAndTurns.md](./02_PositionAndTurns.md) | 03, 15 | `Scripts/Combat` | DD rank system, gap formula, cover, movement, initiative queue |
| [03_CombatRolls.md](./03_CombatRolls.md) | 04, 05, 09 | `Scripts/Combat` | Combo Roll, damage resolver, combat styles, heroism |
| [04_ArmorAndState.md](./04_ArmorAndState.md) | 06, 07, 08, 10 | `Scripts/Combat` | Armour layers, Espoir/Despair, injury, fold state |
| [05_ArmorAbilities.md](./05_ArmorAbilities.md) | 17 | `Scripts/Combat` | Warrior Types, Paladin Shrine/Watchtower, Priest NanoC/Mechanic, Rogue Ghost |
| [06_EnemySystem.md](./06_EnemySystem.md) | 11, 14, 18 | `Scripts/Combat` | Enemy data model, AI, bande controller, boss phases, Charge Brutale, Peur |
| [07_WeaponsAndEffects.md](./07_WeaponsAndEffects.md) | 19, 20 | `Scripts/Data` | Weapon profiles, demo arsenal, effects glossary, dual-wield |
| [08_ContentData.md](./08_ContentData.md) | 12, 21, 23 | `Assets/Data` | Enemy roster, injury table, motivation templates |
| [09_TarotSystem.md](./09_TarotSystem.md) | 24 | `Assets/Data` | 22 Major Arcana: advantages, disadvantages, stat bonuses |
| [10_MissionAndHub.md](./10_MissionAndHub.md) | 13, 22, 25 | `Scripts/Meta` | Mission structure, save system, Camelot hub, campaign loop |
| [11_UIAndPresentation.md](./11_UIAndPresentation.md) | 26, 27, 31 | `Scripts/UI` | UI screen flow, audio/VFX hooks, minimum info contract |
| [12_ArchitectureGuardrails.md](./12_ArchitectureGuardrails.md) | 16, 28, 29, 30, 32 | `Scripts/Core` | RNG, encounter schema, combatant state, project conventions, dependency map |
| [13_NodSystem.md](./13_NodSystem.md) | 33 | `Scripts/Combat` | Nod consumable recovery items: types, usage rules, Fold/Agony/Hémorragie interactions |
| [14_Constants.md](./14_Constants.md) | — | `Scripts/Data` | Consolidated enum & constant reference for all SPECs (read-only, no new logic) |
| [15_IntegrationTests.md](./15_IntegrationTests.md) | — | `Tests` | End-to-end combat scenarios for cross-system integration testing (5 scenarios) |
**Archived (no implementation value):**
| File | Content |
|------|---------|
| [archive/Part1_MDD_Review.md](../archive/Part1_MDD_Review.md) | Part 1 — MDD Review Findings (all resolved) |
| [archive/Part3_IssueTracking.md](../archive/Part3_IssueTracking.md) | Part 3 — Expert Review Issue Tracking + Changelog |
---
## Global Rules (from SPEC-03)
These apply across ALL systems:
- **PG (Points de Gloire):** Meta-currency earned through play, spent at Camelot Hub (SPEC-25). Used for implants, therapy, equipment, and evolutions.
- **Duration:** "Duration: 1 turn" = persists from activation until start of activating combatant's next turn.
- **Math:** All divisions round down. Division/Multiplication always before Addition/Subtraction. Successive doubling = tripling (not ×4). [v8.9] **Example:** If two separate effects both say "double damage" on the same hit, the combined multiplier is ×3 (not ×4). No demo mechanic currently produces successive doubling — this rule exists as a global safeguard for future content where multiple multiplicative effects may stack.
- **Hit threshold:** Successes must **strictly exceed** Defense (melee) or Reaction (ranged). Equal = miss.
- **Defense/Reaction floor:** Defense and Reaction values floor at **0** after all modifiers (Barrage, Agressif, Despair, Watchtower halving, rank modifiers, injuries, etc.). Negative values are clamped to 0. A combatant with Defense/Reaction 0 is hit by any attack that rolls at least 1 success (since 1 > 0).
---
## System Dependency Map (SPEC-16)
[v8.9] **Canonical source:** [12_ArchitectureGuardrails.md](./12_ArchitectureGuardrails.md) §SPEC-16. The full dependency map with version history is maintained there. Do NOT duplicate here — refer to the canonical source to prevent desync.

**Quick reference (build phases):**
- **Phase 1 (Data):** SPEC-01, 02, 18, 19, 20, 24
- **Phase 2 (Position):** SPEC-03, 15
- **Phase 3 (Core Combat):** SPEC-04, 07, 17, 33
- **Phase 4 (Damage):** SPEC-05, 06
- **Phase 5 (Consequences):** SPEC-08, 09, 10, 23
- **Phase 6 (Entities):** SPEC-11, 12, 21
- **Phase 7 (Mission):** SPEC-13, 14, 22
- **Phase 8 (Meta):** SPEC-25
---
## Cross-Reference Convention
All cross-references use the format: `See [SPEC-NN](./filename.md)`.
When an agent needs to implement a system, it should:
1. Read this index
2. Load the target file
3. Load all files listed in that file's "Dependencies" header
4. Load [14_Constants.md](./14_Constants.md) for all enum/constant definitions

---

## [v8.9] Agent Task Boundaries
Maps each SPEC to an agent role for parallel development. Agents must respect the dependency phases (SPEC-16) — do not start a higher-phase task until its dependencies are complete.

| Agent Role | SPECs Owned | Files to Load | Output |
|-----------|-------------|---------------|--------|
| **Data Agent** | 01, 02, 02.5, 02.7, 02.8, 02.9, 24 | 01, 09, 14 (constants) | KnightBase SO, Generation Pipeline, Tarot data, Archetype/HautFait/Blason SOs |
| **Weapon & Content Agent** | 19, 20, 21, 23 | 07, 08, 14 | WeaponProfile SOs, Effect resolver stubs, Enemy roster SOs, Injury table data |
| **Combat Core Agent** | 03, 04, 05, 06, 15 | 02, 03, 04, 07, 14 | Position system, Combo Roll, Damage resolver, Armour layer resolver, Turn queue |
| **Combat State Agent** | 07, 08, 09, 10, 33 | 04, 13 (Nods), 14 | Espoir/Despair, Injury & Agony, Heroism, Fold state, Nod system |
| **Armor Abilities Agent** | 17 | 05, 01, 02, 03, 14 | Warrior Types, Paladin Shrine/Watchtower, Priest NanoC/Mechanic, Rogue Ghost |
| **Enemy AI Agent** | 11, 14, 18 | 06, 07, 08, 14 | Enemy data model, Bande controller, Boss phases, AI decision tree, Charge Brutale, Peur |
| **Mission & Hub Agent** | 12, 13, 22, 25 | 10, 08, 14 | Motivation detector, Save system, Mission structure, Camelot hub |
| **Architecture Agent** | 16, 28, 29, 30, 32 | 12, 14 | RNG system, Encounter schema, Combatant state model, Project conventions |
| **UI Agent** | 26, 27, 31 | 11, 14 | Screen flow, Audio/VFX hooks, Minimum info contract |

**Shared dependencies (all agents must load):**
- [00_INDEX.md](./00_INDEX.md) — this file (global rules, task boundaries)
- [14_Constants.md](./14_Constants.md) — all enums and constants
- [12_ArchitectureGuardrails.md](./12_ArchitectureGuardrails.md) — project conventions, RNG contract, state isolation rules

**Boundary rules:**
1. Each SPEC has exactly ONE owning agent. No shared ownership.
2. Cross-SPEC dependencies are read-only: an agent may READ another agent's output but must NOT modify it.
3. If an agent discovers a contradiction between its SPECs and a dependency, it must flag it (not silently resolve).
4. The Data Agent and Weapon & Content Agent (Phase 1) must complete first — all other agents depend on their SOs.
5. Combat Core Agent and Combat State Agent can work in parallel (Phase 3–5), but Combat State depends on Combat Core's SPEC-05/06 outputs.
