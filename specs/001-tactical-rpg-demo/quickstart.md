# Quickstart: Validating the Demo End-to-End

## Prerequisites
- Unity 6.3 LTS installed (Hub → Installs → Unity 6.3 LTS)
- Git clone of the DarkestKnight repository
- Branch: `001-tactical-rpg-demo`

## Project Setup
1. Open Unity Hub → Add → Browse to `/DarkestKnight/` root folder
2. Unity will import packages: `Unity.Mathematics`, `TextMeshPro`, `Input System`, `Test Framework`
3. Open `Assets/Scenes/Main.unity` (single scene architecture per SPEC-32)
4. Verify Console has no errors

## Running Tests (Priority Order)

### 1. RNG Determinism (SPEC-28)
```
Edit → Test Runner → EditMode → RNGDeterminismTests
```
Validates: Same seed → identical sequence. Cross-platform uint consistency.

### 2. Knight Generation (SPEC-02)
```
Edit → Test Runner → EditMode → KnightGenerationTests
```
Validates: 8 unique knights generated from seed. Correct archetypes, Tarot cards, derived values.

### 3. Core Combat Pipeline (SPEC-04/05/06)
```
Edit → Test Runner → EditMode → ComboRollTests, DamageResolverTests, ArmourLayerTests
```
Validates: All worked examples from TechSpec produce correct results.

### 4. Integration Scenarios (SPEC-15_IntegrationTests)
```
Edit → Test Runner → EditMode → IntegrationScenario1..5
```
- Scenario 1: Fold → Despair → Hémorragie Cascade
- Scenario 2: Ghost Alpha-Strike + Exploit + Dual-Wield
- Scenario 3: Barrage + Boss Phase Transition
- Scenario 4: Grenade Friendly Fire + Despair + Stabilization
- Scenario 5: Full Node Transition Attrition

### 5. Full Mission Smoke Test
```
Edit → Test Runner → PlayMode → FullMissionSmokeTest
```
Validates: Complete Camelot → Mission → Camelot loop.

## Manual Playthrough Validation

### Quick Combat Test
1. Press Play in Main.unity
2. Click "New Campaign" (generates 8 knights from random seed)
3. Select 4 knights at roster screen
4. Deploy to ranks (drag to slots or use defaults)
5. Play through combat: select knight → choose style → pick target → resolve
6. Verify: damage numbers match roll breakdown panel, PEs penalties apply, log entries appear

### Full Demo Loop
1. New Campaign → Camelot Hub
2. Select Squad (4 knights) → Deploy
3. Node 1: Combat vs Nocte Bande (verify Violence → Cohesion, Débordement per turn)
4. Node 2: Branch choice (nest or ruins) → Combat + Event
5. Node 3: Combat vs Faune + Bestians
6. Node 4: Narrative Event (verify PEs restoration)
7. Node 5: Boss (Ours Corrompu)
   - Phase 1: Normal combat, deplete PS to 0
   - Phase 2: Verify new abilities, Bande reinforcement, AI shift
8. Victory → Return to Camelot
9. Verify: PS/PA restored, PEs UNCHANGED, PG awarded
10. Visit Infirmary (if injuries), check recruitment (if deaths)
11. Deploy again for 2nd mission

### Critical Checks
- [ ] No `UnityEngine.Random` usage: `grep -r "UnityEngine.Random" Assets/Scripts/` returns 0
- [ ] Same seed produces identical combat: run with seed 12345 twice, compare combat logs
- [ ] Fold state disables abilities: PA → 0 → Guardian Suit → verify OD/styles/PE spending blocked
- [ ] Hémorragie carries between nodes: take bleeding knight to next node, verify countdown continues
- [ ] Permadeath permanent: dead knight never reappears in roster
- [ ] Squad defeat: all 4 incapacitated/dead → mission fails
- [ ] PEs penalty: verify penalty = max(0, 10 - currentPEs) applied to all rolls
- [ ] Boss Phase 2: PS = 0 → phase transition, new abilities, -1D6 PEs all knights

## Architecture Validation
- [ ] Single scene: no `SceneManager.LoadScene` calls in combat code
- [ ] State isolation: CombatSession destroyed after combat, no NRE in Hub
- [ ] UI decoupling: UI subscribes to EventBus events, sends commands to CombatManager
- [ ] Naming: French enums (AGRESSIF), English fields (isInAgony), French UI strings

## Build Targets
```bash
# PC Standalone (primary)
File → Build Settings → PC, Mac & Linux Standalone → Build

# Verify: launches, new campaign works, full loop completable
```
