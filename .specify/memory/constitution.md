<!--
  Sync Impact Report
  ==================
  Version change: 0.0.0 (new) -> 1.0.0
  Modified principles: N/A (initial creation)
  Added sections:
    - Core Principles (9 principles)
    - Scope & Exclusions
    - Development Workflow
    - Governance
  Removed sections: N/A
  Templates requiring updates:
    - .specify/templates/plan-template.md: ✅ compatible (Constitution Check section exists)
    - .specify/templates/spec-template.md: ✅ compatible (no constitution-specific refs)
    - .specify/templates/tasks-template.md: ✅ compatible (phase structure aligns)
    - .specify/templates/commands/: ✅ no command files exist yet
  Follow-up TODOs: None
-->

# Knight: Into the Darkness Constitution

## Core Principles

### I. Tabletop Fidelity First

The tabletop source material in `/Context/` is the authoritative design
reference. All game mechanics, formulas, and behaviors MUST align with the
tabletop RPG "Knight" by Antre Monde Editions unless a deliberate deviation
is documented and justified for demo scope. When ambiguity arises, the
tabletop rules govern.

- Deviations from tabletop rules MUST be documented with rationale.
- Undocumented deviations are treated as bugs.
- `/Context/` files are read-only references; never modify them.

### II. Agent-Ready Specifications

All implementation MUST reference the TechSpec files in
`/TechSpec/specifications/`. These files contain complete mathematical
formulas, pseudocode, worked examples, and acceptance criteria.

- Implementers MUST NOT invent mechanics; implement exactly what the
  spec prescribes.
- Each SPEC file is the single source of truth for its domain.
- If a spec is ambiguous or incomplete, flag it for clarification
  before implementing a guess.

### III. Architecture Conventions (SPEC-32)

The project follows strict architectural patterns to ensure consistency
and maintainability:

- **Single Scene**: One Unity scene (`Main.unity`); `GameStateMachine`
  manages all state transitions.
- **State Isolation**: `RunState` (persistent across encounters) vs
  `CombatSession` (ephemeral per combat) MUST remain strictly separated.
- **Combat Authority**: `CombatManager` is the sole authority for combat
  logic. No other system may directly mutate combat state.
- **Command Flow**: UI sends commands -> `CombatManager` validates ->
  executes -> fires events -> UI subscribes. No direct UI-to-state
  mutation.
- **Naming Convention**: French enums (`AGRESSIF`, `DEFENSIF`), English
  field/variable names, French UI-facing strings, English class names.

### IV. Math Rules (SPEC-03)

All mathematical operations MUST follow these non-negotiable rules:

- All integer divisions floor (round down).
- Division and multiplication evaluate before addition and subtraction.
- Successive doubling equals tripling (x3, not x4).
- Hit threshold: successes MUST STRICTLY EXCEED defense/reaction value
  (not equal).
- Defense and Reaction values floor at 0 (never go negative).

### V. Data Architecture

All game data MUST be implemented as ScriptableObject (SO) instances.

- No hard-coded gameplay values in MonoBehaviours or C# classes.
- All tunable parameters (stats, thresholds, damage tables, weapon
  data) MUST live in SO assets.
- Runtime state MAY use plain C# objects, but static configuration
  MUST be SO-based.

### VI. Testing Discipline

Testing is mandatory and follows defined standards:

- Integration tests defined in `/TechSpec/specifications/15_IntegrationTests.md`
  MUST pass before a feature is considered complete.
- EditMode tests MUST validate deterministic RNG behavior per SPEC-28.
- Tests MUST use the worked examples from TechSpec as test vectors.
- All tests MUST be repeatable and deterministic.

### VII. Dependency Phase Order (SPEC-16)

Build order is strictly enforced across 8 phases:

1. **Phase 1** - Data (ScriptableObjects, enums, data structures)
2. **Phase 2** - Position (grid, range, zone systems)
3. **Phase 3** - Core Combat (dice, attack resolution, defense)
4. **Phase 4** - Damage (damage pipeline, reduction, application)
5. **Phase 5** - Consequences (status effects, wounds, death)
6. **Phase 6** - Entities (Knight, enemy, NPC assembly)
7. **Phase 7** - Mission (encounter flow, objectives, rewards)
8. **Phase 8** - Meta (run progression, meta-game loops)

Work on a higher-phase system MUST NOT begin until all its
lower-phase dependencies are complete and tested.

### VIII. Demo Scope

This is a demo build with explicit scope boundaries:

- **OUT OF SCOPE**: Modules (Knight module system), Point Faible
  (weak points), full campaign progression.
- **SCAFFOLDED**: Items marked "SCAFFOLDED" receive no-op stub
  implementations only (interface + empty body). They MUST NOT
  contain gameplay logic.
- **DEFERRED**: Items marked "DEFERRED" are not implemented at all.
  No files, no stubs, no references in active code paths.

### IX. Commit Strategy

Each completed task produces exactly one commit:

- Commit messages MUST reference the relevant SPEC number
  (e.g., `feat(SPEC-03): implement floor division utility`).
- One commit per task; do not bundle unrelated changes.
- Commits MUST leave the project in a compilable, test-passing state.

## Scope & Exclusions

This constitution governs the "Knight: Into the Darkness" demo build,
a 2D side-scrolling tactical RPG with roguelite elements adapted from
the tabletop RPG "Knight" by Antre Monde Editions.

**In Scope**:
- Core combat loop (attack, defense, damage, consequences)
- Single-knight gameplay with enemy encounters
- Roguelite run structure (run start -> encounters -> run end)
- ScriptableObject data pipeline
- Integration and EditMode test suites

**Excluded**:
- Multiplayer / co-op
- Full module system (Modules are OUT OF SCOPE)
- Point Faible mechanic
- Full campaign / meta-progression beyond demo scope
- Online services, leaderboards, cloud saves

## Development Workflow

All implementation follows a spec-driven workflow:

1. **Read the SPEC**: Before writing any code, read the relevant
   `/TechSpec/specifications/` file in full.
2. **Check Phase Dependencies**: Verify all lower-phase dependencies
   are complete per Principle VII.
3. **Implement to Spec**: Follow the pseudocode, formulas, and
   acceptance criteria exactly.
4. **Test with Worked Examples**: Use the spec's worked examples as
   test vectors.
5. **Commit per Task**: One task, one commit, referencing the SPEC.

Architecture decisions not covered by existing SPECs MUST be raised
for discussion before implementation.

## Governance

This constitution is the supreme authority for project decisions.
All implementation, review, and design work MUST comply with these
principles.

- **Amendments**: Any change to this constitution MUST be documented
  with rationale, approved by the project lead, and accompanied by a
  version increment.
- **Versioning**: Constitution follows semantic versioning:
  - MAJOR: Principle removed or redefined incompatibly.
  - MINOR: New principle or section added, or material expansion.
  - PATCH: Clarifications, typo fixes, non-semantic refinements.
- **Compliance Review**: Each task completion MUST be verified against
  applicable principles before commit. Spec fidelity (Principle II)
  and math rules (Principle IV) are the most common review targets.
- **Conflict Resolution**: When specs, architecture docs, or code
  conflict, resolution priority is:
  Constitution > TechSpec > Tabletop Source > Implementation.

**Version**: 1.0.0 | **Ratified**: 2026-03-10 | **Last Amended**: 2026-03-10