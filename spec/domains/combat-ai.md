# Combat AI — Domain Specification

**Status**: 🟢 Complete
**Last interrogated**: 2026-02-17
**Last verified**: —
**Depends on**: [combat](combat.md), [traits-and-perks](traits-and-perks.md), [characters](characters.md)
**Depended on by**: [tournaments](tournaments.md)

---

## Overview

Combat AI governs what characters choose to do on each turn during automated combat. With 20–40+ Actions per character, multi-resource costs, tag-scoped bonuses, and complex board state, the system uses a **Utility AI** model: every available Action is scored against current game state via weighted Considerations, and the best-scoring Action is selected with controlled randomness. AI "intelligence" is a character stat (Judgment), not a system constant — low-Judgment characters make personality-driven, impulsive decisions while high-Judgment characters approach optimal play.

AI configuration remains a player-facing build optimization surface. Players influence behavior through weight overrides on the scoring system, not by writing priority lists.

---

## Core Concepts

### Utility AI Model

Each turn, the AI evaluates every Action available to the character:

1. **Gate check**: Binary prerequisites (resource cost, cooldown, range, valid targets). Any Gate returning 0 vetoes the Action.
2. **Score calculation**: Remaining Actions are scored on two tracks — Tactical (objective effectiveness) and Personality (character temperament).
3. **Blend**: Tactical and Personality scores are blended using the character's Judgment stat.
4. **Selection**: A weighted random pick from all scored Actions, with Judgment-controlled sharpness determining how reliably the top-scoring Action is chosen.

This replaces the priority-list model (try A, then B, then C) which does not scale to large Action pools with complex interactions.

### Judgment — Derived Stat

Judgment is a derived combat stat that controls AI decision quality. See [combat.md](combat.md) for the canonical formula.

**Function** — controls two parameters:

- **Tactical-vs-Personality blend** (`judgment_blend`): How much the character weighs objective effectiveness vs personality impulses. Range ~0.3 (low Judgment) to ~0.95 (high Judgment).
- **Selection sharpness**: How reliably the character picks the top-scoring Action vs making "suboptimal" choices. Range ~1 (low Judgment, flat distribution) to ~10 (high Judgment, heavily favors top score).

**Modifiers beyond the base formula:**
- Character Star Rating (higher stars = modest baseline boost)
- Specific Perks and equipment affixes
- Status effects (confusion, fear could lower it temporarily)

High Judgment = near-optimal play. Low Judgment = personality-driven, impulsive, sometimes brilliant, sometimes foolish. A Berserker with low Judgment *will* sometimes charge in and die — that's a deliberate design trade-off for taking that Trait.

### AI as Build Tool

AI configuration is a core part of character optimization. A well-built character with bad AI weight overrides will underperform. AI config is the "strategy" layer that sits on top of the "build" layer (Traits, Perks, Equipment). Unlike the old priority-list model, players tune weight multipliers on a scoring system rather than manually ordering Actions.

---

## Consideration System

Each Action defines a list of **Considerations** — functions that evaluate game state and return a score. Considerations are the atomic unit of AI reasoning.

### Two Types

**Gates** (binary: 0.0 or 1.0):
Hard prerequisites. Any Gate returning 0.0 vetoes the Action entirely. Gates are pass/fail — no partial credit.

**Scorers** (continuous: 0.01–1.0, never exactly 0.0):
Preference signals that evaluate how good this Action is in the current context. The floor of 0.01 ensures that no single Scorer can completely eliminate an Action that passes all Gates — only Gates can veto.

### Score Composition

Scorers within a track (Tactical or Personality) are **multiplied together**, then normalized via **geometric mean** (Nth root, where N = number of Scorers in that track).

**Why geometric mean?**
- Removes structural bias: Actions with different Scorer counts are compared fairly (a 3-Scorer Action and a 6-Scorer Action produce scores on the same scale).
- Preserves the property that a single very-low Scorer drags the overall score down — one terrible factor meaningfully penalizes the Action.
- Product-then-normalize is computationally simple.

**Example**: An Action with 3 Tactical Scorers scoring 0.8, 0.9, 0.6:
- Product = 0.432
- Geometric mean = 0.432^(1/3) ≈ 0.756

### Standard Consideration Library

Reusable Consideration types that Perk Actions reference by name. Content authors select from this library when defining Action AI behavior. Custom Considerations can be added for unique Actions.

#### Standard Gates

| Gate | Evaluates |
|------|-----------|
| `resource_gate` | Can the character afford all resource costs? For multi-resource Actions, checks each pool independently — all must pass. |
| `cooldown_gate` | Is the Action off cooldown? |
| `range_gate` | Is at least one valid target within the Action's range band? |
| `target_exists_gate` | Does at least one valid target exist (respecting target tags, Sneaking visibility, and team alignment)? |

#### Standard Scorers

| Scorer | Track | Evaluates |
|--------|-------|-----------|
| `resource_efficiency` | Tactical | Cost relative to remaining pool, fight phase (early = conserve, late = spend), and regen rate. For multi-resource costs, evaluates each pool independently and takes the **minimum** score (bottleneck pool dominates). |
| `damage_output` | Tactical | Estimated damage vs target defenses (Soak, Defense Value, damage type resistances). Higher when the Action deals more effective damage. |
| `healing_urgency` | Tactical | How badly do allies (or self) need healing? Scales with missing HP across valid heal targets. Near-zero when all allies are healthy. |
| `aoe_clustering` | Tactical | Net value of targets in AoE zone. Σ(enemy value) − Σ(ally penalty × 1.5–2.0× weight). Negative-net zones score ~0.05 (naturally deprioritized but not vetoed). |
| `target_vulnerability` | Tactical | Target's vulnerability to this Action's damage type and tags. Higher when the target has low resistance or relevant debuffs. |
| `action_speed_efficiency` | Tactical | Effect value normalized by Initiative cost. Fast Actions score higher per unit of time spent. |
| `self_hp_critical` | Tactical | Defensive bonus that scales as own HP drops. Boosts defensive/healing Actions when the character is in danger. |
| `position_need` | Tactical | Am I at the wrong range for my strongest abilities? Boosts Move Actions when repositioning would unlock better Actions. |
| `status_value` | Tactical | Value of applying a status effect (debuff on enemy, buff on ally) or removing one (cleanse). Considers existing stacks, resistance, and strategic impact. |
| `anti_repetition` | Tactical | Small penalty (~0.85×) for repeating last turn's Action. Creates natural rotation and more varied, watchable combat. |
| `revival_priority` | Tactical | Value of reviving a Fallen ally. Considers the ally's combat value, team HP state, and fight phase. |
| `aggression_preference` | Personality | Boosts offensive/damage Actions. Derived from aggressive personality archetypes. |
| `caution_preference` | Personality | Boosts defensive/positioning Actions. Derived from cautious personality archetypes. |
| `protective_preference` | Personality | Boosts ally-supporting Actions (heals, buffs, shields). Derived from protective personality archetypes. |
| `vindictive_preference` | Personality | Boosts Actions targeting whoever last hurt this character or their allies. Derived from vindictive personality archetypes. |
| `showoff_preference` | Personality | Boosts flashy, high-impact, high-cost Actions. Derived from showoff personality archetypes. |

New Consideration types can be added as content without schema changes — the library is extensible.

### Response Curves

Scorers use **response curves** to map raw game-state values to the 0.01–1.0 output range. Common curve shapes:

- **Linear**: Direct proportional mapping (e.g., HP percentage → score)
- **Logistic (S-curve)**: Soft threshold — low sensitivity at extremes, high sensitivity in the middle
- **Exponential**: Accelerating urgency (e.g., healing urgency spikes as HP drops below 25%)
- **Step**: Binary-ish with a soft transition zone

Each Scorer definition includes curve parameters (thresholds, slopes, inflection points) as part of its data. These are tuning values, not hardcoded.

---

## Two-Track Scoring: Tactical + Personality

Each Action produces two independent scores before blending:

### Tactical Score

Product of all Tactical Scorers, normalized by geometric mean. Represents objective combat effectiveness — "how good is this Action right now?"

All Actions have Tactical Scorers. Default Actions (Attack, Defend, Move, Search) have Tactical Scorers defined on the Combatant system Trait's Perk.

### Personality Score

Product of all Personality Scorers, normalized by geometric mean. Represents how well this Action fits the character's temperament — "does this feel like something this character would do?"

**Personality archetypes** are derived from Core Traits. Each Core Trait that defines a personality can tag itself with one or more personality archetypes:

| Archetype | Bias | Example Core Trait |
|-----------|------|--------------------|
| **Aggressive** | Offensive, damage-dealing Actions score higher | Berserker's Fury |
| **Cautious** | Defensive, positioning, evasive Actions score higher | Vigilant Mind |
| **Protective** | Ally-supporting Actions (heals, buffs, shields) score higher | Guardian's Oath |
| **Vindictive** | Prioritize whoever last hurt self or allies | Vengeful Spirit |
| **Showoff** | Flashy, high-impact, expensive Actions preferred | Born Performer |

A character can have multiple personality archetypes from multiple Core Traits. When multiple archetypes apply, their corresponding Personality Scorers all contribute to the Personality Score. A character with no personality-tagged Core Traits has a neutral Personality Score (all Personality Scorers return 0.5, producing a neutral geometric mean).

**Actions without Personality Scorers**: If an Action has no Personality Scorers defined, its Personality Score defaults to 0.5 (neutral — neither boosted nor penalized by personality).

### Blending

The Judgment stat controls how much weight is given to each track:

```
judgment_blend = f(Judgment stat)  // range ~0.3 to ~0.95
final_score = judgment_blend × tactical_score + (1 - judgment_blend) × personality_score
```

- At **minimum Judgment** (~0.3 blend): 30% tactical + 70% personality. The character acts on instinct and temperament. A Berserker charges; a Coward retreats.
- At **maximum Judgment** (~0.95 blend): 95% tactical + 5% personality. The character makes near-optimal decisions with a subtle personality flavor.
- At **typical Judgment** (~0.6 blend): 60% tactical + 40% personality. Competent but characterful.

The 30% minimum tactical weight prevents truly suicidal decisions in most cases. However, a character with a strong aggressive archetype and low Judgment *can* make fatal mistakes — that's the design intent and the trade-off for taking that Trait combination.

---

## Selection Mechanism

After scoring all Actions, selection uses **weighted random with Judgment-controlled sharpness**:

```
selection_weight(action) = final_score ^ (1 + sharpness)
```

Where `sharpness` is derived from Judgment (range ~1 to ~10):

- **Low sharpness (~1)**: Score differences are modest. A 0.8 vs a 0.6 has only a ~1.8:1 weight ratio. Frequent "upsets" — lower-scored Actions get picked regularly.
- **High sharpness (~10)**: Score differences are extreme. A 0.8 vs a 0.6 has a ~180:1 weight ratio. The top-scoring Action is picked almost every time.

This creates **characterful variance**: low-Judgment characters are unpredictable and occasionally brilliant or foolish; high-Judgment characters are consistent and reliable.

### Two-Phase Evaluation

Each turn resolves in two phases:

**Phase 1 — Action Selection:**
Score all available Actions against general board state. Gates filter first, then Scorers evaluate, blend, and select one Action via weighted random.

**Phase 2 — Target Selection:**
For the selected Action, score all valid targets (or zones for AoE) using the same Gate/Scorer/weighted-random pipeline. Target scoring uses a subset of Considerations relevant to targeting:

- Single-target Actions: Score each valid target (damage_output, target_vulnerability, vindictive_preference, etc.)
- AoE-Zone Actions: Score each valid zone. Net value = Σ(enemy value) − Σ(ally penalty × 1.5–2.0× weight). Negative-net zones score ~0.05.
- Self-target Actions: No target selection needed.
- Random-target Actions: No target selection needed (system picks randomly).

Target selection uses the same sharpness parameter as Action selection.

### Fallback Chain

If all Actions are vetoed by Gates (should essentially never happen with well-defined content):

1. **Defend** (no resource cost, no cooldown — always passes Gates)
2. **Move** toward nearest ally (no resource cost, no cooldown)
3. **Attack** nearest enemy (no resource cost, no cooldown)

The Combatant system Trait's default Actions (Attack, Defend, Move, Search) have no resource cost and no cooldown, so at least one always passes all Gates. The fallback chain exists as a safety net, not a regular code path.

---

## AI Considerations Embedded in Perk Definitions

Each Perk Action definition includes its AI Considerations as part of the data model. When a content author designs a new Perk Action, they also define which Considerations the AI uses to evaluate that Action.

### Data Structure

```
Action: <action_name>
  target: [<target_tags>]
  cost: { <resource>: <amount>, ... }
  range: <Short|Medium|Long>
  cooldown: <turns>
  action_speed: <modifier>
  effects: [<effect_component_list>]
  ai:
    gates: [<gate_type>, ...]
    tactical_scorers:
      - <scorer_type>: { <curve_params> }
      - <scorer_type>: { <curve_params> }
    personality_scorers:
      - <scorer_type>: { weight: <float> }
```

### Example: Flame Burst (AoE Fire Spell)

```
Action: Flame Burst
  target: [Enemy, AoE-Zone]
  cost: { Mana: 30 }
  range: Medium
  cooldown: 3
  action_speed: -20
  effects: [Damage(Fire, 40), ApplyStatus(Burning, 3, 2)]
  ai:
    gates: [resource_gate, cooldown_gate, range_gate, target_exists_gate]
    tactical_scorers:
      - aoe_clustering: { min_targets: 2, ideal_targets: 4 }
      - target_vulnerability: { tag: Fire }
      - resource_efficiency: { resource: Mana }
      - action_speed_efficiency: {}
    personality_scorers:
      - aggression_preference: { weight: 0.6 }
      - showoff_preference: { weight: 0.4 }
```

### Example: Healing Word (Single-Target Heal)

```
Action: Healing Word
  target: [Ally, Single]
  cost: { Faith: 20 }
  range: Medium
  cooldown: 0
  action_speed: +10
  effects: [Heal(35)]
  ai:
    gates: [resource_gate, cooldown_gate, range_gate, target_exists_gate]
    tactical_scorers:
      - healing_urgency: { critical_threshold: 0.3 }
      - resource_efficiency: { resource: Faith }
      - action_speed_efficiency: {}
    personality_scorers:
      - protective_preference: { weight: 0.8 }
```

### Default Action Considerations

Default Actions from the Combatant system Trait have embedded Considerations designed to make them viable gap-fillers:

**Attack:**
- Gates: `range_gate`, `target_exists_gate`
- Tactical: `damage_output`, `target_vulnerability`, `anti_repetition`
- No personality scorers (neutral)

**Defend:**
- Gates: none (always valid)
- Tactical: `self_hp_critical`, `resource_efficiency` (Stamina recovery value)
- No personality scorers (neutral)

**Move:**
- Gates: none (always valid — even "stay in zone" is a valid move outcome)
- Tactical: `position_need`
- No personality scorers (neutral)

**Search:**
- Gates: `target_exists_gate` (requires at least one Sneaking enemy)
- Tactical: `status_value` (value of detecting hidden enemies)
- No personality scorers (neutral)

Default Actions naturally lose to Perk Actions as characters scale, because Perk Actions benefit from Trait × Perk level multipliers (up to ×4.0) while the Combatant system Trait is fixed at level 1 (×1.0).

---

## Player Configuration

Player-facing AI configuration maps to **weight overrides** on the Consideration scoring system. Players don't write priority lists — they tune multipliers.

### Configuration Axes

**Target Priority**: Weight multiplier on target-scoring Considerations.
- Closest enemy (default)
- Lowest HP enemy
- Highest threat enemy (most damage dealt this fight)
- Specific role targeting (e.g., "prioritize healers")

**Range Preference**: Weight multiplier on `position_need` Scorer.
- Prefer Short range (melee fighters)
- Prefer Medium range (polearm, mid-range casters)
- Prefer Long range (archers, long-range casters)

**Action Preferences**: Per-Action weight multipliers.
- Boost or suppress specific Actions (e.g., "always prefer Flame Burst" → 1.5× weight; "avoid basic Attack" → 0.5× weight)
- Applies as a multiplier on the final score before selection

**Consumable Triggers**: Gate conditions for consumable-type Actions.
- Use healing potion below X% HP
- Use buff potion on first turn
- Use bomb when 3+ enemies in zone
- Custom trigger conditions as Gate overrides

### Phased Rollout

| Phase | AI Capability |
|-------|--------------|
| Phase 1 (MVP 0) | No player configuration. Utility AI runs with default Consideration weights. Characters behave based on their Judgment stat and personality archetypes. |
| Phase 2 (MVP 1) | Basic config: target priority + Action preference weight multipliers. Enough to influence fights meaningfully without overwhelming new players. |
| Phase 4 | Full customization: range preferences, per-range-bracket overrides, consumable triggers, advanced weight tuning. |

### Configuration Persistence

AI configuration is **per-character, persistent**. Settings carry across fights. Players can modify between any combat (not mid-fight). Default configuration on newly recruited characters uses standard weights (no overrides).

---

## NPC Team AI

NPC teams (tournament backfill, PvE encounters) use the **same Utility AI system** as player teams. The difference is in Judgment stat values and personality archetype distribution:

- **Low-tier NPCs** (PvE fodder, early tournament backfill): Low Judgment (impulsive, personality-driven). Star Rating and attribute generation naturally produce lower Judgment. No player config overrides — default weights only.
- **High-tier NPCs** (championship contenders, boss encounters): High Judgment (near-optimal play). Better attributes, more Perks, and potentially Judgment-boosting Perks.
- **Named NPCs** (persistent characters): Same as player characters — their Judgment and personality come from their stats and Traits.

No separate "smart NPC AI" or "dumb NPC AI" logic exists. AI quality is entirely driven by the Judgment stat and available Actions/Perks. This ensures NPCs feel like real characters, not scripted opponents.

---

## AI Preview and Debugging

Players cannot preview or simulate AI behavior before a match ("dry run" mode is not supported in any phase). However, the Utility AI system provides **post-combat transparency**:

- **Combat log**: Records each turn's Action selection, showing the top 3 scored Actions and their final scores. Players can review why their character chose what they did.
- **Phase 2+**: Configuration screen shows estimated Action priority rankings based on typical board states, helping players tune weights without needing a simulation.

The combat log replaces simulation needs — players learn their character's tendencies by watching fights and reviewing logs, then adjust configuration accordingly. This fits the "watch auto-battles, optimize between fights" core loop.

---

## Edge Case Handling

| Edge Case | Solution |
|-----------|----------|
| All Actions vetoed by Gates | Default Actions (Attack, Defend, Move) have no resource cost and no cooldown — always pass Gates. Fallback chain ensures a turn always happens. |
| Score normalization across different Scorer counts | Geometric mean (Nth root) normalizes products fairly regardless of Scorer count. |
| Resource conservation across fight phases | `resource_efficiency` Scorer factors: cost ÷ remaining pool, fight phase (early = conserve, late = spend), regen rate. Characters naturally conserve early and spend freely as the fight progresses. |
| Multi-resource Action costs | `resource_gate` checks each pool independently — all must pass. `resource_efficiency` Scorer evaluates each pool independently, takes minimum score (bottleneck pool dominates). |
| AoE friendly fire | `aoe_clustering` Scorer: net value = Σ(enemy value) − Σ(ally penalty × 1.5–2.0× weight). Negative-net zones score ~0.05, naturally deprioritized but not hard-vetoed. |
| Action repetition | `anti_repetition` Scorer: ~0.85× for Actions used last turn. Creates natural rotation without hard-blocking repeat usage when it's genuinely the best option. |
| Personality vs survival | Even at lowest Judgment blend (30% tactical), survival-critical Actions score high enough tactically to usually win. Content authors can add `self_hp_critical` to defensive Actions for extra safety. Berserkers dying to aggression is a feature, not a bug. |
| Stealth/detection trade-offs | `target_exists_gate` respects Sneaking visibility. Search Action's `status_value` Scorer evaluates the value of detecting hidden enemies. AI naturally searches when enemy stealth is impactful. |
| Revival priority | `revival_priority` Scorer evaluates fallen ally value, team state, fight phase. High-value allies in close fights get revived; low-value allies in won fights don't waste Actions on revival. |
| No personality archetypes | Characters without personality-tagged Core Traits get neutral Personality Scores (0.5). Judgment blend still applies — their behavior is purely tactical, modulated by selection sharpness. |

---

## Decisions

### Utility AI Model (Not Priority Lists)

- **Decision**: AI uses a Utility AI model — score all Actions via weighted Considerations, select via weighted random — rather than ordered priority lists.
- **Rationale**: Priority lists don't scale to 20–40+ Actions with multi-resource costs, tag-scoped bonuses, and complex board state. Utility AI evaluates all options simultaneously, naturally handling large Action pools. Weighted random with Judgment-controlled sharpness produces characterful, watchable behavior.
- **Implications**: Every Perk Action must include AI Consideration definitions. Content authors define which Considerations apply and with what parameters. The standard library covers common cases; custom Considerations handle unique Actions.
- **Alternatives considered**: Priority lists (original model — rejected due to scaling issues with large Action pools), behavior trees (rejected — too complex for data-driven content, hard for players to understand), GOAP (rejected — overkill for turn-based single-action decisions).

### Judgment as AI Quality Stat

- **Decision**: AI decision quality is controlled by the Judgment derived stat (from Awareness + Willpower + Intellect), not a system constant.
- **Rationale**: Makes AI quality a build investment. Characters built for smart play (high Awareness/Willpower/Intellect) make better decisions. Creates meaningful trade-offs — a pure Might/Endurance bruiser hits hard but makes suboptimal decisions. Keeps AI quality on the same progression curve as other combat stats.
- **Implications**: Characters spec needs Judgment in the derived stat table. NPC AI quality is naturally driven by their attributes. Balance must ensure low-Judgment characters are still fun to watch, not frustratingly stupid.

### Two-Track Scoring (Tactical + Personality)

- **Decision**: Actions produce separate Tactical and Personality scores, blended by Judgment.
- **Rationale**: Separating the tracks makes personality an explicit, tunable system rather than an emergent side effect. Content authors can design personality archetypes that produce specific behavioral patterns. The Judgment-controlled blend creates a clean spectrum from "impulsive and characterful" to "optimal and clinical."
- **Implications**: Core Traits that represent personality must be tagged with personality archetypes. Characters with no personality archetypes default to neutral (purely tactical behavior, modulated by sharpness).

### Personality Overrides as Soft Bias (Not Hard Override)

- **Decision**: Personality modifiers from Core Traits are soft biases via the Personality Score track, not hard overrides that ignore player configuration.
- **Rationale**: Hard overrides would frustrate players who invest in AI configuration only to have it ignored. Soft bias through the Personality Score creates meaningful personality flavor while respecting player agency. The Judgment stat controls the strength of the personality influence — players who want more predictable behavior can invest in Judgment-boosting Perks.
- **Implications**: No Trait can force the AI to take a specific Action regardless of game state. Even a Berserker's aggressive bias is filtered through scoring — it makes offensive Actions more likely, not mandatory. Extreme personality effects come from low Judgment + strong archetype, not from hard override mechanics.

### Considerations Embedded in Perk Definitions

- **Decision**: AI Considerations are defined as part of each Perk Action's data model, not in a separate configuration layer.
- **Rationale**: Keeps AI behavior co-located with ability definitions. Content authors designing a new Perk Action define its AI behavior at the same time, ensuring consistency. No separate "AI configuration database" to maintain. The standard Consideration library handles most cases; content authors compose from reusable types.
- **Implications**: Perk Action data model in [traits-and-perks](traits-and-perks.md) must include an `ai` block. Data model spec must support the Consideration schema (type + parameters).

### Player Config as Weight Overrides

- **Decision**: Player AI configuration maps to weight multipliers on the scoring system, not to priority list ordering.
- **Rationale**: Weight overrides compose naturally with the Utility AI system. Players express preferences ("prefer this Action," "target healers first") without needing to manually order 20–40+ Actions. The system handles interactions between preferences automatically.
- **Implications**: UI must present configuration as intuitive preferences, not raw weight numbers. Phase 1 ships without configuration (default weights only). Phase 2 adds basic preferences. Phase 4 adds full weight tuning.

### Same AI System for NPCs and Players

- **Decision**: NPC teams use the identical Utility AI system as player teams. AI quality differences come from Judgment stat values, not separate code paths.
- **Rationale**: One system to maintain and balance. NPCs feel like real characters with genuine decision-making. Difficulty scaling is natural — low-stat NPCs make worse decisions because their Judgment is low, not because they're running "dumb AI" code.
- **Implications**: NPC generation must produce appropriate Judgment stat ranges for the intended difficulty. No "cheating" NPC AI that sees hidden information or bypasses the scoring system.

### Geometric Mean for Score Normalization

- **Decision**: Scorer products are normalized via geometric mean (Nth root where N = number of Scorers).
- **Rationale**: Removes structural bias from different Scorer counts — Actions are compared fairly regardless of how many Scorers they have. Simple to compute. Preserves the property that one very-low Scorer meaningfully drags the score down.
- **Implications**: Content authors should understand that adding more Scorers to an Action doesn't inherently boost or penalize it. Each Scorer's scale (0.01–1.0) is meaningful relative to other Scorers of the same type.
- **Alternatives considered**: Raw product (rejected — biased against Actions with more Scorers), arithmetic mean (rejected — doesn't penalize low individual scores enough), weighted sum (rejected — loses the multiplicative property where one bad factor matters).

### No AI Simulation/Preview

- **Decision**: No "dry run" or simulation mode for previewing AI behavior before a match.
- **Rationale**: Simulation would require approximating opponent behavior and board state, producing unreliable results that might mislead players. The post-combat log provides more actionable information — players see actual decisions in real game states, then tune configuration based on observed behavior.
- **Implications**: Combat log must be detailed enough for players to understand AI decisions. Phase 2+ configuration screen shows estimated rankings based on typical states as a lighter alternative to full simulation.

---

## Open Questions

All design questions are resolved. Remaining items are tuning values:

1. **Judgment blend function**: Exact mapping from Judgment stat value to the 0.3–0.95 blend range. Likely logistic curve with tuning parameters.
2. **Selection sharpness function**: Exact mapping from Judgment stat value to the 1–10 sharpness range. Likely linear or logistic.
3. **Response curve parameters**: Default curve shapes and parameters for each standard Consideration. These are tuning values refined through playtesting.
4. **Personality archetype weights**: Default influence strength per archetype. How strongly does "Aggressive" bias toward offensive Actions?
5. **Anti-repetition penalty**: Exact multiplier (~0.85× is the starting point). May vary by Action type (spamming the same AoE might be worse than repeating a basic Attack).
6. **AoE ally penalty weight**: Exact multiplier (1.5–2.0× range) for ally presence in AoE zones.
7. **Fight phase detection**: How does `resource_efficiency` determine "early" vs "late" fight? Tick count thresholds, team HP ratios, or combination?

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [combat](combat.md) | Judgment is a new derived stat (Awareness + Willpower + Intellect blend). Combat log must record AI scoring data (top 3 Actions per turn with scores). AI evaluation happens during the Turns Phase of each tick. |
| [traits-and-perks](traits-and-perks.md) | Perk Action data model must include an `ai` block (gates, tactical_scorers, personality_scorers). Core Traits may include personality archetype tags. Default Actions (Combatant system Trait) have embedded Considerations. |
| [characters](characters.md) | Judgment is a new derived stat feeding the AI system. Awareness, Willpower, and Intellect gain additional importance as contributors to AI quality. |
| [consumables](consumables.md) | Consumable Actions need AI Considerations. Player-configured consumable triggers map to Gate overrides (e.g., "use healing potion below 30% HP" adds a custom Gate). |
| [tournaments](tournaments.md) | NPC teams use the same AI system — difficulty comes from stat distribution, not AI code. AI configuration quality affects tournament performance (Phase 2+). |
| [equipment](equipment.md) | Equipment affixes can modify Judgment (boosting or reducing AI quality). Equipment with AI-relevant effects (stealth, AoE, revival) benefits from appropriate Considerations. |
| [data-model](../architecture/data-model.md) | Must support: Consideration schema (type + curve parameters), per-Action AI block (gates + tactical_scorers + personality_scorers), per-character AI configuration (weight overrides), personality archetype tags on Core Traits. |

---

_Last updated: 2026-02-17 — Full rewrite: replaced priority-list model with Utility AI system. Defined Consideration architecture (Gates + Scorers), standard library (4 Gates, 16 Scorers), two-track scoring (Tactical + Personality), Judgment-controlled blending and selection sharpness, Perk-embedded AI definitions, player config as weight overrides, NPC AI parity, edge case handling. All 8 original open questions resolved._
