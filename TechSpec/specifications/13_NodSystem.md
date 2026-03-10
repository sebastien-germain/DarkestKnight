# Nod System
> **SPEC:** 33
> **Domain:** `Scripts/Combat`
> **Dependencies:** [01_KnightDataModel.md](./01_KnightDataModel.md) (SPEC-01), [02_PositionAndTurns.md](./02_PositionAndTurns.md) (SPEC-03), [04_ArmorAndState.md](./04_ArmorAndState.md) (SPEC-06, SPEC-07, SPEC-08, SPEC-10)
> **Depended on by:** [04_ArmorAndState.md](./04_ArmorAndState.md) (SPEC-10 Fold exit), [08_ContentData.md](./08_ContentData.md) (SPEC-12 Blason VOEU_OURS), [10_MissionAndHub.md](./10_MissionAndHub.md) (SPEC-13 between-node use, SPEC-25 refill)

---

## SPEC-33: Nod System (Field Recovery Items)

### Purpose
Nods are single-use consumable recovery items carried by every knight. They are the **primary in-field recovery mechanic** — the only way to restore resources during combat and between mission nodes. Nods are critical for sustaining attrition across the 5-node mission structure (SPEC-22), exiting Fold state (SPEC-10), and preventing death from Hémorragie (SPEC-08/23).

Source: Knight Livre de Base, "Nods de soin / d'armure / d'énergie." Simplified for digital: no crafting, no quality tiers, fixed 3D6 restore.

### 33.1 Nod Types

| Nod Type | Restores | Amount | Tarot Modifier |
|----------|----------|--------|----------------|
| Nod de Soin | PS (Points de Santé) | 3D6 (roll 3 dice, SUM all face values) | Guérison Rapide (Tarot XIV): +3 flat bonus → 3D6 + 3 |
| Nod d'Armure | PA (Points d'Armure) | 3D6 (roll 3 dice, SUM all face values) | None |
| Nod d'Énergie | PE (Points d'Énergie) | 3D6 (roll 3 dice, SUM all face values) | None |

**Important distinctions:**
- **Nod de Soin → PS only.** Does NOT restore PEs.
- **Nod d'Armure → PA only.** This is the primary Fold state exit path (SPEC-10).
- **Nod d'Énergie → PE only.** Does NOT restore PEs (Espoir). PEs recovery comes from Motivations (SPEC-12), Exploits (SPEC-04), and narrative events (SPEC-22) — never from Nods.

Restore values are capped at the target's maximum: `target.currentX = min(target.currentX + rollResult, target.maxX)`.

### 33.2 Inventory & Refill

| Field | Value |
|-------|-------|
| Starting quantity | 3 of each type per knight per mission (9 Nods total per knight) |
| Carry limit | 3 of each type (no stacking beyond 3) |
| Refill | Automatic at Camelot between missions (SPEC-25). All Nods reset to 3/3/3. Free — no PG cost. |
| Shared pool | No. Each knight carries their own Nods independently. |

Data Model:
```csharp
struct NodInventory {
  int soin;    // 0–3, starts at 3
  int armure;  // 0–3, starts at 3
  int energie; // 0–3, starts at 3
}
// Stored on KnightBase.nods
```

### 33.3 Usage Rules — In Combat

| Field | Value |
|-------|-------|
| Action cost | 1 Movement Action |
| Roll required | None. Nod use is automatic — no Combo Roll, no failure possible. |
| Target: Self | Always valid. No range restriction. |
| Target: Ally | Courte range (Gap ≤ 2, per SPEC-03 gap formula). Target must be a **living** knight (LifeState == Alive, per SPEC-30). |
| Target: Ally in Agony | **YES — permitted.** Nod de Soin can be used on an ally in Agony (PS = 0) to restore PS ≥ 1, which exits Agony. This is the primary way to save a bleeding knight. **This file (SPEC-33 §33.3) is the canonical source for all Nod targeting rules.** Only Nod de Soin may target an Agony knight (Nod d'Armure and Nod d'Énergie cannot, as the Agony knight has no use for PA/PE recovery while incapacitated). |
| Nod on self while in Agony | **No.** Agony disables all actions (SPEC-08). The knight cannot use Nods on themselves. An ally must administer the Nod. |
| Uses per turn | 1 Nod per Movement Action. A knight can use at most 1 Nod per turn under standard action economy (1 Combat + 1 Movement). If a knight forfeits their Combat Action for a 2nd Movement Action (SPEC-15), they may use 2 Nods that turn. |
| PE cost | None. Nod use does not cost PE. |

### 33.4 Usage Rules — Outside Combat (Between Nodes)

| Field | Value |
|-------|-------|
| Action cost | Free. No action economy outside combat. |
| Target | Self or any ally (no range restriction outside combat). |
| Roll | 3D6 (same as combat). No Combo Roll. |
| Restriction | Cannot use on dead knights. Cannot use during Despair (knight is under hostile AI). |

Between mission nodes (SPEC-13): Nod use is the **ONLY** recovery mechanism. No automatic healing occurs. This is the core attrition management decision — players must choose which Nods to use between encounters and which to save for the boss fight.

### 33.5 Fold State Interactions (SPEC-10)

For complete Fold State rules (systems offline/active, Guardian Suit stats, exit conditions), see [SPEC-10](./04_ArmorAndState.md) (canonical). This section covers only the Nod-specific interactions:

| Nod Type | Usable in Fold? | Notes |
|----------|-----------------|-------|
| Nod de Soin | **Yes** | PS is unchanged in Fold. Normal restore. |
| Nod d'Armure | **Yes** | **Primary Fold exit path.** Restores PA. If PA after restore exceeds Guardian Suit's 5, Fold state ends and all systems come back online (SPEC-10). |
| Nod d'Énergie | **No** | PE is frozen in Fold state. Nod d'Énergie is blocked while Folded. The Nod is not consumed. UI must display "PE frozen — cannot use Nod d'Énergie while in Fold state" (SPEC-31). |

**Fold exit worked example:**
Knight in Fold (PA = 5, Guardian Suit). Ally uses Nod d'Armure on them. Rolls 3D6 = [3, 4, 5] = 12. New PA = 5 + 12 = 17. 17 > 5 (Guardian threshold) → Fold ends. Armor reactivates. CdF returns to base. ODs, modules, abilities, PE spending all restored.

### 33.6 Tarot & Disadvantage Interactions

| Effect | Interaction |
|--------|-------------|
| **Guérison Rapide** (Tarot XIV) | Nod de Soin restores 3D6 + 3 (flat bonus) instead of 3D6. Applies whether the Nod is used on self or on an ally. The bonus belongs to the **user** (the knight with Guérison Rapide), not the target. If a knight without Guérison Rapide uses a Nod on an ally who has it, no bonus. If a knight WITH Guérison Rapide uses a Nod on ANY target, +3 applies. |
| **Prisonnier** (Tarot XX disadvantage) | Blocks ALL PEs gains. Nods do NOT restore PEs (they restore PS/PA/PE), so Prisonnier has **no direct interaction** with Nods. Included for clarity since the disadvantage text mentions "Nods" in its exclusion list — that reference is to hypothetical future Nods or a general catchall; the demo Nod types are unaffected. |
| **Esprit d'Acier** (Tarot IX) | Reduces PEs losses by 1. No Nod interaction (Nods don't affect PEs). |
| **Infatigable** (Tarot I) | No PA passthrough on Nod restores (Nod restores are gains, not damage — passthrough only applies to damage). No interaction. |

### 33.7 Hémorragie Interaction (SPEC-08/23)

When a knight is bleeding (Hémorragie injury, 3-turn countdown to permadeath — see [SPEC-08](./04_ArmorAndState.md) §Hémorragie System for complete specification):
- The bleeding knight is in **Agony** (PS = 0, no actions).
- An ally can use **Nod de Soin** on the bleeding knight (see §33.3 — Agony targeting exception).
- If the Nod restores PS >= 1: Agony ends, and **Hémorragie is removed** from the injury list (it is NOT permanent — exception to the general injury rule). The countdown stops and the knight is fully stabilized.
- If no ally administers a Nod de Soin (or other PS restore) before the countdown reaches 0: instant permadeath.
- For all other Hémorragie rules (countdown timing, Ignorer l'Agonie interaction, between-node carry-over, Despair interaction), see [SPEC-08](./04_ArmorAndState.md) §Hémorragie System (canonical).

### 33.8 Edge Cases

| Scenario | Resolution |
|----------|------------|
| Nod de Soin on knight at full PS | Nod is consumed. Restore capped at maxPS. Effectively wasted. UI should warn "PS already at maximum." |
| Nod d'Armure on knight at full PA | Same — consumed, capped, wasted. UI warning. |
| Nod d'Énergie on knight at full PE | Same — consumed, capped, wasted. UI warning. |
| Knight has 0 Nods of requested type | Action blocked. UI: "No Nods remaining." |
| Nod on dead knight | Blocked. Dead knights cannot be targeted by any action. |
| Nod on Despaired knight (by ally) | **Permitted** for Nod de Soin and Nod d'Armure (Despair doesn't block being targeted). Nod d'Énergie is also permitted — PE restore may enable the Despaired knight to spend PE on abilities once stabilized. Note: Nods do NOT restore PEs, so they cannot directly end Despair (which requires PEs > 0). |
| Nod de Soin on Agony knight — Blason VOEU_OURS | If the Nod restores PS ≥ 1 and the target exits Agony, the VOEU_OURS event fires for the knight who used the Nod (SPEC-02.5 Blason L'Ours trigger). |
| Multiple Nods same target same turn | Allowed if the using knight has multiple Movement Actions (e.g., forfeited Combat Action). Each Nod is a separate 1 Movement Action. |

### 33.9 Data Schema — NodUseAction

```csharp
struct NodUseAction {
  NodType type;          // SOIN, ARMURE, ENERGIE
  string userId;         // Knight using the Nod (owns the inventory)
  string targetId;       // Knight receiving the effect (self or ally)
}

enum NodType { SOIN, ARMURE, ENERGIE }

// Resolution pseudocode:
function resolveNodUse(action, user, target):
  // Validation
  assert user.nods[action.type] > 0          // Has Nods remaining
  assert target.lifeState != Dead             // Not dead
  assert action.type != ENERGIE || target.armorState != Folded  // PE frozen in Fold
  if user != target:
    assert gapBetween(user, target) <= 2      // Courte range (combat only)
    if target.actionState == Incapacitated:   // Agony
      assert action.type == SOIN              // Only Soin can target Agony

  // Consume
  user.nods[action.type] -= 1

  // Roll
  restoreAmount = rollDice(3, D6)  // SUM faces
  if action.type == SOIN && user.hasAdvantage(GUERISON_RAPIDE):
    restoreAmount += 3

  // Apply
  switch action.type:
    SOIN:
      oldPS = target.currentPS
      target.currentPS = min(target.currentPS + restoreAmount, target.maxPS)
      if oldPS == 0 && target.currentPS > 0:
        exitAgony(target)           // SPEC-08
        stopHemorragie(target)      // if active
        fire OnAgonyExited(target)
        fire OnNodHealFromAgony(user, target)  // for VOEU_OURS check
    ARMURE:
      target.currentPA = min(target.currentPA + restoreAmount, target.maxPA)
      if target.armorState == Folded && target.currentPA > 5:
        exitFold(target)            // SPEC-10
        fire OnFoldEnded(target)
    ENERGIE:
      target.currentPE = min(target.currentPE + restoreAmount, target.maxPE)

  fire OnNodUsed(user, target, action.type, restoreAmount)
```

### Acceptance Criteria

**SPEC-33-AC1:** Knight uses Nod de Soin on self. PS 20/40, rolls 3D6 = [3, 4, 6] = 13. New PS = 33/40. Nod inventory: Soin 3 → 2.

**SPEC-33-AC2:** Knight with Guérison Rapide uses Nod de Soin on ally. Ally PS 5/40, rolls 3D6 = [2, 3, 5] = 10. Bonus +3 = 13. New PS = 18/40.

**SPEC-33-AC3:** Knight uses Nod d'Armure on Folded ally (PA = 5). Rolls 3D6 = [4, 5, 6] = 15. New PA = 20. 20 > 5 → Fold ends. All armor systems online.

**SPEC-33-AC4:** Knight attempts Nod d'Énergie on Folded ally. Action blocked: "PE frozen in Fold state."

**SPEC-33-AC5:** Ally in Agony (PS = 0, Hémorragie active, 2 turns remaining). Adjacent knight (Gap <= 2) uses Nod de Soin. Rolls 3D6 = [1, 2, 4] = 7. Target PS = 7. Agony ends. Hémorragie **removed** from injuries (not permanent). Knight survives and acts normally next turn.

**SPEC-33-AC6:** Knight tries to use Nod de Soin on self while in Agony. Action blocked: "No actions available in Agony."

**SPEC-33-AC7:** Knight at Rank 4 tries to use Nod on ally at Rank 1 (Gap = 3+0 = 3). Blocked: "Out of range (Gap 3 > Courte max 2)."

**SPEC-33-AC8:** Between nodes (outside combat), knight uses Nod de Soin. No Movement Action consumed. PS restored by 3D6. Nod consumed.

**SPEC-33-AC9:** Knight uses Nod d'Armure on ally. Ally PA 95/100, rolls 3D6 = [6, 6, 5] = 17. Capped at 100. Effective restore = 5. Nod consumed (not refunded).

**SPEC-33-AC10:** Knight uses Nod de Soin on Agony ally → VOEU_OURS fires for the healing knight if they have L'Ours Blason.

