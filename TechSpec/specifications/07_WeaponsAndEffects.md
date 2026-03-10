# Weapons & Effects Glossary
> **SPECs:** 19, 20
> **Domain:** `Scripts/Data`
> **Dependencies:** None (reference tables)
> **Depended on by:** [03_CombatRolls.md](./03_CombatRolls.md) (SPEC-05), [04_ArmorAndState.md](./04_ArmorAndState.md) (SPEC-06), [06_EnemySystem.md](./06_EnemySystem.md) (SPEC-18), [08_ContentData.md](./08_ContentData.md) (SPEC-21)
---
## [v5] SPEC-19: Weapon Data Model & Demo Arsenal
### Purpose
Defines the WeaponProfile schema and provides complete stat blocks for all weapons available in the demo. Links to SPEC-20 for effect definitions.

### 19.1 WeaponProfile Schema
```csharp
class WeaponProfile : ScriptableObject {
  string weaponId; string displayName;
  WeaponCategory category; // STANDARD, ADVANCED, RARE
  WeaponSlot slot; // CONTACT, RANGED, DUAL_PROFILE
  int pgCost; // Glory point cost (for future purchase system)

  // A weapon may have multiple profiles (e.g. Marteau-Épieu: contact + tir)
  DamageProfile[] profiles;
  ProfileSwitchCost profileSwitchCost; // [v8.9] FREE or MOVEMENT_ACTION. See SPEC-04 §Profile Switch Cost Rules.
}

struct DamageProfile {
  string profileName; // e.g. 'Contact', 'Tir', 'Grenade Explosive'
  ProfileType type; // CONTACT, RANGED
  int damageDice; // Number of D6 for damage
  int damageFlat; // Flat bonus added to damage roll
  bool addForce; // If true, add Force score (contact weapons)
  ForceMode forceMode; // NORMAL (×1), LESTE (×2)
  int violenceDice; // Number of D6 for violence
  int violenceFlat; // Flat bonus to violence
  WeaponRange range; // CONTACT, COURTE, MOYENNE, LONGUE, LOINTAINE
  int energyCost; // PE per use (0 = free)
  WeaponEffectId[] effects; // References to SPEC-20 effect definitions
}

enum WeaponRange { CONTACT, COURTE, MOYENNE, LONGUE, LOINTAINE }
enum ForceMode { NONE, NORMAL, LESTE }
```

### 19.2 Range Rules (Tabletop Faithful)

| Range | Distance | Targeting in DD Ranks |
| --- | --- | --- |
| Contact | Melee (0–2m) | Gap ≤ 1. See SPEC-03 Gap Matrix. |
| Courte | 2–15m | Gap ≤ 2. See SPEC-03 Gap Matrix. |
| Moyenne | 15–50m | Any rank on either side (except Gap > 4) |
| Longue | 50–300m | Any rank on either side (except Gap > 5) |
| Lointaine | 300m+ | Any rank on either side |

Firing beyond range: Not allowed. Attack is blocked by the system. UI shows "Out of range" per SPEC-31. Use the gap formula `(attackerRank − 1) + (targetRank − 1) ≤ weapon.maxGap` for authoritative validation. See SPEC-03 for the full Gap Matrix.

### 19.3 Demo Weapon Roster
Complete stat blocks for all weapons available in the demo. Every knight receives a Pistolet de Service + Marteau-Épieu as standard kit, plus class-specific weapons.

[v8.9] **Stat block column conventions:** Damage and Violence columns use explicit `XD6 + Y` format. `0D6 + N` means flat N with no dice roll. All values must be machine-parsable — never use bare numbers without the `D6` prefix.

### Pistolet de Service (Standard Ranged — 15 PG)
Service sidearm. All knights carry one. Dual-profile: contact knife + ranged pistol. [v8.9] Polymorphic weapon — morphs to action type. profileSwitchCost = FREE.

| Profile | Damage | Violence | Range | Energy | Effects |
| --- | --- | --- | --- | --- | --- |
| Contact (knife) | 1D6 + Force | 0D6 + 1 | Contact | — | Silencieux |
| Tir (pistol) | 2D6 + 6 | 1D6 | Moyenne | — | Silencieux |


### Marteau-Épieu (Standard Contact — 10 PG)
Heavy warhammer with built-in single-shot incendiary charge. Default melee for all knights. [v8.9] Swing vs trigger — weapon adapts. profileSwitchCost = FREE.

| Profile | Damage | Violence | Range | Energy | Effects |
| --- | --- | --- | --- | --- | --- |
| Contact | 3D6 + Force | 1D6 | Contact | — | Perce Armure 40 |
| Tir (charge) | 3D6 + 12 | 3D6 + 12 | Courte | — | Dégâts Continus 3, Dispersion 3, Lumière 2, Chargeur 1 |


### Couteau de Combat (Standard Contact — 15 PG)
Combat knife. Silent, precise. Favored by Rogues.

| Profile | Damage | Violence | Range | Energy | Effects |
| --- | --- | --- | --- | --- | --- |
| Contact | 3D6 + Force + Dextérité (Orfèvrerie) | 1D6 | Contact | — | Orfèvrerie, Silencieux, Jumelé (Ambidextrie) |


### Fusil de Précision (Standard Ranged — 40 PG)
Sniper rifle with camera-assisted targeting. Priest/Rogue favorite.

| Profile | Damage | Violence | Range | Energy | Effects |
| --- | --- | --- | --- | --- | --- |
| Tir | 4D6 + 6 + Tir (Précision) | 1D6 | Lointaine | — | Tir en Sécurité, Précision, Assistance à l'Attaque, Désignation, Deux Mains |


### Fusil d'Assaut (Standard Ranged — 30 PG)
Assault rifle. High violence for bande suppression.

| Profile | Damage | Violence | Range | Energy | Effects |
| --- | --- | --- | --- | --- | --- |
| Tir | 2D6 + 6 | 3D6 + 9 | Longue | — | Ultraviolence, Barrage 2, Deux Mains |


### Pistolet Mitrailleur (Standard Ranged — 25 PG)
Twin-barrel SMG. High rate of fire.

| Profile | Damage | Violence | Range | Energy | Effects |
| --- | --- | --- | --- | --- | --- |
| Tir | 3D6 | 4D6 | Moyenne | — | Meurtrier, Ultraviolence, Jumelé (Akimbo) |


### Shotgun Escamotable (Standard Ranged — 20 PG)
Foldable shotgun. Devastating at close range.

| Profile | Damage | Violence | Range | Energy | Effects |
| --- | --- | --- | --- | --- | --- |
| Tir | 2D6 + 10 | 2D6 | Moyenne | — | Meurtrier, Choc 1, Barrage 2, Deux Mains |


### Lance-Grenade Léger (Standard Ranged — 40 PG)
Grenade launcher. Three ammo types. [v8.9] Changing ammo type costs 1 Movement Action (manual magazine swap). profileSwitchCost = MOVEMENT_ACTION.

| Profile | Damage | Violence | Range | Energy | Effects |
| --- | --- | --- | --- | --- | --- |
| Explosive | 4D6 | 3D6 | Moyenne | — | Dispersion 3 |
| Antiblindage | 3D6 + 10 | 0D6 + 1 | Moyenne | — | Destructeur |
| Incendiaire | 2D6 | 3D6 + 10 | Moyenne | — | Dégâts Continus 3 |


### 19.4 Grenades Intelligentes
All knights receive 5 grenades per mission. Base stats: 3D6 damage, 3D6 violence, Courte range (max gap 2). [v8.9] **Force OD range extension:** each Force OD level increases the grenade's max gap by +1. Formula: `grenadeMaxGap = 2 + forceODLevel`. At Force OD 0: gap 2 (Courte). OD 1: gap 3. OD 2: gap 4 (Moyenne equivalent). OD 3: gap 5 (Longue equivalent). This extension does NOT consume OD — it is a passive benefit. Choose type before each throw. Grenade use = 1 Combat Action, base Tir. [v8.9] Hit resolution: Combo Roll with Tir as Base Characteristic. Compare total successes vs target's **Reaction** (grenades are ranged weapons — always Reaction, never Defense, regardless of range or rank). On hit: apply grenade effects to target; Dispersion X covers a blast zone of radius (X-1)/2 ranks centered on the target, hitting all combatants within (any faction — see SPEC-20.5). On miss: no effect. Grenades ignore weapon effects — they use only the grenade's own effect tags.

| Type | Additional Effects | Notes |
| --- | --- | --- |
| Shrapnel | Ultraviolence, Meurtrier, Dispersion 5 | Anti-personnel. Max carnage. |
| Flashbang | Choc 1 (auto), Barrage 2, Lumière 2, Dispersion 5 | No damage/violence dealt. Crowd control. **Friendly fire:** Dispersion is faction-agnostic — allied knights in the blast zone are tested vs Reaction. On hit: Choc 1 (auto) and Barrage 2 apply to allies. Extremely dangerous near friendlies. |
| Anti-Blindage | Destructeur, Perce Armure 20, Pénétrant 6, Dispersion 5 | Anti-armor. Bypasses defenses. |
| IEM | Parasitage 2, Dispersion 5 | No damage/violence dealt. Disables machines. **Friendly fire:** Dispersion is faction-agnostic — allied knights in the blast zone are tested vs Reaction. On hit: Parasitage 2 applies to allies (they lose 2 actions — meta-armor is technological). IEM is extremely dangerous to use near allies. |
| Explosive | Anti-Véhicule, +3D6 vs objects/vehicles, Choc 1, Dispersion 3 | Environmental destruction. |


### 19.5 Unarmed Strike
Any knight can punch/kick without a weapon:
In meta-armor: 1D6 + Force (+ OD bonuses) damage. 0D6 + 1 violence.
Without armor: Force score as damage. 0D6 + 1 violence.

### 19.6 Default Loadouts by Armor Class
Each armor class receives the following at mission start. Player may customize from available roster (weapons purchased with PG in future versions).

| Armor Class | Default Weapons | Rationale |
| --- | --- | --- |
| Warrior | Marteau-Épieu + Fusil d'Assaut + Pistolet de Service | Frontline melee + AoE suppression |
| Paladin | Marteau-Épieu + Shotgun Escamotable + Pistolet de Service | Tanking + close-range damage |
| Priest | Marteau-Épieu + Fusil de Précision + Pistolet de Service | Support + long-range precision |
| Rogue | 2× Couteau de Combat + Pistolet de Service | Dual-wield stealth + silent ranged |

All knights additionally carry 5 Grenades Intelligentes and 3 Nods of each type (Soin, Énergie, Armure).

### 19.7 Enemy Weapon Profiles
Enemy weapons use the same DamageProfile schema but are defined inline on EnemyBase (not purchased separately). See SPEC-21 (future) for demo enemy stat blocks with complete weapon data.

### Acceptance Criteria

Marteau-Épieu Lesté variant would be Force×2 + 3/OD. Default Marteau is NOT Lesté.

[v8] Grenade Flashbang: 0 damage, 0 violence. On hit (standard Combo Roll vs target Reaction): no damage or violence is dealt. Only weapon effects are applied: Choc 1 (auto-apply, no Chair threshold needed), Barrage 2, Lumière 2, Dispersion 5. The grenade exists purely for tactical effect application.  
[v8.9] Grenade Shrapnel thrown at enemy in same rank (Gap = 0): still resolves vs Reaction, NOT Defense. Grenades are always ranged.

All weapons resolve through SPEC-05 damage chain. Effects resolve per SPEC-20 glossary.


---

## [v5] SPEC-20: Weapon Effects Glossary
### Purpose
Canonical mechanical definitions for every weapon effect referenced in the spec. Each effect is a discrete system behavior that modifies the damage chain (SPEC-05), armor resolver (SPEC-06), or combat state.

### 20.1 Damage Modifier Effects
[v8.9] **Demo Scope column:** `YES` = at least one demo weapon/enemy uses this effect — implement fully. `SCAFFOLDED` = no demo weapon uses it — implement as stub (interface + no-op). Agents should prioritize YES effects.

| Effect | Demo Scope | Mechanic | Implementation Notes |
| --- | --- | --- | --- |
| Assistance à l'Attaque | YES | Each excess success above Defense/Reaction threshold = +1 flat damage (or +1 violence vs bandes). | Applied after hit confirmation, before routing to SPEC-06. |
| Dégâts Continus X | YES | Target takes X damage at start of each of their turns for 1D6 turns. Ignores CdF/Bouclier. [v8.9] **Stacking rule:** Cannot stack with itself on same target. A new application of Dégâts Continus on a target that already has an active DC effect **replaces** the existing one: use the higher X value (max of old and new), and roll a new 1D6 duration (replacing the old remaining duration). This is a refresh, not an addition — only one DC effect is ever active per target. | Track: target, remaining turns, X value. Duration rolled once on initial hit. On reapplication: `X = max(existingX, newX)`, `remainingTurns = roll(1D6)`. |
| Destructeur | YES | When damage reaches PA layer (at least 1 point passes CdF/Bouclier): +2D6 bonus damage added to total. PA still absorbs normally — Destructeur adds damage, it does NOT bypass PA. | Applied during SPEC-06 PA step. After confirming damage reaches PA, roll 2D6, add to running total before PA absorption. |
| En Chaîne | SCAFFOLDED | **SCAFFOLDED — no demo weapon uses En Chaîne.** On kill of Hostile or Salopard, spend 3 PE to make one free attack on another target within the same weapon's range band, same turn. New Combo Roll for the chain attack. One chain per kill. Implement as stub — full implementation deferred to post-demo weapons. | Trigger: target.isDead after damage resolution. 3 PE check. One chain per kill. |
| Fureur | SCAFFOLDED | **SCAFFOLDED — no demo weapon uses Fureur.** On kill, next attack this turn gets +2D6 damage. Implement as stub. | Trigger: target.isDead. Buff expires at end of turn. |
| Lesté | SCAFFOLDED | **SCAFFOLDED — no demo weapon uses Lesté (Marteau-Épieu is NOT Lesté).** Force is multiplied by ×2 (instead of ×1) for melee damage calculation. Implement as stub. | Modify SPEC-05 step 4: forceMode = LESTE. |
| Meurtrier | YES | [v6] When damage reaches PS (at least 1 point passes CdF, Bouclier, and PA): +2D6 bonus damage added to total. Does NOT skip or bypass any defensive layer. All layers resolve normally. [v8.9] **Chair Majeur immunity:** targets with Chair Exceptionnelle Majeur are immune to Meurtrier — the +2D6 bonus is NOT rolled (SPEC-18.3). | Applied during SPEC-06 PS step. After confirming damage reaches PS, roll 2D6, add to running total before PS reduction. Check target.hasChairMajeur before rolling. |
| Orfèvrerie | YES | Add Dextérité score (+1/OD) to damage, like Précision adds Tir. | SPEC-05 step 5. Stacks with Force for contact weapons. |
| Précision | YES | Add Tir score (+1/OD) to damage for ranged weapons. | SPEC-05 step 5. For PNJ: add Machine/2 (round down) instead. |
| Ultraviolence | YES | Violence dice that roll 1 or 2 are rerolled once. Keep new result. | Reroll mechanic for violence (anti-bande). NOT related to Meurtrier (+2D6 bonus). |


### 20.2 Defense Bypass Effects

| Effect | Demo Scope | Mechanic | Implementation Notes |
| --- | --- | --- | --- |
| Anti-Anathème | SCAFFOLDED | **SCAFFOLDED — no demo weapon uses Anti-Anathème.** Ignores Bouclier AND Chair Exceptionnelle entirely. PA then PS are hit directly. Implement as stub. | Used vs Anathème creatures. Bypasses their special defenses. |
| Anti-Véhicule | YES | Ignores Colosse tranche rule (damage applies 1:1). Also instantly destroys common vehicles. | |
| Ignore Armure | YES | Damage bypasses PA entirely, hitting PS directly. CdF/Bouclier still applies. | Skip PA step in SPEC-06. [v6] Distinct from Destructeur: Ignore Armure skips PA entirely, Destructeur adds +2D6 when damage reaches PA but PA still absorbs. |
| Ignore CdF | YES | Damage bypasses Champ de Force. PA then PS take full damage. | Skip CdF step in SPEC-06. Does NOT bypass Bouclier. |
| Pénétrant X | YES | Reduces target's effective CdF by X for this attack (minimum 0). Bouclier is NOT affected. | Reduce effective CdF by X (min 0). Does NOT affect Bouclier. |
| Perce Armure X | YES | Reduces target's effective PA by X for this attack (minimum 0). Damage that exceeds remaining PA goes to PS. | Reduce effective PA by X (min 0). Remaining damage after CdF goes straight to PS. |


### 20.3 Crowd Control Effects

| Effect | Demo Scope | Mechanic | Implementation Notes |
| --- | --- | --- | --- |
| Barrage X | YES | [v8.9] **Suppress (Barrage)** is an alternative to Attack. Player chooses 'Attack' or 'Suppress' before the action. Suppress: costs 1 Combat Action. No attack roll, no damage dealt, no other weapon effects apply (Ultraviolence, Meurtrier, etc. are inactive). Target must be within the weapon's range band and subject to standard gap/position validation (SPEC-09). Line of sight required. Target's Defense AND Reaction reduced by X until start of the **suppressor's** next turn (tied to the character who applied the Suppress, not the target). This means allies acting between the suppressor and the target's turn benefit from the debuff. **Expiry rule:** Barrage debuffs expire at the start of the suppressor's next turn. **If the suppressor dies or enters permanent incapacitation (permanent Despair, permadeath) before their next turn, all Barrage debuffs from that source expire immediately.** Temporary incapacitation (Agony with potential rescue, temporary Despair) does NOT expire Barrage — the debuff persists until the suppressor's next turn would occur (the turn is skipped, but the expiry event still fires at the suppressor's initiative slot). **Stacking rules:** Multiple Barrage effects from **different sources** stack additively. A "source" = one specific weapon instance wielded by one specific character. The same source (same weapon, same character) **cannot** Suppress the same target twice — a second Suppress with the same weapon on the same target is rejected. Different characters' weapons are always different sources. Ambidextre dual-wield: each weapon is a separate source (see below). **Ambidextre Suppress:** A character in Ambidextre style with two one-handed Barrage weapons can Suppress with both weapons as 1 Combat Action (same as Ambidextre attack = 2 separate actions for 1 Combat Action). Each weapon targets independently (same or different targets). Each weapon is a separate source — both can Suppress the same target and their Barrage values stack. No −3 dice penalty applies (Suppress has no attack roll). **Chair Majeur immunity:** targets with Chair Exceptionnelle Majeur are immune to Barrage — the effect is ignored entirely (SPEC-18.3). | Costs 1 Combat Action. No attack roll needed. Track active Barrage debuffs per target as list of {sourceCharId, sourceWeaponId, value, expiryOnSourceTurnStart}. Each entry expires when the source character's next turn begins, or immediately if the source dies/enters permanent incapacitation. On Suppress: check no existing entry with same sourceCharId + sourceWeaponId on this target. Sum all active Barrage values for Defense/Reaction reduction. Check target.hasChairMajeur before applying. |
| Choc X | YES | **Standard Choc (default):** If attack roll exceeds target's `floor(Chair/2)`: target loses X next actions. **Auto-Choc:** Always applies on hit — skip the `Chair/2` comparison entirely. [v8.9] **Default rule: Choc is standard unless the weapon/grenade explicitly specifies `isAutoChoc = true`.** If a stat block lists "Choc X" without further qualifier, it is standard Choc. In demo, only Flashbang grenade has `isAutoChoc = true`. All enemy weapons (Bestian Flot de ténèbres Choc 1, Behemot Griffes Choc 2, Ours Griffes Choc 1) use **standard** Choc. **Anathème interaction:** Choc is a post-hit effect (§20.6 Resolution Order). Anathème only changes the damage routing (CdF → PEs instead of CdF → PA → PS). The attack roll and hit determination are standard. Therefore Choc applies normally on Anathème attacks — if the attack hits and the Choc threshold is met, the target loses actions regardless of how damage is routed. **"Loses X actions"** means X total actions of **any type** (Combat or Movement). The target skips the next X actions they would normally take. Choc 1 = skip 1 action (e.g., lose Combat Action, keep Movement). Choc 2 = skip 2 actions (e.g., lose both Combat and Movement = full turn lost for a standard combatant). For enemies with Actions Multiples: Choc 2 removes 2 of their (1+N) Combat Actions + 1 Movement Action pool — the enemy still gets remaining actions if any. Does not stack while already under Choc. Does not stack with Parasitage. [v8.9] **Chair Majeur immunity:** targets with Chair Exceptionnelle Majeur are immune to Choc — **both** standard and auto-Choc are ignored entirely (SPEC-18.3). Auto-Choc does NOT bypass Chair Majeur immunity. | Check after hit: standard Choc → successes > floor(Chair/2). Auto-Choc → skip comparison, always apply. **Default: standard Choc.** Both variants → check target.hasChairMajeur first; if true, skip entirely. [v8.9] **Stacking rule:** If target already has an active Choc effect (`chocActionsRemaining > 0`), any new Choc application is **ignored entirely** — no refresh, no replacement, no upgrade to a higher X value. The original Choc runs its full course. Once all Choc actions are consumed (`chocActionsRemaining == 0`), the target is eligible for a new Choc. Same mutual exclusion applies to Parasitage: if Parasitage is active, Choc is also blocked (and vice versa). |
| Démoralisant | SCAFFOLDED | **SCAFFOLDED — no demo weapon uses Démoralisant.** Reduces bande Débordement by 2 for the current turn only. Multiple applications add duration, not extra reduction. Implement as stub. | Modify SPEC-11 débordement calculation for current turn. |
| Dispersion X | YES | [v8.9] Explosion centered on primary target. X = total positions affected (primary + collateral). The blast covers a symmetric radius of `(X-1)/2` ranks outward from the target on the linear track (KR4–KR3–KR2–KR1–ER1–ER2–ER3–ER4). X is always odd (1, 3, 5, 7). ALL combatants (any faction) occupying positions within the radius are hit — enemies AND allies. One attack roll for the primary; compare vs each splash target's Defense (contact) or Reaction (ranged) separately. All hit targets take full damage. **Tabletop conversion note:** tabletop Dispersion values may need rounding to the nearest odd number for the DD rank system. | See §20.5 for full geometry, radius table, examples, and friendly fire rules. |
| Lumière X | YES | Target Anathème creatures with Hypersensibilité Lumineuse: disables 1+ capacities for 1D6 turns AND doubles damage dealt by this weapon. X is intensity. | Check target.hasHypersensibilite. Double damage in SPEC-05 step 8 (after all bonuses). |
| Parasitage X | YES | Target loses X next actions (same action-loss mechanic as Choc — X total actions of any type). Affects enemies with the Machine seigneur tag OR any combatant with active technological systems — **knights in meta-armor ARE vulnerable** (meta-armor is technological). IEM grenades can disable allied knights in the Dispersion zone. Cannot stack while active. Cannot stack with Choc (mutual exclusion — if Choc is active, Parasitage is blocked and vice versa). | Similar to Choc but for machine/technological targets. Track `parasitageActionsRemaining` the same way as Choc. Mutual exclusion with Choc. |


### 20.4 Targeting & Utility Effects

| Effect | Demo Scope | Mechanic | Implementation Notes |
| --- | --- | --- | --- |
| Artillerie | SCAFFOLDED | **SCAFFOLDED — no demo weapon uses Artillerie (depends on Désignation which is also scaffolded).** [v6] Can fire without line of sight if target was previously Désignated. Ignores target's Cover bonus (+3 Reaction → +0). [v8.9] Artillerie fires over/around cover — it hits the combatant directly and does NOT damage or destroy the cover object. Even with future destructible cover, Artillerie bypasses cover entirely. Implement as stub. | Requires Désignation active on target. See SPEC-03 Cover System. |
| Assassin X | SCAFFOLDED | **SCAFFOLDED — no demo weapon uses Assassin, and surprise mechanics are undefined in demo.** Adds XD6 to damage during surprise attacks only. Surprise attack definition deferred to post-demo (requires ambush/stealth encounter design). | Trigger: isSurpriseAttack flag on combat init. Implement as stub. |
| Cadence X | SCAFFOLDED | **SCAFFOLDED — no demo weapon or enemy uses Cadence.** Weapon can target X enemies per Combat Action (instead of 1). Each attack has −3 dice. | X separate attack rolls, each at −3 dice. [v8.9] **Targeting constraint:** all Cadence targets must be on **adjacent enemy ranks** (within 1 rank of each other, i.e., `abs(targetA.rank - targetB.rank) <= 1`). This is adjacency between targets, NOT gap from the attacker. Each target must also independently be within the weapon's range from the attacker (standard gap/position rules apply per SPEC-09). Implement as stub — full implementation deferred to post-demo weapons. |
| Chargeur X | YES | Weapon has X uses per mission. At 0: weapon cannot fire. Refill at Camelot. | Track uses remaining. See SPEC-04 §4.4 L4 rules. |
| Désignation | SCAFFOLDED | **SCAFFOLDED — Fusil de Précision has this effect but full mechanic is deferred to post-demo.** Désignation is a Free Action (once per turn). The knight declares a target (not a bande) — that target gains the 'Designated' status, which persists until the designating knight marks a new target or dies. ALL allies gain +1 auto-success on attacks against the Designated target. Ignore Échec Critique vs the Designated target. Implement as stub: store `designatedTargetId` on the designating knight, apply +1 auto-success modifier during attack resolution. | Set designated flag on target. Persists until new target designated or designator dies. |
| Deux Mains | YES | Requires both hands. Cannot dual-wield or use shield simultaneously. On Ranged weapons: also enables Style Pilonnage (SPEC-04). | Equipment validation: block incompatible combinations. |
| Jumelé (Akimbo) | YES | Reduces Akimbo style attack penalty by 2 (from −3 to −1). Weapon must be one-handed. Two identical weapons required. See SPEC-04 Dual-Wielding Rules. | If weapon.hasJumeleAkimbo: dualWieldPenalty = −1 instead of −3. |
| Jumelé (Ambidextrie) | YES | Reduces Ambidextre style attack penalty by 2 (from −3 to −1). Weapon must be one-handed. Two different weapons allowed. See SPEC-04 Dual-Wielding Rules. | If weapon.hasJumeleAmbidextrie: dualWieldPenalty = −1 instead of −3. |
| Lourd | SCAFFOLDED | **SCAFFOLDED — no demo weapon uses Lourd.** Cannot move and attack in same turn (movement action forfeited). Enables Style Puissant only. (Style Pilonnage requires Deux Mains on a Ranged weapon — see Deux Mains effect.) Implement as stub. | If weapon.hasLourd: attack = forfeit movement. Unlock Style Puissant. |
| Silencieux | YES | User not detectable by sound when attacking. From stealth/Ghost/Changeling: add Discrétion (+OD) to damage. | SPEC-05 step 5. Required for Rogue Ghost ranged attacks (SPEC-17.4). |
| Tir en Sécurité | YES | [v6] Can fire from a Cover rank without the −3 dice penalty. Cover's +3 Reaction bonus remains active on the shooter. | See SPEC-03 Cover System. Knight retains full defensive benefit while attacking. |


### 20.5 Dispersion Adaptation for DD Ranks
[v8.9] **Complete rewrite.** Dispersion models an explosion centered on the primary target. The blast radiates outward symmetrically and hits all combatants within the blast radius, regardless of faction.

#### Linear Track Model
The battlefield is treated as a single 8-position linear track:
```
KR4 — KR3 — KR2 — KR1 — | — ER1 — ER2 — ER3 — ER4
pos1   pos2   pos3   pos4   pos5   pos6   pos7   pos8
```
**Distance** between any two positions = `abs(posA - posB)`. KR1 (pos4) and ER1 (pos5) are 1 step apart — they are adjacent across the center line.

#### Dispersion X — Radius Model
**X = total positions covered** by the explosion (primary target + collateral). X is always **odd** (1, 3, 5, 7).
**Radius** = `(X - 1) / 2` ranks outward from the target in each direction.

| Dispersion X | Total positions | Radius (ranks) | Pattern relative to target |
|---|---|---|---|
| 1 | 1 | 0 | Target only |
| 3 | 3 | 1 | Target ± 1 rank |
| 5 | 5 | 2 | Target ± 2 ranks |
| 7 | 7 | 3 | Target ± 3 ranks (max — covers almost entire track) |

**ALL combatants** (any faction — enemies AND allies) occupying positions within the radius are hit. Empty positions are skipped (they don't consume a "slot"). The explosion doesn't pick targets — it covers a zone.

**Edge of track:** If the radius extends beyond the track (e.g., Dispersion 5 at ER4 — radius 2 would reach pos10, which doesn't exist), the explosion simply doesn't reach those non-existent positions. No wraparound. The explosion is truncated at the track edges.

#### Dispersion Resolution

**Resolution sequence:**
1. The **primary target** is resolved by the standard attack roll (SPEC-04 Combo Roll → SPEC-05 Damage Chain). This is the normal attack — Dispersion does not change it.
2. **One damage roll** is performed for the primary target. This same damage result is reused for all splash targets (no separate damage rolls per target). [v8.9] **If the primary attack misses, a damage roll is still performed** — it is used exclusively for splash target resolution (the primary target takes no damage since the attack missed).
3. [v8.9] **Splash fires regardless of primary hit or miss.** The explosion happens at the target location whether or not the primary target was hit (the projectile still detonates). **Splash targets** (all combatants within the blast radius, excluding the primary): the primary attack roll's **total successes** are compared against each splash target's individual Defense (contact) or Reaction (ranged) threshold. No separate attack roll for splash targets. If the primary roll's successes exceed a splash target's threshold → that target is hit and takes the **same damage amount** as the primary would have, routed through their own ArmourLayerResolver (SPEC-06) independently.
4. Splash targets that are not hit (successes ≤ their threshold) take no damage.
5. **Weapon effects on splash targets:** All weapon effects carried by the Dispersion weapon (Choc, Barrage, Dégâts Continus, Meurtrier, etc.) apply independently to each splash target that is hit. For example: Flashbang (Choc 1 auto, Barrage 2, Dispersion 5) — each hit splash target receives Choc 1 (auto, no Chair check — but Chair Majeur immunity still applies per M-03) and Barrage 2 independently. For zero-damage weapons (Flashbang, IEM): splash targets that are hit receive the weapon's effects even though no damage is dealt.

```python
function resolveDispersion(primaryTarget, X, attackSuccesses, damageResult, allCombatants):
  radius = (X - 1) / 2
  primaryPos = getLinearPosition(primaryTarget)
  minPos = max(1, primaryPos - radius)
  maxPos = min(8, primaryPos + radius)
  for each combatant in allCombatants:
    if combatant == primaryTarget: continue  # already resolved by normal attack
    if combatant.isBande: continue  # Bandes are off-track
    combatantPos = getLinearPosition(combatant)
    if combatantPos >= minPos AND combatantPos <= maxPos:
      threshold = combatant.reaction if weapon.isRanged else combatant.defense
      if attackSuccesses > threshold:
        routeThroughArmourLayerResolver(combatant, damageResult)  # same damage as primary
```

#### Friendly Fire
**Any combatant** within the blast radius takes damage — including allied knights. The explosion doesn't discriminate by faction. All hit targets (primary and splash) take the **same damage amount** (one damage roll, reused for all), routed through each target's individual ArmourLayerResolver (SPEC-06). Different targets may take different final damage due to different CdF/PA/PS values.
- **Knight attacks enemy:** Allied knights within the radius are hit.
- **Enemy attacks knight:** Allied enemies within the radius are hit.
- **UI warning:** Before confirming a Dispersion attack, the UI must highlight the blast zone (all positions within radius) and require confirmation if allies would be hit (SPEC-31).

#### Bande Exception
Bandes are **off-track** (no position). Dispersion does NOT splash onto bandes. Use Violence to damage bandes (SPEC-11).

#### Tabletop Conversion
Tabletop Dispersion values (often even numbers) must be converted to odd numbers for the DD rank radius system. Rule: **round up to nearest odd number.** Tabletop Dispersion 6 → DD Dispersion 7... but this would be too wide (radius 3 = 7 positions). For balance, we cap at Dispersion 5 (radius 2) for grenades. Tabletop Dispersion 2 → DD Dispersion 3 (radius 1). Tabletop Dispersion 4 → DD Dispersion 5 (radius 2). This conversion should be done when authoring weapon data, not at runtime. Each converted value should be validated for game balance.

| Tabletop Value | DD Conversion | Radius | Notes |
|---|---|---|---|
| 1 | 1 | 0 | Target only |
| 2 | 3 | 1 | Round up |
| 3 | 3 | 1 | Already odd |
| 4 | 5 | 2 | Round up |
| 5 | 5 | 2 | Already odd |
| 6 | 5 | 2 | Capped for balance (7 would be radius 3 = nearly entire track) |

#### Worked Examples

**Example 1 — Grenade Shrapnel (Dispersion 5, radius 2) at enemy mid-rank:**
```
KR4(K4) — KR3(K3) — KR2(K2) — KR1(K1) — | — ER1(E1) — ER2(E2) — ER3(E3) — ER4(E4)
pos1        pos2       pos3       pos4        pos5        pos6       pos7       pos8
```
K3 throws Shrapnel at E2 (pos6). Radius 2: blast zone = pos4 to pos8.
Positions in blast: K1(pos4), E1(pos5), E2(pos6=target), E3(pos7), E4(pos8).
→ Hits 3 enemies (E1, E3, E4) + 1 ally (K1). K2 at pos3 is outside blast zone — safe.
**Tactical note:** Targeting E2 instead of E1 pulls the blast zone 1 rank deeper into enemy territory, saving K2 from friendly fire.

**Example 2 — Grenade Shrapnel (Dispersion 5, radius 2) at enemy backline:**
```
KR4(K4) — KR3(K3) — KR2(K2) — KR1(K1) — | — ER1(E1) — ER2(—) — ER3(E3) — ER4(E4)
pos1        pos2       pos3       pos4        pos5        pos6       pos7       pos8
```
K3 throws Shrapnel at E4 (pos8). Radius 2: blast zone = pos6 to pos8 (truncated — pos9/10 don't exist).
Positions in blast: ER2(pos6=empty), E3(pos7), E4(pos8=target).
→ Hits 1 enemy (E3) + 0 allies. E1 at pos5 is outside the blast. Perfectly safe for knights!
**Tactical note:** Targeting enemy backline with Dispersion 5 is very safe — the radius doesn't reach the center line.

**Example 3 — Grenade Shrapnel (Dispersion 5, radius 2) at enemy frontline:**
```
KR4(K4) — KR3(K3) — KR2(K2) — KR1(K1) — | — ER1(E1) — ER2(E2) — ER3(E3) — ER4(E4)
pos1        pos2       pos3       pos4        pos5        pos6       pos7       pos8
```
K1 throws Shrapnel at E1 (pos5). Radius 2: blast zone = pos3 to pos7.
Positions in blast: K2(pos3), K1(pos4), E1(pos5=target), E2(pos6), E3(pos7).
→ Hits 2 enemies (E2, E3) + 2 allies (K1 self-hit, K2). E4 at pos8 is outside. K3/K4 safe.
**Tactical note:** Frontline grenade is extremely risky — the blast crosses the center line and hits 2 allies.

**Example 4 — Lance-Grenade Explosive (Dispersion 3, radius 1) at mid-rank:**
```
KR4(—) — KR3(K3) — KR2(K2) — KR1(K1) — | — ER1(E1) — ER2(E2) — ER3(E3) — ER4(—)
pos1       pos2       pos3       pos4        pos5        pos6       pos7       pos8
```
K3 fires Lance-Grenade at E2 (pos6). Radius 1: blast zone = pos5 to pos7.
Positions in blast: E1(pos5), E2(pos6=target), E3(pos7).
→ Hits 2 enemies (E1, E3) + 0 allies. K1 at pos4 is outside. Clean hit!
**Tactical note:** Dispersion 3 (radius 1) is surgical — only ±1 rank from target. Friendly fire is rare unless targeting the frontline.

**Example 5 — Marteau-Épieu Tir charge (Dispersion 3, radius 1) at point blank:**
```
KR2(K2) — KR1(K1) — | — ER1(E1) — ER2(E2)
pos3        pos4        pos5        pos6
```
K1 fires incendiary charge at E1 (pos5). Radius 1: blast zone = pos4 to pos6.
Positions in blast: K1(pos4), E1(pos5=target), E2(pos6).
→ Hits 1 enemy (E2) + K1 self-hit (friendly fire). Point-blank Marteau charge is dangerous!

**Tactical takeaways:**
- **Dispersion 3 (radius 1)** is surgical. Safe as long as you don't target the frontline enemy from your own frontline.
- **Dispersion 5 (radius 2)** is powerful but risky at mid-range. Targeting backline enemies is safe; frontline is dangerous.
- **Targeting enemy backline** minimizes friendly fire (blast zone stays on enemy side).
- **Targeting enemy frontline** pulls the blast zone across the center line — high friendly fire risk.
- **Positioning matters:** keep allies away from the center line when using Dispersion weapons.
- **The radius is predictable:** players can always calculate exactly who will be hit by counting ranks from the target.

### 20.6 Effect Resolution Order
When a weapon has multiple effects, resolve in this order:
Pre-hit effects: Désignation (free), Barrage (replaces damage).
Hit determination: Attack roll vs threshold.
Damage modification: Précision/Orfèvrerie/Silencieux (flat bonus), Lesté (Force multiplier), Assassin (surprise bonus), Fureur (kill chain bonus), Ultraviolence (reroll 1-2 on violence dice).
Defense bypass: Anti-Anathème, Ignore CdF, Pénétrant, Ignore Armure, Perce Armure, Anti-Véhicule.
Layer-triggered bonuses: [v6] Destructeur (+2D6 on reaching PA), Meurtrier (+2D6 on reaching PS). Applied during SPEC-06 layer resolution.
Post-hit effects: Choc/Parasitage (action loss), Dégâts Continus (DoT apply), En Chaîne (chain kill), Lumière (vs Anathème).
AoE resolution: Dispersion (additional targets), Cadence (multi-target).

### Acceptance Criteria

[v8.9] Cadence 2 (SCAFFOLDED): Knight at Rank 1 attacks with Cadence 2 weapon (Moyenne range). Primary target E1 at enemy Rank 1. Extra targets: E2 at enemy Rank 2 (1 rank from E1 → valid, adjacent), E3 at enemy Rank 4 (3 ranks from E1 → invalid, too far). Knight attacks E1 and E2 only, each at −3 dice.  
Perce Armure 40 vs PA 30: effectivePA = max(30−40,0) = 0. PA absorbs nothing (fully pierced). PA stays at 30 (unchanged). All remaining damage hits PS.  
Perce Armure 40 vs PA 60: effectivePA = max(60−40,0) = 20. PA absorbs up to 20. PA reduced by absorbed amount only (e.g., PA 60 → 40 if 20 absorbed). Remaining damage after absorption hits PS.  
Pénétrant 10 vs CdF 8: CdF fully ignored. vs Bouclier 8: Bouclier still applies (Pénétrant doesn't bypass Bouclier).  
Choc 2 auto: target loses 2 actions regardless of Chair score.  
Dispersion 3 (radius 1): target E2 at ER2 (pos6). Blast zone pos5–pos7. E1(pos5) and E3(pos7) hit. K1(pos4) outside — safe. 2 collateral enemies, 0 friendly fire.
[v8.9] Dispersion 5 (radius 2) at ER1: target E1 (pos5). Blast zone pos3–pos7. K2(pos3), K1(pos4), E2(pos6), E3(pos7) hit. 2 enemies + 2 allies (K1, K2) friendly fire. K3(pos2) outside — safe.  
[v8.9] Barrage stacking: C1 (Barrage 2 weapon) Suppresses E1: E1 gets −2 Def/React. C1 tries to Suppress E1 again with same weapon: rejected (same source). C2 (Barrage 2 weapon) Suppresses E1: E1 now −4 Def/React (two different sources stack).  
[v8.9] Barrage Ambidextre: C3 has main-hand Barrage 2 + off-hand Barrage 4 in Ambidextre. Uses 1 Combat Action to Suppress E1 with both weapons. E1 gets −2 (main) + −4 (off) = −6 Def/React. Each weapon is a separate source.  
[v8.9] Barrage + Attack combo: C1 has Actions Multiples (1) = 2 Combat Actions. Uses 1st Combat Action to Suppress E1 (−2 Def/React). Uses 2nd Combat Action to Attack E1 (benefits from −2 Def/React).

#### [v8.9] Barrage Tactical Worked Example — 3 Turns, 4 Knights vs 3 Faune

**Setup:**
- K1 (Warrior, Init 12): Fusil d'Assaut (2D6+6, Longue, Barrage 2). Tir 4, OD 1.
- K2 (Paladin, Init 8): Shotgun Escamotable (2D6+10, Moyenne, Barrage 2, Meurtrier, Choc 1). Tir 3, OD 0.
- K3 (Priest, Init 5): Fusil de Précision (4D6+6+Tir, Lointaine, Précision). Tir 5, OD 2. The ranged heavy hitter.
- K4 (Warrior, Init 4): Marteau-Épieu Contact (3D6+Force, Contact, Perce Armure 40). Force 5, Combat 5, OD 2. The melee heavy hitter. Positioned at Rank 1.
- F1, F2, F3: Faune (Defense 6, Reaction 2, PS 80, no PA/Bouclier). Init 3 each. All at enemy Rank 1 (ADVANCE).

**Turn 1 — Focus fire on F1 (ranged + melee):**
```
K1 (Init 12): Suppress F1 with Fusil d'Assaut.
  → F1 Barrage debuffs: [{source: K1/FusilAssaut, value: 2}]
  → F1 effective Defense: 6 − 2 = 4. Reaction: 2 − 2 = 0.
  → Barrage expires: start of K1's next turn (Turn 2, Init 12).

K2 (Init 8): Suppress F1 with Shotgun Escamotable.
  → F1 Barrage debuffs: [{K1/FusilAssaut, 2}, {K2/Shotgun, 2}]
  → F1 effective Defense: 6 − 4 = 2. Reaction: 2 − 4 = −2 → floored to 0.
  → Barrage expires: start of K2's next turn (Turn 2, Init 8).

K3 (Init 5): Attack F1 with Fusil de Précision (ranged → vs Reaction).
  → F1 effective Reaction = 0 (both Barrages active).
  → K3 rolls 5+3 (Combo) = 8D6. Rolls [2,4,1,6,3,2,5,4] = 5 evens + 2 OD = 7 successes.
  → 7 > 0 = HIT (trivial with Barrage debuff on Reaction).
  → Damage: 4D6 rolls [5,3,6,4] = 18 + 6 flat + 5 (Tir) + 2 (OD Précision) = 31 raw.
  → No PA, no Bouclier, no CdF. F1 PS: 80 − 31 = 49 PS.

K4 (Init 4): Attack F1 with Marteau-Épieu Contact (melee → vs Defense).
  → F1 effective Defense = 2 (normally 6, but Barrage −4 from two sources).
  → K4 rolls Combat 5 + Hargne 4 (Combo) = 9D6. Rolls [6,2,4,1,3,6,2,5,4] = 6 evens + 2 OD = 8 successes.
  → 8 > 2 = HIT (would have been 8 > 6 without Barrage — still hits, but with 6 excess instead of 2).
  → Excess successes: 8 − 2 = 6 (with Barrage) vs 8 − 6 = 2 (without).
  → Damage: 3D6 rolls [5,4,6] = 15 + 5 (Force) = 20 raw.
  → F1 PS: 49 − 20 = 29 PS. Barrage made the melee attack far more decisive.

F1, F2, F3 (Init 3): Faune act. F1 charges K2 (Charge Brutale). F2, F3 advance.
```

**Turn 2 — Finish F1, set up F2:**
```
K1 (Init 12): Start of K1's turn → K1's Barrage on F1 expires.
  → F1 Barrage debuffs: [{K2/Shotgun, 2}] (K2's hasn't expired yet).
  → F1 effective Defense: 6 − 2 = 4. Reaction: 2 − 2 = 0.
  K1 Attacks F1 (ranged, finish it off) vs Reaction 0. Hits easily.
  → Damage: 2D6 rolls [4,3] = 7 + 6 flat = 13 raw. F1 PS: 29 − 13 = 16 PS.

K2 (Init 8): Start of K2's turn → K2's Barrage on F1 expires.
  → F1 Barrage debuffs: [] (all expired). F1 Defense restored to 6, Reaction to 2.
  K2 Suppresses F2 (set up next target).
  → F2 Barrage debuffs: [{K2/Shotgun, 2}]
  → F2 effective Defense: 6 − 2 = 4. Reaction: 2 − 2 = 0.

K3 (Init 5): Attack F1 to finish it (ranged vs Reaction 2 — no Barrage on F1 anymore).
  → K3 rolls 8D6. Gets 6 successes + 2 OD = 8. 8 > 2 = HIT.
  → Damage: 4D6 rolls [6,5,2,6] = 19 + 6 + 5 + 2 = 32 raw. F1 PS: 16 − 32 = −16.
  → F1 DEAD. K3 earns +1 Héroïsme.

K4 (Init 4): Attack F2 with Marteau-Épieu (melee vs Defense).
  → F2 effective Defense = 4 (6 − 2 from K2's Barrage, still active).
  → K4 rolls 9D6. Gets 5 evens + 2 OD = 7. 7 > 4 = HIT.
  → Damage: 3D6 rolls [3,6,2] = 11 + 5 (Force) = 16 raw. F2 PS: 80 − 16 = 64 PS.

F2, F3 (Init 3): F2 attacks K4 (nearest). F3 advances to Rank 1.
```

**Turn 3 — Double Suppress F2, heavy burst:**
```
K1 (Init 12): Suppress F2 with Fusil d'Assaut.
  → F2 Barrage debuffs: [{K2/Shotgun, 2}, {K1/FusilAssaut, 2}]
  → F2 effective Defense: 6 − 4 = 2. Reaction: 2 − 4 = −2 → floored to 0.
  → K2's Barrage on F2 still active (doesn't expire until K2's turn at Init 8).

K2 (Init 8): Start of K2's turn → K2's Barrage on F2 expires.
  → F2 Barrage debuffs: [{K1/FusilAssaut, 2}]
  → F2 effective Defense: 6 − 2 = 4. Reaction: 2 − 2 = 0.
  K2 Attacks F2 with Shotgun (ranged vs Reaction 0). Easy hit.
  → Damage: 2D6 rolls [5,6] = 11 + 10 flat = 21 raw.
  → Meurtrier: damage reaches PS (no PA) → +2D6 rolls [4,3] = 7 bonus. Total = 28 raw.
  → F2 PS: 64 − 28 = 36 PS.

K3 (Init 5): Attack F2 with Fusil de Précision (ranged vs Reaction 0, K1's Barrage still active).
  → Easy hit. Damage: ~31 raw. F2 PS: 36 − 31 = 5 PS.

K4 (Init 4): Attack F2 with Marteau-Épieu (melee vs Defense 4, K1's Barrage still active).
  → K4 rolls 9D6. Gets 5 evens + 2 OD = 7. 7 > 4 = HIT.
  → Damage: 3D6 rolls [6,4,5] = 15 + 5 (Force) = 20 raw. F2 PS: 5 − 20 = −15.
  → F2 DEAD. K4 earns +1 Héroïsme.

F3 (Init 3): F3 attacks K4 (nearest).
```

**Tactical takeaways:**
- **Barrage reduces BOTH Defense and Reaction.** Melee attackers (K4 vs Defense) and ranged attackers (K3 vs Reaction) both benefit.
- **Faune Defense 6 → 2 with double Barrage** makes the melee warrior's excess successes jump from 2 to 6, dramatically increasing hit reliability.
- **Faune Reaction 2 → 0 with double Barrage** guarantees ranged hits for the sniper.
- **Duration matters:** K1's Barrage (Init 12) persists through K2 (Init 8), K3 (Init 5), K4 (Init 4), and the Faune turn (Init 3) — the entire round benefits. K2's Barrage (Init 8) expires sooner but still covers K3 and K4.
- **Suppress is a team force multiplier:** Two knights sacrificing their attacks to Suppress create a devastating window for the other two to exploit.  




End of SPEC-20. Continue to SPEC-21 for demo enemy roster. The encounter compositions defined in SPEC-22 (Demo Mission Structure) must be authored as EncounterSO instances per SPEC-29 (Encounter Authoring Schema). Do not hardcode encounter data in scene logic.
