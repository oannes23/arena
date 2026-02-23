# Foundation

## Core Concepts

### Numeric System

All combat values are **integers**. Fractional results always **round up** (ceiling). This applies to damage, healing, Soak reduction, Initiative gain, formula evaluation, status decay, and all other combat calculations.

- **Minimum 1 damage per component**: Every damage component that successfully hits deals at least 1 damage after all reductions. Chip damage always matters.
- **Minimum 1 stack**: Status effects always apply at least 1 stack even after Resistance reduction (see [Status Effect System](resolution.md#status-effect-system)).
- **Rounding direction**: Ceiling rounding favors attackers and status appliers — even tiny fractional damage rounds to 1. All formula evaluations produce integers.

### Battle Format

- **Scale**: 1v1 to 20v20. **2+ teams supported** — free-for-all, multi-faction, and asymmetric sizes (3v5, 1v10, etc.) are all valid configurations. "Enemy" = anyone not on your team.
- **Automation**: Pre-configured AI orders; combat executes automatically (idle/auto-battler). **Simulation-first**: the combat engine runs the entire fight to completion, then presents results (see [Combat Event Stream](systems.md#combat-event-stream--deterministic-simulation)).
- **Player interaction**: Configure builds and AI before combat; review results after (dramatized summary + detailed event logs).
- **Win condition**: Elimination with **per-team elimination order**. Last team standing = 1st place. Teams are ranked by elimination order (last eliminated = 2nd, second-to-last = 3rd, etc.). Teams eliminated on the same tick share the same placement rank (next rank skipped). **Mutual elimination** (all remaining teams eliminated on the same tick) results in a **draw** — all teams share 1st place, consistent with the same-tick shared rank rule. The Attrition Ramp (see below) ensures fights always converge to a conclusion.
- **Target match length**: 25–150 ticks. The range reflects fight variety: short PvE fodder encounters at the low end (~25 ticks), standard tournament matches in the middle (~50–80 ticks), and epic championship finals at the high end (~100–150 ticks). Controlled via the global Initiative multiplier and team composition.

### Spatial System — Zones and Ranges

**Ranges** (binary — an ability is in-range or not, no damage falloff):
- **Short**: Same zone (melee range)
- **Medium**: Adjacent zones
- **Long**: Non-adjacent zones

**Zone Maps**:
- **Default**: 5 zones (North, South, East, West, Center)
- **Default adjacency** (center + ring topology):
  - Center is adjacent to all four cardinal zones
  - Cardinal zones are adjacent to their ring neighbors: N↔E, E↔S, S↔W, W↔N
  - Opposite cardinals (N↔S, E↔W) are **not** adjacent = **Long range**
  - This creates meaningful Long range on the default map — ranged characters can target across opposite sides without moving
- **Variants**: Custom layouts scaled to combatant count (Phase 4)
- **Gimmicks**: Traps, timed hazards, movement modifiers, obstacles (Phase 4)
- **Zone Capacity**: No strict limits by default; can be specified per-zone per-battlefield

**Starting Positions**:
- **Event-type defined**: Each event template specifies a list of allowed starting zones per team (e.g., Team A: [North], Team B: [South]). Characters are distributed within their team's allowed zones by the system.
- Tournaments might use fixed opposing zones; PvE encounters could place enemies in ambush positions; free-for-all events might spread teams across all zones.
- Starting position configuration is part of the event template data, alongside other per-event settings (Context Flags, attrition ramp parameters, etc.).

**Movement**:
- Basic Move action costs the character's full turn
- **Forced movement**: Unrestricted placement by default — no resistance roll or check. The target is moved to the specified zone with no saving throw. Fires `OnForcedMove` Triggers on the affected character. Resistance Perks can reduce or prevent forced movement (e.g., "Immovable Stance" Perk might grant 50% chance to resist forced movement, or reduce push distance by 1 zone). Without such Perks, forced movement is guaranteed.

**Design Intent**: Making Move cost a full turn creates meaningful positioning. Ranged characters have a genuine advantage — melee fighters pay a real cost to close distance. Perks that grant "attack + move" or "free movement on kill" become high-value abilities.

### Tick Resolution Order

Each combat tick resolves in a fixed order:

1. **Effects Phase**: Per-tick status effects resolve (DoT damage, stack decay, regen ticks, environmental hazards)
2. **Initiative Phase**: All characters add `sqrt(Speed) × global_initiative_multiplier` to their Initiative meters
3. **Turns Phase**: Characters whose Initiative ≥ 100 take their turns in **global Initiative order** (highest first, with tie-breaking rules), interleaved between teams — there is no team-alternating turn structure. Characters evaluate and act **sequentially** — board state updates after each character acts, so later characters react to the updated state. See [combat-ai](../combat-ai.md) for team coordination details.
4. **Fallen Resolution**: A pure HP check at tick end — all characters whose HP is below 1 at this point enter the Fallen state. **Healing saves**: if a character was reduced below 0 HP earlier in the tick but healing (from allies, self-heal via Dying Blow, or Triggers) brought them back above 0 HP before this step, they do **not** enter Fallen. Characters enter Fallen in **reverse Initiative order** (lowest Initiative first); `OnFallen` Triggers fire sequentially in this order. Overkill values are recorded (overkill = abs(final_HP) regardless of damage source, including DoTs). Dying Blow Actions taken during the Turns Phase are recorded in the Combat Scoreboard.
5. **Kill Triggers**: Batch-fire all confirmed kill-related triggers (`OnKill`, `OnFallen`, `OnAllyFallen`) for characters who entered Fallen in Step 4. Each kill event tracks the causing Action and character. AoE kills from the same Action share a batch. See [Deferred Kill Trigger Model](vocabularies.md#deferred-kill-trigger-model) for details.

**One turn per tick**: Each character acts **at most once per tick**. If a character's Initiative remains ≥ 100 after acting (e.g., fast Action with high Speed), the excess carries over to the next tick. Very fast characters can act in consecutive ticks before anyone else gets a turn — appearing to chain actions — but each action is in a separate tick.

**Full-tick death deferral**: When a character's HP drops below 0 — whether from the Effects Phase (DoTs) or from another character's Action during the Turns Phase — they are **not** immediately removed. They finish the full tick: remaining effects resolve, they accumulate Initiative, and they can even take their turn if they have Initiative ≥ 100. The Fallen state is applied at the end of the tick (step 4). Actions taken by a character at ≤0 HP are **Dying Blows** (see below).

This order means DoTs and status decay happen before any character acts, Initiative accumulates before turns resolve, and death is always deferred to the end of the tick for maximum drama.

### Effects Phase Internal Ordering

The Effects Phase (Step 1 of tick resolution) resolves in a fixed internal order:

1. **Decay**: All stacks on all active statuses decay. Per-character ordering: highest stack count first (tiebreaker: alphabetical status name or definition order). All characters process decay simultaneously (no cross-character dependency).
2. **Effects**: All per-stack effects apply (DoT damage, regen ticks, stat modifications). Same per-character ordering (highest remaining stack first). All characters process effects simultaneously.
3. **Shield Expiry**: Expired shields are removed last.

**Key implication**: Shields protect against DoTs on their final tick — shields expire AFTER effects apply. A shield set to expire this tick still absorbs DoT damage dealt in the same Effects Phase.

### Dying Blows

When a character takes their turn while at ≤0 HP (between being reduced and the end-of-tick Fallen resolution), their Action is a **Dying Blow**. Dying Blows are:

- **Explicitly tracked** in the Combat Scoreboard (count of Dying Blow Actions per character)
- **Available as a Trigger condition**: `OnDyingBlow` fires when a character takes a Dying Blow action, enabling Perks like "Last Stand" (+100% Attack/Damage on Dying Blows) or "Dying Breath" (special effect on final action)
- **Dramatic moments**: A character reduced to 0 HP by another character's attack earlier in the Turns Phase can still act and potentially take down their killer before falling

### Initiative System

**Starting Initiative**: All characters begin combat with Initiative = 0 by default. Exceptions:
- **Surprise/ambush events**: Event configurations can grant specific teams an Initiative offset (e.g., ambushing team starts with one tick's worth of Initiative)
- **Perks and equipment affixes**: Can grant flat Initiative offset (e.g., "+10 starting Initiative") or scaled offset (e.g., "+sqrt(Speed) × global_multiplier" = one tick's head start)
- Starting Initiative offsets are applied before the first tick's Initiative Phase

**Speed-based meter with diminishing returns and a global multiplier**:
- Each tick, characters add `sqrt(Speed) × global_initiative_multiplier` to their Initiative meter
- **Global Initiative Multiplier**: A per-combat tuning value (default ~3.0) that controls how many characters act per tick. Higher = more turns per tick = faster combat. This knob directly controls match length without touching per-character balance.
- When Initiative ≥ 100, character takes a turn
- After action, Initiative reduced by `100 - Action Speed modifier` (see Action Speed below)
- Speed derived from: Speed Attribute + Equipment + Perks + Status Effects

**Diminishing returns**: Square root scaling means doubling Speed from 100→200 only increases Initiative gain from 10.0→~14.1 per tick before the global multiplier (41% increase for 100% more Speed).

**Haste and Slow effects** — two distinct channels:
- **Speed modification**: Directly changes the Speed stat (permanent or duration-based). Affects the `sqrt(Speed)` calculation.
- **Initiative gain modification**: Multiplies the per-tick Initiative gain independently of Speed (temporary buff/debuff). A "Haste" buff might grant +50% Initiative gain per tick without changing the Speed stat.

Both channels stack — a character can have both increased Speed AND an Initiative gain multiplier.

**Tie-Breaking Order** (when multiple characters reach Initiative ≥ 100 on the same tick):
1. Highest total Initiative (current + this tick's addition)
2. Highest Initiative added this tick
3. Highest Speed attribute
4. Highest Awareness attribute
5. Highest sum of all Attributes
6. Random coin flip

### Action Economy

A standard turn: **one Action**. All characters have access to default Actions via the hidden Combatant system Trait (see [Default Actions](systems.md#default-actions--combatant-system-trait) below). Additional Actions come from Perks.

**Default Actions** (available to all characters):
- **Attack**: Standard damage using equipped weapon's formulas. If no weapon equipped, uses a weak **unarmed attack** (Blunt damage, attribute-scaled with Might). **Costs a small amount of Stamina** (tuning value). See [Attack & Damage Formulas](resolution.md#attack--damage-formulas).
- **Move**: Change zones (costs full turn). **Free** (no resource cost). **Action Speed +25** (costs only 75 Initiative).
- **Defend**: Applies the **Defending** status (1 stack, special decay: clears on turn start). While active, provides a fixed percentage boost to **both** Physical Defense and Magic Defense, and **all three** Soak families (Physical, Elemental, Magical). Also provides a Stamina recovery burst equal to Y% of max Stamina pool (tuning value). **Free** (no resource cost). **Action Speed +25** (costs only 75 Initiative). If the character is stunned and cannot take turns, Defending persists (no tick limit — accepted as thematic: a braced character who gets stunned is harder to kill while helpless).
- **Search**: Active stealth detection — searches entire map (see [Stealth & Detection](systems.md#stealth--detection)). **Free** (no resource cost). **Action Speed +50** (costs only 50 Initiative).

**Defend + Stun design note**: The interaction where Defending persists through stun (since "clears on turn start" and stunned characters never get turns) is an accepted emergent strategy. A team could deliberately stun their own Defended tank for indefinite defense — this costs the stunner's Action and deals friendly fire damage from the stun-applying ability, creating a meaningful trade-off. Content can provide anti-stun and dispel mechanics as counters.

**Perk-granted Actions**: Active abilities with variable Action Speed, cooldowns, and resource costs. All cooldowns reset fully between fights (per-combat only — see [traits-and-perks](../traits-and-perks/index.md)).

Perks may grant bonus actions, free movement, or combined movement+attack. These are exceptions, not baseline.

### Action Speed

Action Speed is a flat modifier to the base −100 Initiative cost after acting.

- **Base cost**: −100 Initiative
- **Fast action**: Positive Action Speed modifier. Example: Action Speed +30 → character loses only 70 Initiative (100 − 30 = 70)
- **Slow action**: Negative Action Speed modifier. Example: Action Speed −40 → character loses 140 Initiative (100 − (−40) = 140)
- **Default Action Speeds**:
  - **Attack**: Action Speed 0 (costs 100 Initiative) — standard baseline
  - **Move**: Action Speed +25 (costs 75 Initiative) — quick non-offensive action
  - **Defend**: Action Speed +25 (costs 75 Initiative) — quick non-offensive action
  - **Search**: Action Speed +50 (costs 50 Initiative) — deliberately fast, map-wide detection at low cost

Fast actions let a character act again sooner; slow actions impose a longer delay. This creates meaningful trade-offs between powerful-but-slow abilities and weaker-but-fast ones. Non-offensive default Actions are faster than Attack to reward tactical decisions — a character who Defends or Searches recovers Initiative faster, making these genuinely competitive with attacking.
