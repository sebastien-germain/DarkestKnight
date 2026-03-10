# Tarot Advantages & Disadvantages
> **SPECs:** 24
> **Domain:** `Assets/Data`
> **Dependencies:** None (data table)
> **Depended on by:** [01_KnightDataModel.md](./01_KnightDataModel.md) (SPEC-02 generation)
---
## [v5] SPEC-24: Tarot Advantages & Disadvantages
### Purpose
Complete mechanical definitions for all Tarot-derived advantages and disadvantages used in knight generation (SPEC-02 step 4). Each knight draws 5 cards from the full 22-card Major Arcana, selects 2 advantages and 1 disadvantage. [v6] Corrected: 5 cards drawn (not 3), full 22-card pool (not 12).

### 24.1 Full Tarot Pool
[v6] The demo uses the full 22-card Major Arcana (Tarot de Marseille, cards 0–XXI). Each card grants +1 Aspect point, +3 Characteristic points, 1 Advantage, and 1 Disadvantage. [v7] All 22 Major Arcana cards are fully defined below with complete mechanical advantage/disadvantage pairs. Note: Le Fou (0) and La Maison-Dieu (XVI) have special bonus structures — see 24.4. Incompatibility: L'Arcane sans Nom (XIII) and La Maison-Dieu (XVI) cannot both appear in the same knight's draw — if both are drawn, redraw one.

### 24.2 Advantages

| Tarot | Advantage | Mechanic |
| --- | --- | --- |
| I. Le Bateleur | Infatigable | No PS loss from PA damage (normally 1 PS per 5 PA lost in one hit). |
| II. La Papesse | Connaissance Secrète | One knowledge type (chosen at gen): difficulty reduced by 1 level for Savoir tests. |
| V. Le Pape | Forteresse Spirituelle | +5 max PEs at creation. |
| VII. Le Chariot | Sûr de Soi | +1 auto-success on all Parole tests vs humans. |
| VIII. La Justice | Bon Sens | +1 auto-success on any test to detect deception or manipulation (regardless of combo used). On Échec Critique for that test, treat as normal failure instead. |
| IX. L'Ermite | Esprit d'Acier | All PEs losses reduced by 1 (minimum 0). |
| X. La Roue | Chanceux | Once per mission: reroll all dice on one failed test. |
| XI. La Force | Dur à Cuire | +5 max PS at creation. |
| XIII. L'Arcane | Trompe la Mort | [v8.9] On Mort injury result: reroll the Injury Table, take the new result (whatever it is). **Once per mission.** If the knight rolls Mort a second time during the same mission (whether from a second Agony or from the reroll itself), the Death result is final — the knight dies permanently. Track `trompeLaMortUsed: boolean` on RunState, reset at mission start. |
| XIV. La Tempérance | Guérison Rapide | Nod de Soin: +3 flat PS bonus on top of the 3D6 roll (total = 3D6 + 3). Applies when the knight WITH this advantage uses a Nod de Soin on any target (self or ally). See [SPEC-33](./13_NodSystem.md) §33.6 (canonical). |
| XV. Le Diable | Instinct Animal | +10 Initiative during ambushes. Keep Def/React when surprised or flanked. |
| XIX. Le Soleil | Rayonnement | Allies within Courte range: +1 auto-success on all tests. Once per turn. |
| [v7] 0. Le Fou | Chevalier Véritable | Heroic Mode triggers at 4 Héroïsme (instead of 6). Total still capped at 6. Any malevolent act: lose 1 Héroïsme. |
| [v7] III. L'Impératrice | Mémoire Efficace | Once per mission: ask system for one lore hint about current encounter (enemy weak point, hidden path, etc.). |
| [v7] IV. L'Empereur | Magnétique | +1 auto-success on all Aura tests vs human NPCs. |
| [v7] VI. L'Amoureux | Aisance | Ignores the Lourd weapon effect. Still requires two hands to wield. |
| [v7] XII. Le Pendu | Code Moral | Knight gains 1 additional minor Motivation: "Respect du Code" — Tag: MN_CODE_MORAL. Trigger: knight completes a combat node without breaking their personal code. [v8.9] **Digital "code breach" conditions** (any of these during a combat node = code broken, motivation does NOT trigger): (1) **Friendly fire:** this knight's attack (including Dispersion X splash from any weapon — grenades, Lance-Grenade, Marteau-Épieu charge, etc.) deals damage to an allied knight. Track: `codeMoralBroken = true` whenever `applyDamage(target)` is called and target is an ally AND source is this knight. (2) **Fleeing from a downed ally:** this knight moves from Rank 1 to Rank 2+ while any allied knight at the same Rank 1 is in Agony. Code Moral functions as a 4th minor Motivation (+1D6 PEs per trigger). Mirror of Sacrifice Total disadvantage. |
| [v7] XVI. La Maison-Dieu | Soif d'Apprendre | Once per mission at Camelot: knight can train with another knight (the "teacher"). If teacher's target Characteristic is ≥2 higher than this knight's, this knight gains +1 in that Characteristic. PG cost for the increase is halved (rounded down). Characteristic still capped at parent Aspect. |
| [v7] XVII. L'Étoile | Rêves Prémonitoires | Once per mission: receive a cryptic hint about the current mission node (enemy type, trap, or objective clue). |
| [v7] XVIII. La Lune | Menteur Professionnel | All Parole combo Discrétion tests: difficulty reduced by 1 level. |
| [v7] XX. Le Jugement | Empathie | +1 auto-success on Instinct tests to read emotions/detect lies. Ignore crit fail on those tests. |
| [v7] XXI. Le Monde | Créateur-né | +1 additional Minor Motivation (Réaliser une Œuvre) — Tag: VOEU_MONDE_ART. Trigger condition: NEVER in demo. The Art system is not part of the demo and requires dedicated design for the full game. In Knight lore, Art is a surviving fragment of the Ether and Light — a source of hope that can counter Despair. The Créateur-né knight is one of the few remaining artists. In demo: this motivation exists in the data model, takes a 4th minor motivation slot, but provides no PEs recovery. The knight effectively has 3 active minor motivations + 1 dormant slot. This is the intended cost/benefit: Le Monde grants strong stat bonuses (+1 Chair, +3 Chair chars) but one wasted motivation slot until Art is implemented. FUTURE: when Art system is designed, this motivation will trigger on art creation/performance events and become one of the most powerful PEs recovery tools in the game. |


### 24.3 Disadvantages

| Tarot | Disadvantage | Mechanic |
| --- | --- | --- |
| I. Le Bateleur | Colérique | −3 dice on all Sang-Froid base tests when attempting to resist anger or calm down. Digital: once per mission, system triggers a rage event — knight must pass a Sang-Froid test (−3 dice) or lose next Combat Action (frozen in rage). |
| II. La Papesse | Curiosité Maladive | Once per mission: system forces knight to investigate a non-priority target (1D6 turns distracted). |
| V. Le Pape | Fanatique | Roleplay-driven. Digital: once per mission, knight may refuse to execute a tactically optimal order. |
| VII. Le Chariot | Ennemi Juré | Narrative: a recurring enemy NPC targets this knight specifically. +1 enemy in one random encounter. |
| VIII. La Justice | Trop Prudent | Initiative roll: 2D6 instead of 3D6 (+ bonuses). |
| IX. L'Ermite | Solitaire | [v8.9] When this knight acts as an **assistant** (SPEC-04 Assistance), the main Combo Roll's difficulty threshold is increased by +1. In combat: Defense +1 (melee) or Reaction +1 (ranged) for that roll only. In narrative tests: required successes +1. The assistant's successes still contribute normally — but the bar is harder to clear. Does NOT apply when this knight is the primary roller being assisted by others. |
| X. La Roue | Mauvaises Intuitions | Once per mission: one Instinct/Perception test auto-fails (system-triggered). |
| XI. La Force | Forcené | Once per mission: system forces knight to attack nearest enemy (no tactical choice). |
| XIII. L'Arcane | Vétéran | Aspect scores cannot exceed 7 (normally cap at 9). |
| XIV. La Tempérance | Immunité Déficiente | Poison/toxin damage: always maximum (no roll, take max face values). |
| XV. Le Diable | Brute | Machine aspect cannot exceed 5. |
| XIX. Le Soleil | Egoiste | Egoiste: Knight cannot serve as Adjuvant during Mode Héroïque (SPEC-09). Knight cannot use the Assist action on allies (SPEC-04). Self-centered — refuses to subordinate to others or support them directly. |
| [v7] 0. Le Fou | Trouble Mental | Knight has a phobia or compulsion (random at gen). Once per mission: system triggers phobia/compulsion, knight loses 1 combat action (frozen/distracted). |
| [v7] III. L'Impératrice | Esprit de Contradiction | [v8.9] **Trigger:** At the start of each of this knight's combat turns, roll a proc check: **30% chance** (use Combat RNG stream, `rng.Chance01(0.30f)`). On proc: system selects the **nearest allied knight** (by rank distance; ties broken randomly). That ally's **next Combo Roll** (regardless of when it occurs — this round, next round, or later) has **+1 difficulty level** (Defense/Reaction +1 for combat, required successes +1 for narrative). The debuff persists until consumed by exactly one Combo Roll, then is removed. **Assist interaction:** The +1 difficulty applies to the affected ally's next Combo Roll regardless of type (attack, assist, stabilization, Ghost stealth, etc.). For attacks: Defense/Reaction threshold +1. For Assist rolls: no threshold applies to the assist itself (it just contributes successes), so the debuff is effectively wasted — but it IS consumed (removed) by the Assist roll. The ally's next roll after the Assist will be debuff-free. The contrarian knight verbally contests the ally's plan (UI: brief dialogue popup). The proc can trigger **at most once per combat encounter** (track `espritDeContradictionUsed: boolean`, reset per encounter). If no ally is within any rank (solo knight), the disadvantage does not proc. |
| [v7] IV. L'Empereur | Présomptueux | Once per mission: system forces knight to attack highest-tier enemy present (boss/colosse over hostile), ignoring tactical priority. |
| [v7] VI. L'Amoureux | Graveleux | All Parole and Aura tests vs specific NPC types (random at gen): +1 difficulty level. |
| [v7] XII. Le Pendu | Sacrifice Total | Knight has no Major Motivation at character creation and cannot gain one during the game. SPEC-02 step 6: skip major motivation draw entirely (majorMotivation = null). Minor motivations function normally (+1D6 PEs per trigger). The knight permanently lacks the +25 PEs current+max bonus that other knights can earn. This is the price of the Code Moral advantage (4th minor motivation). |
| [v7] XVI. La Maison-Dieu | Amnésique | [v8.9] At character generation (SPEC-02 step 4), after 2 advantages and 1 disadvantage are selected: if Amnésique is the selected disadvantage, one of the knight's 2 active advantages is chosen at random and **replaced** by a random advantage drawn from the **full advantage table** (all 22 cards' advantages, not limited to the knight's drawn cards). The replacement must not duplicate the knight's remaining advantage. If it would, redraw (max 10 retries, then keep as-is). The original advantage is permanently lost — the knight's memory has overwritten it. This happens once at generation, not at runtime. |
| [v7] XVII. L'Étoile | Cauchemars | On rest/heal: roll 1D6. Odd = nightmare, no PS recovery from rest, lose 1 PEs. |
| [v7] XVIII. La Lune | Lunatique | Once per mission: system imposes a random mood shift on the knight. Roll 1D6: 1–2 = Melancholy (−1 die on all Dame tests this encounter), 3–4 = Reckless (−1 die on all Machine tests this encounter), 5–6 = Euphoric (−1 die on all Bête tests this encounter). Lasts entire encounter. |
| [v7] XX. Le Jugement | Prisonnier | [v8.9] Knight's Major Motivation is LOCKED to "Retrouver la Liberté" (Tag: MJ_PRISONNIER). Until fulfilled, knight **cannot recover or gain PEs from ANY source** (motivations, Nods, kills — all PEs gains are blocked). This is the harshest disadvantage — the knight is a ticking Despair bomb until freed. **Fulfillment: FUTURE CONTENT.** The fulfillment condition (complete 3 missions successfully) requires multi-mission tracking and is not achievable in the demo (single mission). In demo: the disadvantage is permanently active with no resolution. PEs are blocked for the entire run. On fulfillment (future): unlock PEs recovery, +25 PEs current and max (standard Major Motivation bonus). Implementation note: the PEs blocking mechanic must be implemented in demo. Only the fulfillment/unlock is deferred. |
| [v7] XXI. Le Monde | Porte-malheur | Once per mission: system converts one failed test (by this knight or any ally) into a critical failure. |


### 24.4 Aspect Bonuses per Card

| Tarot | Aspect +1 | Characteristic Points |
| --- | --- | --- |
| 0. Le Fou | (none) | +6 Char pts free. [v8.9] **Distribution:** Use the full 15-characteristic column for the knight's armor class from SPEC-02 Weighted Distribution Table. Normalize weights across all 15 characteristics (total = 43 per class). Each of the 6 points is rolled independently against this normalized distribution (probability per char = weight/43). Characteristic scores are still capped at parent Aspect score — if a characteristic is at cap, exclude it from the eligible pool and redistribute its weight among remaining candidates for that roll. If all 15 are capped, discard remaining points. |
| I. Le Bateleur | Dame | +3 in Dame chars |
| II. La Papesse | Masque | +3 in Masque chars |
| III. L'Impératrice | Machine | +3 in Machine chars |
| IV. L'Empereur | Dame | +3 in Dame chars |
| V. Le Pape | Machine | +3 in Machine chars |
| VI. L'Amoureux | Bête | +3 in Bête chars |
| VII. Le Chariot | Bête | +3 in Bête chars |
| VIII. La Justice | Machine | +3 in Machine chars |
| IX. L'Ermite | Machine | +3 in Machine chars |
| X. La Roue | Bête | +3 in Bête chars |
| XI. La Force | Chair | +3 in Chair chars |
| XII. Le Pendu | Chair | +3 in Chair chars |
| XIII. L'Arcane | Masque | +3 in Masque chars |
| XIV. La Tempérance | Chair | +3 in Chair chars |
| XV. Le Diable | Bête | +3 in Bête chars |
| XVI. La Maison-Dieu | (none) | +2 Aspect pts free (pick 2 random aspects from knight's 2 primary aspects per SPEC-02 Slot Allocation table, may repeat. Apply +1 to each selected aspect. NO Characteristic point distribution — this card raises ceilings only. Intentional and unique to La Maison-Dieu) |
| XVII. L'Étoile | Masque | +3 in Masque chars |
| XVIII. La Lune | Masque | +3 in Masque chars |
| XIX. Le Soleil | Dame | +3 in Dame chars |
| XX. Le Jugement | Dame | +3 in Dame chars |
| XXI. Le Monde | Chair | +3 in Chair chars |


### 24.5 Generation Integration
[v6] During SPEC-02 step 4:  
Draw 5 random cards from the 22-card Major Arcana pool (no duplicates within the same knight's draw).  
Apply Aspect +1 and +3 Characteristic points from all 5 drawn cards.  
Auto-select 2 advantages and 1 disadvantage from the 5 drawn cards ([v8] random selection for demo — pick 2 advantages and 1 disadvantage at random from the drawn cards. Player choice if manual generation is enabled).  
[v8.9] **Amnésique post-processing:** If the selected disadvantage is Amnésique (XVI. La Maison-Dieu), immediately after selection: pick one of the 2 active advantages at random, replace it with a random advantage from the full 22-card advantage table (§24.2). Replacement must differ from the remaining advantage (redraw up to 10 times if duplicate, then keep as-is).  
Unused advantages/disadvantages from the remaining 2 cards are discarded (stat bonuses from all 5 still apply).  
Cross-knight uniqueness: NOT enforced. Multiple knights may draw the same card. Each knight draws independently from the full 22-card pool.  

### Acceptance Criteria

Trop Prudent: initiative = 2D6 + Masque bonus. Not 3D6.  
[v8.9] Solitaire: K1 attacks E1 (Defense 6). K2 (Solitaire) assists. Threshold becomes 6+1=7. K1 needs >7 to hit (not >6). K2's successes still contribute to K1's total.  
[v8.9] Solitaire: K1 attacks E1 (Reaction 3). K2 (Solitaire) and K3 (Solitaire) both assist. Threshold becomes 3+2=5. Two Solitaire assistants = +2.  

[v6] Generation: 5 cards drawn from 22-card pool. All 5 apply stat bonuses (+1 Aspect, +3 Char pts each = +5 Aspects, +15 Char pts total). 2 advantages + 1 disadvantage selected from the 5.  
[v8.9] Amnésique: Knight draws cards I, V, IX, XVI, XX. Selected advantages: Infatigable (I), Forteresse Spirituelle (V). Selected disadvantage: Amnésique (XVI). Post-processing: random pick = Infatigable. Replace with random from full table → rolls Chanceux (X). Final advantages: Chanceux + Forteresse Spirituelle. Knight no longer has Infatigable despite drawing card I (stat bonuses from I still apply).
