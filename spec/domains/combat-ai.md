# Combat AI — Domain Specification

**Status**: 🟢 Complete (Round 2: 14 additional decisions)
**Last interrogated**: 2026-02-17
**Last verified**: —
**Depends on**: [combat](combat.md), [traits-and-perks](traits-and-perks.md), [characters](characters.md)
**Depended on by**: [tournaments](tournaments.md)

---

## Overview

Combat AI governs what characters choose to do on each turn during automated combat. With 20–40+ Actions per character, multi-resource costs, tag-scoped bonuses, and complex board state, the system uses a **Utility AI** model: every available Action is scored against current game state via weighted Considerations, and the best-scoring Action is selected with controlled randomness. AI "intelligence" is a character stat (Judgment), not a system constant — low-Judgment characters make personality-driven, impulsive decisions while high-Judgment characters approach optimal play. High-Judgment characters also gain **lookahead** — the ability to project future game state and factor predictions into scoring.

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

**Function** — controls three parameters:

- **Tactical-vs-Personality blend** (`judgment_blend`): How much the character weighs objective effectiveness vs personality impulses. Range ~0.3 (low Judgment) to ~0.95 (high Judgment).
- **Selection sharpness**: How reliably the character picks the top-scoring Action vs making "suboptimal" choices. Range ~1 (low Judgment, flat distribution) to ~10 (high Judgment, heavily favors top score).
- **Lookahead depth**: How many ticks ahead the character can project deterministic game-state events. Range 1 tick (low Judgment) to 10 ticks (high Judgment). See [Judgment-Gated Lookahead](#judgment-gated-lookahead).

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
| `position_need` | Tactical | Range-gap + zone density. Am I at the wrong range for my strongest abilities? Also evaluates zone density — avoid overcrowded zones, prefer zones with allies for mutual support. Boosts Move Actions when repositioning would unlock better Actions or improve spatial positioning. |
| `status_value` | Tactical | Value of applying a status effect (debuff on enemy, buff on ally) or removing one (cleanse). Considers existing stacks, resistance, and strategic impact. |
| `anti_repetition` | Tactical | Small penalty (~0.85×) for repeating last turn's Action. Creates natural rotation and more varied, watchable combat. |
| `revival_priority` | Tactical | Value of reviving a Fallen ally. Uses forward-looking "remaining potential" model: evaluates the fallen ally's available Actions, resource pools, cooldowns, and remaining fight length. High-value allies with full cooldowns and resources in long fights score highest; depleted allies in nearly-won fights score lowest. |
| `stealth_awareness` | Tactical | Adjusts scoring when stealth enemies are present. Boosts AoE Actions when Sneaking enemies exist in targetable zones. Boosts Search for high-value stealth targets. Reduces single-target damage scores when the best available targets are Sneaking (can't be targeted). |
| `consumable_scarcity` | Tactical | Scarcity-aware scoring for limited-use consumable Actions. Penalizes usage unless the situation warrants it — scales with remaining uses (fewer left = higher bar), impact threshold (only score high when effect is impactful), and tournament awareness (conserve across rounds when `combat_context.is_tournament` and consumables don't replenish). |
| `future_state_value` | Tactical | Judgment-gated prediction Scorer. Projects deterministic future events (Effects Phase outcomes + Initiative timing) up to the character's lookahead depth (1–10 ticks based on Judgment). Scores Actions higher when they set up favorable future states and lower when future state is unfavorable regardless of action. Existing Scorers stay simple (current state only); this Scorer adds prediction as an independent signal. See [Judgment-Gated Lookahead](#judgment-gated-lookahead). |
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

A character can have multiple personality archetypes from multiple Core Traits. When multiple archetypes apply, **all Personality Scorers contribute equally** — they are multiplied together and normalized by geometric mean like any other Scorer track. A character with [Aggressive, Protective] has both `aggression_preference` and `protective_preference` active simultaneously. Contradictory archetypes create internally conflicted characters — sometimes one Scorer wins, sometimes the other, driven by situational context. This is a feature: an aggressive-but-protective warrior might charge offensively when allies are healthy, but pivot to shielding when allies are hurt.

A character with no personality-tagged Core Traits has a neutral Personality Score (all Personality Scorers return 0.5, producing a neutral geometric mean).

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

## Consumable AI

Consumable Actions (potions, bombs, scrolls) use the **same Gate/Scorer pipeline** as Perk Actions. Consumables are not special-cased — they go through Gate checks, Tactical/Personality scoring, blending, and selection like any other Action.

The key addition is the `consumable_scarcity` Scorer, which prevents the AI from burning limited-use items carelessly:

**Scarcity Model** — three factors:

1. **Remaining uses**: Fewer remaining uses = higher threshold for usage. A character with 1 potion left needs a more compelling reason to drink it than one with 5 potions.
2. **Impact threshold**: The consumable must be impactful in the current situation. A healing potion when at 90% HP scores low; the same potion at 20% HP scores high. Impact is evaluated by the consumable's primary effect (healing urgency for health potions, damage output for bombs, etc.).
3. **Tournament awareness**: When `combat_context.is_tournament` is true and consumables don't replenish between rounds, the Scorer further penalizes usage in early rounds, conserving for later when stakes are higher.

Player-configured **consumable triggers** (Phase 2+) map to Gate overrides on consumable Actions, working alongside `consumable_scarcity` to provide fine-grained control.

---

## Combat Context Flags

At fight start, the combat system passes an explicit **context flags** object to the AI system. Any Scorer can query these flags to adjust behavior based on fight circumstances.

```
combat_context: {
  is_tournament: bool,
  round: int,              // current round (1-indexed)
  total_rounds: int,       // total rounds in this event
  consumables_replenish: bool,  // do consumables reset between rounds?
  is_exhibition: bool,
  pve_tier: int | null,    // null if not PvE
  team_size: int,
  opponent_team_size: int,
  ...                      // extensible for future flags
}
```

**Usage examples**:
- `consumable_scarcity` checks `is_tournament`, `round`, `total_rounds`, and `consumables_replenish` to decide conservation strategy
- `resource_efficiency` can adjust early/late phase thresholds based on fight context
- Future Scorers can use `pve_tier` to adjust risk tolerance (low-tier PvE = more aggressive, high-tier = more conservative)

Context flags are defined by the combat system (see [combat.md](combat.md)), not by the AI spec. The AI spec only defines that Scorers can read them.

---

## Judgment-Gated Lookahead

High-Judgment characters gain the ability to **project future game state** and factor predictions into Action scoring. This is the third parameter controlled by Judgment, alongside blend and sharpness.

### Lookahead Depth

Lookahead depth scales with Judgment:

- **Low Judgment**: 1 tick lookahead (minimal prediction)
- **High Judgment**: Up to 10 ticks lookahead (substantial prediction)

With fights lasting 25–150 ticks, 10-tick lookahead represents ~7–40% of a fight — substantial but appropriate as a reward for high-Judgment investment.

### Lookahead Scope

The lookahead projects **deterministic events only** — things that will definitely happen regardless of anyone's decisions:

1. **Effects Phase outcomes**: DoT damage, regen ticks, buff/debuff expiry, stack decay. The AI can predict "this enemy will die to their Burning stacks in 3 ticks" or "my shield buff expires next tick."
2. **Initiative timing**: When characters will reach the Initiative ≥ 100 threshold and take their turns. The AI can predict "I'll act again before that enemy does" or "three enemies act before my healer gets a turn."

The lookahead does **NOT** include:
- **Threat estimation**: No guessing what enemies will do (their Action choices are unpredictable)
- **Probability projection**: No modeling of hit/miss/crit variance
- **Recursive planning**: No "if I do X, they'll do Y, then I'll do Z" chains

### Implementation: `future_state_value` Scorer

Lookahead is implemented as a separate `future_state_value` Scorer in the Tactical track. Existing Scorers remain simple — they evaluate current state only. The `future_state_value` Scorer adds prediction as an **independent signal** that composes with other Scorers through the standard geometric mean.

This Scorer:
- Simulates the Effects Phase and Initiative accumulation for N ticks ahead (N = lookahead depth)
- Evaluates the projected board state (ally HP, enemy HP, buff/debuff status, resource levels, who acts next)
- Returns a score based on how favorable the projected future is for the current Action choice

For characters with 1-tick lookahead, this Scorer provides minimal information (essentially just "will DoTs kill anyone before next turn?"). For characters with 10-tick lookahead, it provides rich strategic context.

---

## Team Coordination

Team AI uses **sequential evaluation with state updates**. Characters within a tick's Turns Phase evaluate and act in **Initiative order** (highest Initiative first), and the board state updates after each character acts. There is no explicit coordination logic — natural coordination emerges from characters reacting to current reality.

**How it works**:
1. Characters are ordered by Initiative (using the standard tie-breaking rules from [combat.md](combat.md))
2. The first character evaluates all Actions against current board state, selects and executes one
3. Board state updates (HP changes, status applied, position changed, etc.)
4. The next character evaluates against the **updated** board state
5. Repeat until all characters with Initiative ≥ 100 have acted

**Why sequential, not simultaneous**: Simultaneous evaluation would require characters to "guess" what allies will do, adding complexity without strategic depth. Sequential evaluation means later-acting characters naturally adapt to earlier characters' Actions — a healer sees the warrior already healed and switches to buffing, a damage dealer sees the priority target already dead and picks a new one.

**Emergent coordination patterns**:
- **Heal triage**: When two healers act in the same tick, the first heals the most wounded ally; the second heals the next most wounded (because the first target is now less wounded)
- **Focus fire**: When allies have already damaged a target, `target_vulnerability` and `damage_output` naturally favor finishing that target
- **AoE avoidance**: When allies move into a zone, `aoe_clustering` naturally adjusts to avoid friendly fire in the new configuration

---

## Content Authoring: Performance Guidelines

Soft guidelines for content authors designing Perk Actions with AI Considerations:

- **Prefer standard Scorers**: The standard Consideration library covers most use cases. Use `damage_output`, `resource_efficiency`, `target_vulnerability`, etc. before creating custom Considerations.
- **Keep custom Considerations simple**: When a standard Scorer doesn't fit, custom Considerations should evaluate a single factor (not replicate the complexity of multiple standard Scorers combined).
- **Optimize the engine, not the content**: Performance optimization happens in the AI evaluation engine — batching, caching, early termination. Content authors should focus on correct behavior, not computational efficiency.
- **No hard limits**: There are no enforced limits on Scorer count per Action or custom Consideration complexity. These guidelines are for content quality, not system constraints.

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
| Stealth/detection trade-offs | `target_exists_gate` respects Sneaking visibility. `stealth_awareness` Scorer boosts AoE when stealth enemies exist, boosts Search for high-value targets, reduces single-target scores when best targets are stealthed. AI naturally adapts tactics to stealth presence. |
| Revival priority | `revival_priority` Scorer uses forward-looking "remaining potential" model. Evaluates fallen ally's available Actions, resource pools, cooldowns, and remaining fight length. High-value allies with full resources in long fights get revived; depleted allies in nearly-won fights don't waste Actions on revival. |
| No personality archetypes | Characters without personality-tagged Core Traits get neutral Personality Scores (0.5). Judgment blend still applies — their behavior is purely tactical, modulated by selection sharpness. |
| Consumable conservation | `consumable_scarcity` Scorer penalizes consumable usage unless impactful. Scales with remaining uses, impact threshold, and tournament awareness (conserve across rounds). Player-configured triggers (Phase 2+) can override via Gate conditions. |
| Stealth AoE preference | When multiple enemies are Sneaking, `stealth_awareness` naturally boosts AoE Actions (which hit Sneaking characters) and deprioritizes single-target damage (which can't target them). No special-case logic — scoring handles it. |
| Personality conflict | Characters with contradictory archetypes (e.g., [Aggressive, Protective]) have all Personality Scorers active simultaneously. Contradictions create internally conflicted characters whose behavior shifts based on situational context — sometimes aggression wins, sometimes protection. This is a feature, not a bug. |
| Team coordination timing | Characters in the same tick act sequentially in Initiative order, with board state updating between each. No explicit coordination — later characters naturally react to earlier characters' Actions. Emergent coordination (heal triage, focus fire) happens automatically. |

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
3. **Lookahead depth curve**: Exact mapping from Judgment stat value to the 1–10 tick lookahead range. Likely linear or logistic with a floor of 1.
4. **Response curve parameters**: Default curve shapes and parameters for each standard Consideration. These are tuning values refined through playtesting.
5. **Personality archetype weights**: Default influence strength per archetype. How strongly does "Aggressive" bias toward offensive Actions?
6. **Anti-repetition penalty**: Exact multiplier (~0.85× is the starting point). May vary by Action type (spamming the same AoE might be worse than repeating a basic Attack).
7. **AoE ally penalty weight**: Exact multiplier (1.5–2.0× range) for ally presence in AoE zones.
8. **Fight phase detection**: How does `resource_efficiency` determine "early" vs "late" fight? Tick count thresholds, team HP ratios, or combination?
9. **Consumable scarcity thresholds**: Exact scoring curves for the `consumable_scarcity` Scorer — how remaining uses, impact assessment, and tournament round factor into the penalty.
10. **Stealth awareness parameters**: Scoring weights for `stealth_awareness` — how much does AoE get boosted vs. single-target deprioritized when stealth enemies exist? At what threshold does Search become worthwhile?
11. **Zone density weights**: How strongly does zone density (ally/enemy count per zone) factor into `position_need` relative to range-gap evaluation?
12. **Future state value weights**: How much influence does the `future_state_value` Scorer have relative to other Tactical Scorers? Should its weight scale with Judgment (more trusted at high Judgment)?

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [combat](combat.md) | Judgment is a new derived stat (Awareness + Willpower + Intellect blend). Combat log must record AI scoring data (top 3 Actions per turn with scores). AI evaluation happens during the Turns Phase of each tick. Characters within a tick act sequentially in Initiative order with state updates between each. Combat system passes context flags to AI at fight start. **Fight length revised to 25–150 ticks** (range reflects PvE fodder to championship finals). |
| [traits-and-perks](traits-and-perks.md) | Perk Action data model must include an `ai` block (gates, tactical_scorers, personality_scorers). Core Traits may include personality archetype tags. Default Actions (Combatant system Trait) have embedded Considerations. |
| [characters](characters.md) | Judgment is a new derived stat feeding the AI system. Awareness, Willpower, and Intellect gain additional importance as contributors to AI quality. |
| [consumables](consumables.md) | Consumable Actions need AI Considerations and use the standard Gate/Scorer pipeline. `consumable_scarcity` Scorer manages limited-use conservation. Player-configured consumable triggers map to Gate overrides (e.g., "use healing potion below 30% HP" adds a custom Gate). Tournament context flags affect consumable conservation strategy. |
| [tournaments](tournaments.md) | NPC teams use the same AI system — difficulty comes from stat distribution, not AI code. AI configuration quality affects tournament performance (Phase 2+). |
| [equipment](equipment.md) | Equipment affixes can modify Judgment (boosting or reducing AI quality). Equipment with AI-relevant effects (stealth, AoE, revival) benefits from appropriate Considerations. |
| [data-model](../architecture/data-model.md) | Must support: Consideration schema (type + curve parameters), per-Action AI block (gates + tactical_scorers + personality_scorers), per-character AI configuration (weight overrides), personality archetype tags on Core Traits. |

---

_Last updated: 2026-02-17 — Round 2 interrogation: 14 additional decisions. Added Consumable AI (consumable_scarcity Scorer, standard pipeline, tournament-aware conservation), Combat Context Flags (explicit context passed to AI), Judgment-Gated Lookahead (1–10 ticks prediction depth, future_state_value Scorer, Effects Phase + Initiative timing projection), stealth_awareness Scorer, team coordination (sequential evaluation with state updates), personality archetype mixing (all contribute equally, contradictory = internally conflicted), fight length revised to 25–150 ticks, revival_priority updated to forward-looking remaining potential model, position_need updated with zone density, content authoring performance guidelines. Standard library expanded to 4 Gates + 14 Tactical Scorers + 5 Personality Scorers. Previous: Full rewrite replacing priority-list model with Utility AI system._
