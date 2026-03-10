# Knight Data Model & Generation Pipeline
> **SPECs:** 01, 02, 02.5, 02.7, 02.8, 02.9
> **Domain:** `Scripts/Data`
> **Dependencies:** [09_TarotSystem.md](./09_TarotSystem.md) (SPEC-24), [05_ArmorAbilities.md](./05_ArmorAbilities.md) (SPEC-17)
> **Depended on by:** All combat systems, [08_ContentData.md](./08_ContentData.md) (SPEC-12)
---
## SPEC-01: Knight Data Model (KnightBase)
### Purpose
Runtime data structure for a knight. All other systems read/write this model.
### Data Schema

```csharp
class KnightBase : ScriptableObject {
  string knightId; string displayName; string callsign;
  ArchetypeData archetype; TarotCard[] drawnCards;
  TarotCard[] activeAdvantages; // 2 selected
  TarotCard activeDisadvantage; // 1 selected
  HautFaitData hautFait; BlasonData blason;
  Motivation[] minorMotivations; // Dynamic list, 3 base entries (2 drawn + 1 Blason Voeu). Max 4 if Tarot grants a bonus motivation (Le Pendu Code Moral OR Le Monde Créateur-né — never both on the same knight, as drawing both is resolved by the 5-card selection logic: only one can be an active advantage). See SPEC-24.
  Motivation majorMotivation; // nullable

  // Aspects (mutable) int chair,bete,machine,dame,masque; // 0–9
  // 15 Characteristics indexed by CharacteristicId enum
  // Access via CharacteristicId enum index (e.g., characteristics[(int)CharacteristicId.COMBAT]).
  // Named aliases (e.g., knight.Combat) are optional convenience getters — not required.
  // Aspects above (chair, bete, etc.) are separate named fields because there are only 5.
  int[15] characteristics;

  // Meta-Armor (immutable after assignment)
  ArmorClass armorClass; // Warrior/Paladin/Priest/Rogue/...
  int maxPA, maxPE, baseCdF;
  int[15] odLevels; // [v8.9] OD per characteristic. Hard cap: 5 from all normal sources (generation, modules, future). Warrior Type +1 is the ONLY exception: can push to 6 (SPEC-17.1). Store base OD here; Type bonus is runtime-only (not persisted).
  SlotCounts slotCounts;

  // Runtime Resources (mutable)
  int currentPS, maxPS; int currentPA; int currentPE;
  int currentPEs, maxPEs; int heroisme; // 0–6
  int currentCdF;

  // Combat State
  int position; // 1–4
  CombatStyle activeStyle;
  bool isInAgony, isInFoldState, isDespair;
  bool isGhostActive; // [v2] Rogue Ghost state
  bool hasHémorragie; // [v8.9] true while bleeding. See SPEC-08 §Hémorragie System.
  int hémorragieCountdown; // [v8.9] 3→2→1→0(dead). Ticks at start of knight's turn.
  int activeWarriorTypeIndex; // [v2] nullable, Warrior only
  List<InjuryResult> activeInjuries; // [v8.9] InjuryResult { severityDie, rowDie, injuryId, isPermanent }. Hémorragie: isPermanent=false (removed on Agony exit). See SPEC-08.
  int implantCount; // 0–6, each reduces maxPEs by 3. Reset to 0 by Reconstruction Therapy (SPEC-25).
  List<StatusEffect> activeStatusEffects;

  // Equipment
  Weapon[] equippedWeapons; int activeWeaponIndex;
  int activeProfileIndex;
  Module[] equippedModules; // [v8] Modules OUT OF DEMO SCOPE
  NodInventory nods; // See SPEC-33 (13_NodSystem.md)
  bool isDead;
}
```

### Derived Value Formulas

| Derived Value | Formula | Recalculate When |
| --- | --- | --- |
| Defense | Max(hargne,combat,instinct) + that char’s OD | BÊTE char/OD change or Type change |
| Reaction | Max(tir,savoir,technique) + that char’s OD | MACHINE char/OD change or Type change |
| Initiative | Max(discretion,dexterite,perception) + that char’s OD | MASQUE char/OD change or Type change |
| maxPS | 10 + 6 × Max(force,endurance,deplacement) [OD excluded] | CHAIR char changes |
| maxPEs | [v8.9] 50 + Tarot mods (e.g. Forteresse Spirituelle +5) + Major Motivation bonuses − (implantCount × 3). Reconstruction Therapy resets implantCount to 0, restoring maxPEs. | Creation; implant/therapy at Camelot; Major Motivation runtime |
| Points de Contact | Max(aura,parole,sangFroid) [OD excluded]. **SCAFFOLDED — no demo consumer. Do NOT compute or store during generation.** The formula is preserved here for future reference only. No system reads this value in demo. Future full-game design: Points de Contact will grant pre-mission bonuses at Camelot (e.g. experimental weapon = bonus damage for 1 mission, reinforced force field, tactical reinforcements). Requires dedicated design pass for full game. | N/A (not computed in demo) |


[v7] RESOLVED: [v7] Defense = highest Bête Characteristic score + that Characteristic's OD level. Reaction = highest Machine Characteristic score + that Characteristic's OD level. Initiative = highest Masque Characteristic score + that Characteristic's OD level. Pattern: pick the single highest-scoring Characteristic within the relevant Aspect, then add only that Characteristic's OD. Do NOT sum all Aspect ODs. Pseudocode: int Defense => highestCharIn(BETE) + odOf(highestCharIn(BETE)); int Reaction => highestCharIn(MACHINE) + odOf(highestCharIn(MACHINE)); int Initiative => highestCharIn(MASQUE) + odOf(highestCharIn(MASQUE)); maxPS = 10 + 6 × Max(Force, Endurance, Déplacement) [OD excluded]. Recalculate whenever any Chair Characteristic changes (injury, buff, debuff). Pseudocode: int maxPS => 10 + 6 * highestCharIn(CHAIR); maxPA = armor base PA (SPEC-02.7). maxPE = armor base PE (SPEC-02.7). baseCdF = armor base CdF (SPEC-02.7). maxPEs = 50 (base, modified by Tarot advantages: Forteresse Spirituelle +5, implants −3 each). [v8.9] Full formula: maxPEs = 50 + tarotMaxPEsBonus − (implantCount × 3). Reconstruction Therapy (SPEC-25): resets implantCount to 0, restoring maxPEs to 50 + tarotMaxPEsBonus + majorMotivationBonuses.  
### Acceptance Criteria
KnightBase with all chars at 1, aspects at 2: correct derived values.   
Injury reducing a Chair Characteristic recalculates maxPS immediately. Injury reducing a Bête Characteristic recalculates Defense immediately. Injury reducing a Machine Characteristic recalculates Reaction immediately. Injury reducing a Masque Characteristic recalculates Initiative immediately.   
Characteristics floor at 0.   
Major Motivation: maxPEs +25, currentPEs +25 independently.   
SPEC-01-AC1: Knight with Bête chars Combat 5 (OD 2), Instinct 3 (OD 0), Hargne 4 (OD 1): Defense = 5 + 2 = 7 (Combat is highest).   
SPEC-01-AC2: Same knight, Machine chars Tir 4 (OD 1), Savoir 2 (OD 0), Technique 3 (OD 0): Reaction = 4 + 1 = 5 (Tir is highest).   
SPEC-01-AC3: Warrior Type Hunter active (+1 OD to Combat, Instinct, Hargne): Combat OD becomes 3, Defense recalculates to 5 + 3 = 8.  
[v8.9] SPEC-01-AC3b: Warrior with Combat baseOD 5, activates Hunter Type: effective Combat OD = 6 (exceeds normal cap of 5 — Type is the only exception). Defense = 5 + 6 = 11.  
[v8.9] SPEC-01-AC3c: Non-Warrior with Combat baseOD 5: OD stays at 5. No Type bonus available. Hard cap enforced.   
SPEC-01-AC4: Characteristic at 6, parent Aspect at 5: validation error. Characteristic capped at Aspect score.   
Aspect/Characteristic Reference: Chair (Body): Déplacement, Force, Endurance. Bête (Beast): Combat, Instinct, Hargne. Machine: Tir, Savoir, Technique. Dame (Lady): Aura, Parole, Sang-Froid. Masque (Mask): Discrétion, Dextérité, Perception. Constraint: Each Characteristic score ≤ its parent Aspect score. Base: Aspects = 2, Characteristics = 1 (before generation bonuses).   
**Aspect cap rules:** Aspect scores during generation are uncapped within the normal maximum of **9**. With 5 Tarot cards (+1 Aspect each) and 1 Haut Fait (+1 Aspect), a single Aspect can reach up to 2 (base) + bonuses = practical max around 7–8. Hard cap: **9** (normal), **7** (Vétéran disadvantage, Tarot XIII). Aspects are NOT reduced by injuries — only Characteristics are. The Characteristic ≤ Aspect constraint applies during generation and leveling, but NOT retroactively: an injury can reduce a Characteristic below its Aspect score without issue. Post-generation validation: assert all Aspects ≤ 9 (or ≤ 7 if Vétéran), and all Characteristics ≤ parent Aspect.   

## SPEC-02: Knight Generation Pipeline
### Purpose
Procedurally generates 8 demo knights at campaign start. [v4] Player selects 4 for each mission.
### Slot Allocation
[v3] RESOLVED (C5): 2 of each demo class. 8 knights total. [v4] 4 deployed per mission (DD rank system).

| Slot | Armor | Primary Aspects |
| --- | --- | --- |
| 1–2 | Warrior | BÊTE + CHAIR |
| 3–4 | Paladin | CHAIR + MACHINE |
| 5–6 | Priest | MACHINE + DAME |
| 7–8 | Rogue | MASQUE + BÊTE |


### [v4] Mission Squad Selection
[v8.9] **Canonical definition:** See [SPEC-03](./02_PositionAndTurns.md) §Squad Selection (canonical). Do not duplicate rules here — refer to SPEC-03 for deployment, composition guidance, and roster view requirements.

### Algorithm (10 Steps)
* Base: All Aspects = 2, all Characteristics = 1. 
* Archetype: Random from 17. Apply +1 to two Characteristics. See SPEC-02.9 Archetype Table below.

#### SPEC-02.9 — Archetype Table (17 entries)
Each archetype represents a personality/background type from the knight's life before joining the order. Mechanically, it provides +1 to exactly two Characteristics (which may be from different Aspects). Source: Knight Livre de Base, Chapter "Étape 2 : Un Archétype."

Data Model:
```csharp
struct ArchetypeData {
  string archetypeId; // e.g., "ARCHETYPE_REBUT"
  string displayName; // French display name
  CharacteristicId bonus1; // First +1 characteristic (fixed, OR null if choiceBonusSlot == 1)
  CharacteristicId bonus2; // Second +1 characteristic (fixed, OR null if choiceBonusSlot == 2)
  CharacteristicId[] choicePool; // nullable. If non-null, the bonus at choiceBonusSlot is randomly picked from this pool.
  int choiceBonusSlot; // 0 = no choice (both fixed). 1 = bonus1 is from choicePool. 2 = bonus2 is from choicePool.
}
```

Generation: Random uniform draw from 17 archetypes. No weighting by armor class. Apply +1 to each of the two listed Characteristics. **Choice archetypes** (#6, #7, #8): the tabletop lets the player choose "Combat OR Tir." For automated generation, resolve randomly (50/50). Characteristics remain capped at parent Aspect score — if a bonus would exceed the cap, it is wasted (no redistribution).

| # | Archetype | Bonus 1 | Bonus 2 | choicePool | Notes |
|---|-----------|---------|---------|------------|-------|
| 1 | Rebut | Déplacement (Chair) | Endurance (Chair) | — | Survivalist / rebel / outcast |
| 2 | Citoyen | Technique (Machine) | Discrétion (Masque) | — | Obedient citizen of the arches |
| 3 | Habitant des Territoires Libres | Savoir (Machine) | Parole (Dame) | — | Free territories resident |
| 4 | Célébrité | Aura (Dame) | Parole (Dame) | — | Famous personality |
| 5 | Survivant | Endurance (Chair) | Hargne (Bête) | — | Survived catastrophe / abduction |
| 6 | Membre d'une Société Secrète | Savoir (Machine) | *(from pool)* | `[Combat, Tir]` | Secret society member. Auto-gen: 50/50 random. |
| 7 | Agent du Nodachi | *(from pool)* | Dextérité (Masque) | `[Combat, Tir]` | Former Nodachi operative. **bonus1** is the choice here. |
| 8 | Membre d'un Service Secret | *(from pool)* | Discrétion (Masque) | `[Combat, Tir]` | Spy / intelligence agent. **bonus1** is the choice here. |
| 9 | Génie | Savoir (Machine) | Technique (Machine) | — | Scientist / inventor |
| 10 | Artiste | Hargne (Bête) | Aura (Dame) | — | Passionate creator |
| 11 | Indépendant | Perception (Masque) | Instinct (Bête) | — | Freelance journalist / detective |
| 12 | Religieux | Sang-Froid (Dame) | Hargne (Bête) | — | Priest / cult member / shaman |
| 13 | Leader | Aura (Dame) | Instinct (Bête) | — | Political / military leader |
| 14 | Hors-la-loi | Sang-Froid (Dame) | Discrétion (Masque) | — | Criminal seeking redemption |
| 15 | Voyageur | Déplacement (Chair) | Perception (Masque) | — | Explorer / war reporter |
| 16 | Combattant | Combat (Bête) | Tir (Machine) | — | Soldier / mercenary / SWAT |
| 17 | Force de la Nature | Force (Chair) | Endurance (Chair) | — | Athlete / physical laborer |

**choicePool column rules:** `—` = both bonuses are fixed. `[Combat, Tir]` = the indicated bonus slot (bonus1 or bonus2, as marked in Notes) is randomly selected from the pool at generation (50/50). The `ArchetypeData.choicePool` field maps directly to this column — null for `—`, array of CharacteristicIds for choice entries.

**Note:** The tabletop also defines an "Archétype Libre" (custom archetype with MJ-approved bonuses). This is excluded from automated generation — demo uses the 17 canonical entries only.

**Characteristic Coverage (each char appears N times across all 17 archetypes — counting choice archetypes as 0.5 each):**

| Characteristic | Count | Characteristic | Count |
|---|---|---|---|
| Déplacement | 2 | Aura | 3 |
| Force | 1 | Parole | 2 |
| Endurance | 3 | Sang-Froid | 2 |
| Combat | 2.5 | Discrétion | 3 |
| Instinct | 2 | Dextérité | 1 |
| Hargne | 3 | Perception | 2 |
| Tir | 2.5 | Technique | 2 |
| Savoir | 3 | | | 
* Tarot: [v6] Draw 5 from full 22-card Major Arcana using weighted random without replacement. [v8.9] **Weighting rule:** Cards whose `linkedAspect` matches one of the armor class's 2 primary aspects get weight 3. All other cards get weight 1. Cards with no linked Aspect (Le Fou #0, La Maison-Dieu #XVI) get weight 1.
```python
function drawTarotCards(armorClass, rng, count=5):
  primaryAspects = SLOT_TABLE[armorClass]  # e.g., Warrior = [BETE, CHAIR]
  pool = ALL_22_MAJOR_ARCANA.copy()
  drawn = []
  for i in range(count):
    weights = []
    for card in pool:
      if card.linkedAspect is None:
        weights.append(1)
      elif card.linkedAspect in primaryAspects:
        weights.append(3)
      else:
        weights.append(1)
    selected = rng.weightedSelect(pool, weights)
    drawn.append(selected)
    pool.remove(selected)  # no replacement
  return drawn
```
Per card: +1 Aspect, distribute 3 Characteristic pts among the card's linked Aspect characteristics ([v8] using the Armor Class Weighted Distribution Table below — each point rolled independently against class weights, capped at parent Aspect score). From 5 drawn: select 2 advantages, 1 disadvantage. All 5 cards apply stat bonuses. [v8] Incompatibility check: if both L'Arcane sans Nom (XIII) and La Maison-Dieu (XVI) are drawn, redraw the second one. **For the complete Tarot draw procedure, selection rules, and Amnésique post-processing, see [SPEC-24.5](./09_TarotSystem.md) (canonical).**
* Haut Fait:
Random from 14 (see Haut Fait Table below). Bonus: +1 to linked Aspect, +2 Characteristic pts distributed among that Aspect's 3 characteristics (same Armor Class Weighted Distribution as Tarot, 2 independent rolls). Condition check: verify knight meets prerequisite after Tarot bonuses; if not, re-draw (max 10 retries, then waive condition). For dual-Aspect entries (#4, #11, #12): resolve using Armor Class Primary Aspect weighting (see Dual-Aspect Resolution below). Callsign: random from entry's surnoms list.

#### SPEC-02.8 — Haut Fait Table (14 entries)

| # | Haut Fait | Bonus Aspect | Condition Type | Condition | Surnoms |
|---|-----------|-------------|----------------|-----------|---------|
| 1 | Combat de Titans | Chair | ASPECT | Chair Aspect ≥ 4 | Le Poing, le Forcené, le Colosse |
| 2 | En Territoire Ennemi | Bête | ASPECT | Bête Aspect ≥ 4 | Le Discret, l'Ombre, le Fantôme |
| 3 | Le Renouveau de l'Espoir | Bête | EITHER_CHAR | Hargne ≥ 4 OR Sang-Froid ≥ 4 | L'Espéré, le Lumineux, l'Innocent |
| 4 | Héros de Guerre | **Bête OR Machine** | EITHER_CHAR | Combat ≥ 4 OR Tir ≥ 4 | Le Champion, le Hérault, le Guerrier |
| 5 | Construction de la Première Arche | Machine | EITHER_CHAR | Technique ≥ 4 OR Savoir ≥ 4 | L'Architecte, le Créateur, le Constructeur |
| 6 | Conception de la Première Méta-Armure | Machine | EITHER_CHAR | Technique ≥ 4 OR Savoir ≥ 4 | L'Armurier, le Forgeron, le Fabricant |
| 7 | Découverte d'une Tache Dissimulée | Masque | EITHER_CHAR | Perception ≥ 4 OR Instinct ≥ 4 | Le Limier, le Traqueur, l'Observateur |
| 8 | Tueur de Ténèbres | Masque | EITHER_CHAR | Discrétion ≥ 4 OR Perception ≥ 4 | Le Nettoyeur, l'Assassin, le Bourreau |
| 9 | Guide | Dame | EITHER_CHAR | Parole ≥ 4 OR Perception ≥ 4 | Le Guide, le Voyageur, le Capitaine |
| 10 | Création du Knight | Dame | EITHER_CHAR | Aura ≥ 4 OR Combat ≥ 4 | Le Fondateur, le Sénéchal, l'Allié |
| 11 | Défenseur de l'Art | **Bête OR Dame** | EITHER_CHAR | Hargne ≥ 4 OR Sang-Froid ≥ 4 | Le Mécène, le Gardien, le Conservateur |
| 12 | Sauveur | **Masque OR Dame** | EITHER_CHAR | Perception ≥ 4 OR Sang-Froid ≥ 4 | Le Protecteur, le Sauveur, le Garde du corps |
| 13 | Survivant de la Peste Rouge | Chair | EITHER_CHAR | Endurance ≥ 4 OR Hargne ≥ 4 | Le Survivant, l'Abîmé, le Dévasté |
| 14 | Torturé dans les Ténèbres | Chair | EITHER_CHAR | Endurance ≥ 4 OR Force ≥ 4 | Le Torturé, le Ténébreux, le Sombre |

**Condition Type rules:**
- `ASPECT`: Check the Aspect score itself (e.g., #1: knight.chair ≥ 4). `condChar1`/`condChar2` unused — the checked Aspect is `bonusAspect`.
- `EITHER_CHAR`: Check either `condChar1 ≥ condMinScore` OR `condChar2 ≥ condMinScore`. Characteristics may belong to different Aspects than the bonus Aspect.

#### Dual-Aspect Resolution (#4, #11, #12)
Entries with two possible bonus Aspects are resolved per armor class using primary Aspect weighting:

| # | Haut Fait | Aspect A / Aspect B | Warrior | Paladin | Priest | Rogue |
|---|-----------|---------------------|---------|---------|--------|-------|
| 4 | Héros de Guerre | Bête / Machine | 75/25 | 25/75 | 25/75 | 75/25 |
| 11 | Défenseur de l'Art | Bête / Dame | 75/25 | 50/50 | 25/75 | 75/25 |
| 12 | Sauveur | Masque / Dame | 50/50 | 50/50 | 25/75 | 75/25 |

Pseudocode:
```python
function resolveHautFaitDualAspect(armorClass, A, B, rng):
primaryAspects = SLOT_TABLE[armorClass] # e.g., Warrior = [BETE, CHAIR]
wA = 3 if A in primaryAspects else 1
wB = 3 if B in primaryAspects else 1
return A if rng.Chance01(wA / (wA + wB)) else B
```

SPEC-02.8 Data Model (standalone for agent extraction): struct HautFaitData { string hautFaitId; string displayName; string callsign; AspectId bonusAspect; int bonusCharPoints = 2; ConditionType conditionType; // ASPECT = check Aspect score, EITHER_CHAR = check either condChar1 or condChar2. CharacteristicId condChar1; CharacteristicId condChar2; // nullable (single-condition entries). int condMinScore = 4; } enum ConditionType { ASPECT, EITHER_CHAR }
* Blason: Random from 9 demo Blasons (see SPEC-02.5 Blason Table below). Voeu = 3rd minor Motivation. Voeu event tag must not duplicate the knight's 2 drawn MN templates (if collision, re-draw Blason). Le Cheval and Le Corbeau are deferred to full game. 
* Motivations: 1 major + 2 minor randomly drawn from template pools (see SPEC-12.1 Motivation Templates). 3rd minor = Blason Voeu. No duplicates within a single knight. Algorithm: draw 2 unique minor templates from pool of 10, draw 1 major template from pool of 5. Each template carries an eventTag string used by SPEC-12 event matching. 
* Meta-Armor: Apply armor base stats from table below. Also assign starting Types/abilities per SPEC-17. 
SPEC-02.7 — Meta-Armor Base Stats Table (Source: Knight Livre de Base, knight-jdr-systeme.fr) 

| Armor | PA | PE | CdF | Gen. | Slots (6 zones) | Base Overdrives |
| --- | --- | --- | --- | --- | --- | --- |
| Warrior | 100 | 40 | 8 | 1st | 7/10/10/12/7/7 | Déplacement, Combat, Tir, Dextérité |
| Paladin | 120 | 20 | 8 | 1st | 7/7/7/10/7/7 | Force, Endurance, Tir, Perception |
| Priest | 70 | 60 | 10 | 1st | 5/5/5/8/5/5 | Force, Endurance, Savoir, Technique |
| Rogue | 50 | 70 | 12 | 2nd | 5/5/5/8/5/5 | Déplacement, Combat, Discrétion, Dextérité |

Slots = Head/Torso/ArmL/ArmR/LegL/LegR. Guardian Suit (Fold state): PA 5, CdF 5, no PE, no OD, no modules. Set knight.maxPA = armor PA, knight.maxPE = armor PE, knight.baseCdF = armor CdF. Initialize currentPA = maxPA, currentPE = maxPE, currentCdF = baseCdF. 
* Sections: Skip (null). 
* Derived Values: Compute per SPEC-01. 
* Equipment: Standard kit + default weapons/modules. 

### [v8] Armor Class Weighted Distribution Table (M-1)
When distributing Tarot/Haut Fait Characteristic points during procedural generation, each point is assigned independently using a weighted random roll. Only the 3 characteristics linked to the drawn card’s Aspect participate. Their weights are normalized into probabilities, then one is selected per point. Weight tiers: Specialized (★7), Primary (5), Secondary (3), Tertiary (1). All classes sum to 43 for balance. Characteristic scores are capped at their parent Aspect score — if a characteristic is already at cap, it is excluded from the eligible pool and its weight is redistributed among remaining candidates. If all 3 are capped, remaining points are discarded.

| Characteristic | Aspect | Warrior | Rogue | Paladin | Priest | Ranger ⚠️FUTURE | Tier Key |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Déplacement | Chair | 3 | 5 | 1 | 1 | 1 | |
| Force | Chair | 5 | 1 | 3 | 1 | 1 | |
| Endurance | Chair | ★7 | 1 | 5 | 1 | 1 | |
| Combat | Bête | 5 | 5 | 3 | 1 | 1 | |
| Instinct | Bête | 3 | 3 | 1 | 1 | 5 | |
| Hargne | Bête | 5 | 1 | 1 | 1 | 1 | |
| Tir | Machine | 3 | 5 | 5 | 5 | ★7 | |
| Savoir | Machine | 1 | 1 | 1 | 5 | 5 | |
| Technique | Machine | 1 | 1 | 3 | ★7 | 3 | |
| Aura | Dame | 1 | 1 | 5 | 5 | 1 | |
| Parole | Dame | 1 | 1 | 3 | 3 | 1 | |
| Sang-Froid | Dame | 5 | 3 | 5 | 5 | 5 | |
| Discrétion | Masque | 1 | ★7 | 1 | 1 | 3 | |
| Dextérité | Masque | 1 | 5 | 1 | 3 | 3 | |
| Perception | Masque | 1 | 3 | 5 | 3 | 5 | |
| Total | | 43 | 43 | 43 | 43 | 43 | |


Tier key: ★7 = Specialized, bold = Primary (5), plain = Secondary (3), 1 = Tertiary. Paladin has no Specialized tier but compensates with 5 Primaries and 4 Secondaries. Priest specializes in Technique (★7) for NanoC construction, with strong Dame support (Aura/Sang-Froid at 5). Ranger column is intentionally preserved as the NEXT PLANNED EXPANSION CLASS (post-demo). Stats are canonical (PA 50, PE 70, CdF 12, Source: Livre de Base). Generation weights are pre-defined to accelerate implementation when the class is added. Ranger is excluded from demo generation pool — do not select during SPEC-02 step 1.

[v8] Distribution Algorithm (pseudocode):
```python
function distributeCardPoints(armorClass, card, numPoints, knight):
aspect = card.linkedAspect
linkedChars = getCharacteristicsForAspect(aspect) // always 3
weights = [WEIGHT_TABLE[armorClass][c] for c in linkedChars]
for i in range(numPoints):
  // Filter out chars already at Aspect cap
  eligible = [(c, w) for c, w in zip(linkedChars, weights)
              if knight.chars[c] < knight.aspects[aspect]]
  if len(eligible) == 0: break // all capped, discard point
  totalW = sum(w for _, w in eligible)
  roll = random(0, totalW)
  selected = weightedSelect(eligible, roll)
  knight.chars[selected] += 1
```

[v8] Worked Example:
Priest draws La Justice (Machine card, 3 pts). Linked chars: Tir(5), Savoir(5), Technique(★7). 
Normalized: 29%/29%/41%. Priest currently has Machine Aspect = 3, all chars at 1. 
Point 1: roll → Technique (41%). Technique: 1→2. 
Point 2: roll → Technique again. Technique: 2→3. Now Technique = Aspect cap (3). 
Point 3: eligible = Tir(5), Savoir(5). Normalized: 50%/50%. Roll → Savoir. Savoir: 1→2. 
Final: Tir 1, Savoir 2, Technique 3. Support specialist with growing knowledge base. 

### Acceptance Criteria
8 distinct knights. No Characteristic exceeds parent Aspect. 
Each has 2 advantages, 1 disadvantage, 3 minor + 1 major Motivation. 
Same seed = identical results. 

### SPEC-02.5 — Blason System (Source: Knight Livre de Base, knight-jdr-systeme.fr/fr/crest/)
Each knight chooses a Blason at creation — an animal emblem displayed on the meta-armor (shoulder or torso). The Blason defines the knight’s moral identity and provides a Vœu (vow): an additional minor Motivation (the 3rd) with its own event tag. In the tabletop, Blasons are purely roleplay. For the video game, each Vœu is mapped to a concrete event trigger following the same SPEC-12 event bus system as MN templates. Vœu motivations are minor (+1D6 PEs per trigger, repeatable). Le Cheval and Le Corbeau are deferred to full game (relationship system and investigation system respectively). 
Generation: Random from 9 demo Blasons. Uniqueness rule: the Vœu event tag must NOT duplicate either of the knight’s 2 drawn MN minor motivation tags. If collision, re-draw Blason (max 10 retries, then pick first non-colliding). 
Demo Blason Table (10 entries, 9 active — Le Dragon #4 is deferred): 
[1] L’Aigle — «Les Justes» — Tag: VOEU_AIGLE — Vœu: Faire respecter le code d’honneur du Knight lorsqu’il semble bafoue. — Trigger: knight stabilizes a Despaired ally (SPEC-07 Stabilization). The Aigle knight enforces honor when a comrade falters. 
[2] L’Ours — «Les Protecteurs» — Tag: VOEU_OURS — Vœu: Empêcher la mort d’un être humain. — Trigger: knight heals an ally who is in Agony (PS = 0) back to PS ≥ 1 via Nod Soin or Priest Mechanic. The Ours knight prevents death directly. 
[3] Le Cerf — «Les Déférents» — Tag: VOEU_CERF — Vœu: Obéir à la lettre aux ordres des chevaliers et d’Arthur. — Trigger: knight does not change rank position during an entire combat node (Track: set neverMovedFlag = true at deployment. Set to false if knight ever occupies a rank ≠ deploymentRank during the node. Checked at node completion: triggers only if neverMovedFlag is still true). Represents disciplined obedience — staying at your assigned post. Triggers once per qualifying combat node. 
[4] Le Dragon — «Les Droits» — DEFERRED TO FULL GAME. Removed from demo Blason rolling table. The Dragon Vœu ("no Failure Critique in combat") requires clearer scoping of which roll types count. Will be redesigned when the relationship system and broader roll tracking are implemented. 
[5] Le Faucon — «Les Guerriers» — Tag: VOEU_FAUCON — Vœu: Pourfendre, sans aide, un salopard ou un patron de l’Anathème. — Trigger: knight deals the killing blow to a T3+ enemy (Salopard, Colosse, or Patron) AND no Assist was used on the killing attack roll. Solo elite kill. Triggers once per qualifying kill. 
[6] Le Lion — «Les Courageux» — Tag: VOEU_LION — Vœu: Ne jamais fuir devant un ennemi s’il met des innocents en danger. — Trigger: knight deployed at Rank 1 (front) survives an entire combat node without ever moving to a higher rank (Rank 2, 3, or 4). Holding the line. Track: flag set to false if knight ever occupies Rank 2+ during combat. Checked at node completion. Triggers once per qualifying node. 
[7] Le Loup — «Les Solidaires» — Tag: VOEU_LOUP — Vœu: Protéger un chevalier alors qu’il est en danger. — Trigger: knight uses Nod d’Armure or Priest Mechanic on an ally who is at ≤25% maxPA at the time of healing. Pack loyalty — supporting the wounded. Triggers once per qualifying heal action. 
[8] Le Sanglier — «Les Persévérants» — Tag: VOEU_SANGLIER — Vœu: Accomplir les objectifs coûte que coûte. — Trigger: knight is alive at mission completion AND has at least one of: ≥1 active injury, entered Fold state during mission, or current PEs ≤ 10. Perseverance through adversity — only triggers if the knight was actually tested. Checked at OnMissionComplete. If the knight is unscathed (no injury, never folded, PEs > 10), the Vœu does NOT trigger — there was no adversity to overcome. 
[9] Le Serpent — «Les Savants» — Tag: VOEU_SERPENT — Vœu: Découvrir les secrets de l’Anathème. — Trigger: knight is the first party member to attack a previously un-attacked enemy type during the mission. Track per-mission per enemy type (e.g., first knight to hit a Bestian, first to hit a Faune, etc.). Scouting the unknown. Triggers once per new enemy type encountered. In the demo mission with 4 enemy types, up to 4 possible triggers per mission. 
[10] Le Taureau — «Les Loyaux» — Tag: VOEU_TAUREAU — Vœu: Respecter, à chaque mission, le code d’honneur du Knight. — Trigger: knight completes a mission without entering Despair AND without any allied knight dying during the mission. Full squad discipline upheld. Strictest Vœu — requires zero deaths and personal morale control. Checked at OnMissionComplete. Triggers once per qualifying mission. 
Deferred Blasons (full game only): Le Cheval (loyalty/relationship system required), Le Corbeau (investigation/discovery system required), Le Dragon (roll scope clarification required — see v8.4 L-1). These are canonical tabletop Blasons and will be implemented when their dependent systems are designed. 
Data Model: struct BlasonData { string blasonId; string displayName; string animalIcon; string voeuText; string voeuEventTag; } — Stored on KnightBase.blason. The Vœu motivation is stored as the 3rd entry in KnightBase.minorMotivations[] with type = VOEU and the Blason’s event tag.


