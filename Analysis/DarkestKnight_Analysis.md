# 🏰 DarkestKnight — Complete Project Analysis

## 1. Project Identity

**DarkestKnight** is a **2D turn-based tactical RPG** built in **Unity**, adapting the French tabletop RPG **"Knight"** into a digital format. The game fuses the **Darkest Dungeon rank-based positioning system** with the deep dice-pool combat mechanics of the Knight TTRPG, wrapped in a **roguelite attrition loop**.

- **Source Material:** Knight JDR (tabletop RPG) — [knight-jdr-systeme.fr](https://knight-jdr-systeme.fr/fr/)
- **Tech Spec:** v8.8 (dated today, 2026-02-26) — 32 SPEC sections, 81 issues tracked across 8 expert review passes, all resolved
- **Status:** Agent-ready specification, **no Unity project code exists yet** — this is a pure design/spec workspace

---

## 2. Core Game Pillars

| Pillar | Description |
|--------|-------------|
| **Darkest Dungeon Positioning** | 4v4 linear rank system (Rank 1–4 per side). Single-occupancy, adjacent swaps, forced displacement |
| **Knight TTRPG Combat** | D6 dice pools, Combo Roll (Base + Combo characteristic), even = success, Exploit/Failure Critique |
| **Roguelite Attrition** | No rest between encounter nodes. Resources carry over. Permadeath. 8-knight roster → pick 4 per mission |
| **Morale (Espoir/PEs)** | Darkest Dungeon-inspired stress system. PEs < 10 = dice penalty. PEs = 0 = Despair (hostile AI) |
| **4 Armor Classes** | Warrior (Types), Paladin (Shrine/Watchtower), Priest (NanoC/Mechanic), Rogue (Ghost) |

---

## 3. Specification Architecture (32 SPECs)

### Foundation Layer
| SPEC | System | Complexity |
|------|--------|------------|
| **01** | Knight Data Model (KnightBase) | Core — all systems read/write this |
| **02** | Procedural Knight Generation (10 steps) | High — Archetypes, Tarot, Haut Fait, Blasons, Motivations, Meta-Armor |
| **18** | Enemy Data Model (EnemyBase) | Core — 5-tier system (Bande → Patron) |

### Combat Engine
| SPEC | System | Key Mechanic |
|------|--------|--------------|
| **03** | Position System (DD Ranks) | 4-rank linear track, cover system, forced displacement |
| **04** | Combo Roll System | D6 pool, Base+Combo, OD auto-successes, styles (Agressif/Défensif/Puissant/Pilonnage) |
| **05** | Damage Resolver | 8-step chain, Force bonus, Précision/Orfèvrerie, Dégâts Maximum |
| **06** | Armour Layer Resolver | CdF → Bouclier → PA → PS pipeline, PA passthrough, Destructeur/Meurtrier |
| **11** | Bande Controller | Off-track swarms, Débordement (auto-damage), Violence-only |
| **15** | Turn Queue & Initiative | FFX-style timeline, 3D6+Init, delay mechanic |
| **20** | Weapon Effects Glossary | 20+ effects (Perce Armure, Choc, Dispersion, Cadence, etc.) |

### State Systems
| SPEC | System | Stakes |
|------|--------|--------|
| **07** | Espoir (PEs) / Despair | Morale → hostile AI, stabilization mechanic |
| **08** | Injury & Agony | 46-entry injury matrix, Hémorragie countdown, permadeath |
| **09** | Heroism (0–6) | Mode Héroïque (spend all 6), Ignorer Agonie/Désespoir |
| **10** | Fold State | PA=0 → Guardian Suit, systems offline, Nod exit path |
| **12** | Motivation Event Detector | 10 minor + 5 major templates, event-bus-driven PEs recovery |

### Content & Meta
| SPEC | System | Scope |
|------|--------|-------|
| **14** | Boss Phase Controller | Ours Corrompu (2-phase T5 Patron) |
| **17** | Armor Ability System | Warrior Types, Paladin Shrine/Watchtower, Priest NanoC/Mechanic, Rogue Ghost |
| **19** | Weapon Data Model | 8 demo weapons + grenades + unarmed |
| **21** | Demo Enemy Roster | 5 enemy types: Nocte (T1), Bestian (T2), Faune (T3), Béhémot (T4), Ours Corrompu (T5) |
| **22** | Demo Mission Structure | "The Corrupted Den" — 5 nodes, branch at Node 2, boss at Node 5 |
| **23** | Complete Injury Table | Full 46-entry matrix with digital adaptations |
| **24** | Tarot Advantages/Disadvantages | Full 22-card Major Arcana, 2 advantages + 1 disadvantage per knight |
| **25** | Camelot Hub & Mission Loop | Between-mission hub, PG economy, campaign loop |

### Infrastructure
| SPEC | System | Purpose |
|------|--------|---------|
| **13** | Save System | Auto-save at nodes, no mid-combat saves, resume from node start |
| **26** | UI Screen Flow | Combat HUD, Deployment, Camelot Hub (minimum viable) |
| **27** | Audio/VFX Event Hooks | 25+ fire-and-forget event hooks |
| **28** | Deterministic RNG | xxHash32 + Xoshiro128**, no UnityEngine.Random, anti-savescum |
| **29** | Encounter Authoring (EncounterSO) | ScriptableObject-based encounter definitions |
| **30** | Combatant State Model | Canonical LifeState/ControlState/ActionState/ArmorState + IsSquadDefeated() |
| **31** | UI Minimum Information Contract | Mandatory roll breakdowns, combat log, intent clarity |
| **32** | Unity Architecture Guardrails | Single source of truth, data vs runtime separation, single-scene architecture |

---

## 4. Demo Scope

The spec defines a **playable demo (v1.6)** with:
- **8 procedurally generated knights** (4 armor classes × 2)
- **1 mission: "The Corrupted Den"** — 5 nodes, linear with one branch
- **5 enemy types** spanning all 5 tiers
- **2-phase final boss** (Ours Corrompu)
- **4 armor classes** with signature abilities (no evolutions)
- **8 weapons** + grenades + unarmed
- **Full Tarot system** (22 cards, advantages/disadvantages)
- **Camelot hub** (roster review, injury treatment, mission launch)
- **Campaign loop** until all 8 knights dead or mission cleared

---

## 5. Technical Constraints & Guardrails

| Constraint | Detail |
|-----------|--------|
| **Deterministic RNG** | xxHash32 seeds, Xoshiro128** PRNG, no `UnityEngine.Random` |
| **Single-scene architecture** | State machine-driven, no scene spaghetti |
| **Combat state isolation** | CombatSession ↔ HubSession isolation, RunState as bridge |
| **ScriptableObject authoring** | Knights, enemies, weapons, encounters all as SOs |
| **No MonoBehaviour Update() logic** | State machine ticks or coroutine managers only |
| **Headless testability** | Combat must run without UI for unit/integration testing |
| **Save anti-exploit** | Deterministic node seeds prevent quit-to-reroll |

---

## 6. Current Project State

| Aspect | Status |
|--------|--------|
| **Technical Specification** | ✅ Complete (v8.8, 81 issues resolved, agent-ready) |
| **Unity Project** | ❌ Not created yet — workspace contains only design documents |
| **Code** | ❌ None written |
| **Assets** | ❌ None (no sprites, audio, or UI art) |
| **PDFs** | ✅ 4 tabletop source books for reference (arsenal, bestiary, character creation, system) |

---

## 7. Key Design Strengths

1. **Exceptionally detailed spec** — Every system has pseudocode, worked examples, acceptance criteria, and edge cases
2. **Smart adaptation** — DD rank system elegantly replaces tabletop's free-form movement
3. **Deep tactical layer** — Combo characteristic choice, style selection, weapon effects, and positioning all interact meaningfully
4. **Meaningful attrition** — No rest between nodes creates genuine resource management tension
5. **Robust morale system** — Despair/Stabilization creates dramatic emergent narratives
6. **Deterministic architecture** — Enables fair save/load, reproducible bugs, and future competitive/replay features

## 8. Key Risks & Considerations

| Risk | Severity | Mitigation |
|------|----------|------------|
| **Mechanical complexity** | High | 20+ weapon effects × 5 armor layers × 5 enemy tiers = massive combinatorial surface. Needs extensive automated testing |
| **No art pipeline** | High | 2D assets (sprites, UI, VFX) are entirely unaddressed. Need art direction/sourcing strategy |
| **Scope creep** | Medium | Spec references "future content" (Ranger class, vehicles, Warmaster, relationship system). Must stay locked to demo scope |
| **Balance** | Medium | Procedural generation + many interacting systems = unpredictable balance. Needs simulation/playtesting framework |
| **French language content** | Low | All narrative/motivation text is in French. Localization strategy needed if targeting international audience |

---

## 9. Recommended Next Steps

1. **Initialize Unity project** with the SPEC-32 folder structure
2. **Build the data layer first** — ScriptableObjects for KnightBase, EnemyBase, WeaponProfile, EncounterSO
3. **Implement SPEC-28 (Deterministic RNG)** early — everything depends on it
4. **Build combat engine headless** — SPEC-04 (Combo Roll) → SPEC-05 (Damage) → SPEC-06 (Armour Layers) → SPEC-03 (Positions)
5. **Add state systems** — SPEC-07/08/09/10 (Espoir/Injury/Heroism/Fold)
6. **Wire up enemy AI** — SPEC-18 (Enemy model + AI decision flowchart)
7. **Build minimal UI** — SPEC-31 contract as guide
8. **Author demo content** — SPEC-22 mission structure as EncounterSO instances
9. **Integrate Camelot hub** — SPEC-25 campaign loop
10. **Art/audio pass** — Placeholder → final assets

---

*This is an ambitious, well-specified project. The specification quality is exceptional for an indie RPG — the level of rigor (worked examples, pseudocode, 81 tracked issues) means development can proceed with very low ambiguity. The primary challenge is execution volume: there are ~30 interlocking systems to implement before the demo is playable.*
