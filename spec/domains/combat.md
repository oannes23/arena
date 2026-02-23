# Combat — Domain Specification

**Status**: 🟢 Complete (updated 2026-02-22: rounds 26-31 — 23 additional decisions: implementation readiness pass — hit bonus primary-only, deferred OnKill queue, Step 5 kill triggers, Effects Phase ordering, default Action Speeds, Revive Step 0, detection per-turn, trigger vocabulary expansion)
**Last interrogated**: 2026-02-22
**Last verified**: —
**Depends on**: [characters](characters.md), [traits-and-perks](traits-and-perks.md)
**Depended on by**: [combat-ai](combat-ai.md), [post-combat](post-combat.md), [tournaments](tournaments.md)

---

## Overview

Combat is the core gameplay loop — automated tactical fights between teams of characters on zone-based maps. Players configure builds and AI before combat; combat executes without player input. This domain covers the spatial system, initiative, action economy, combat resolution, damage types, status effects, stealth, targeting, resources, shields, attrition ramp, win conditions, and the combat scoreboard. Post-combat resolution (injuries, Perk discovery, recruitment, loot) is in [post-combat](post-combat.md). Combat AI configuration is in [combat-ai](combat-ai.md).

---

## Core Concepts

### Numeric System

All combat values are **integers**. Fractional results always **round up** (ceiling). This applies to damage, healing, Soak reduction, Initiative gain, formula evaluation, status decay, and all other combat calculations.

- **Minimum 1 damage per component**: Every damage component that successfully hits deals at least 1 damage after all reductions. Chip damage always matters.
- **Minimum 1 stack**: Status effects always apply at least 1 stack even after Resistance reduction (see [Status Effect System](#status-effect-system)).
- **Rounding direction**: Ceiling rounding favors attackers and status appliers — even tiny fractional damage rounds to 1. All formula evaluations produce integers.

### Battle Format

- **Scale**: 1v1 to 20v20. **2+ teams supported** — free-for-all, multi-faction, and asymmetric sizes (3v5, 1v10, etc.) are all valid configurations. "Enemy" = anyone not on your team.
- **Automation**: Pre-configured AI orders; combat executes automatically (idle/auto-battler). **Simulation-first**: the combat engine runs the entire fight to completion, then presents results (see [Combat Event Stream](#combat-event-stream--deterministic-simulation)).
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
3. **Turns Phase**: Characters whose Initiative ≥ 100 take their turns in **global Initiative order** (highest first, with tie-breaking rules), interleaved between teams — there is no team-alternating turn structure. Characters evaluate and act **sequentially** — board state updates after each character acts, so later characters react to the updated state. See [combat-ai](combat-ai.md) for team coordination details.
4. **Fallen Resolution**: A pure HP check at tick end — all characters whose HP is below 1 at this point enter the Fallen state. **Healing saves**: if a character was reduced below 0 HP earlier in the tick but healing (from allies, self-heal via Dying Blow, or Triggers) brought them back above 0 HP before this step, they do **not** enter Fallen. Characters enter Fallen in **reverse Initiative order** (lowest Initiative first); `OnFallen` Triggers fire sequentially in this order. Overkill values are recorded (overkill = abs(final_HP) regardless of damage source, including DoTs). Dying Blow Actions taken during the Turns Phase are recorded in the Combat Scoreboard.
5. **Kill Triggers**: Batch-fire all confirmed kill-related triggers (`OnKill`, `OnFallen`, `OnAllyFallen`) for characters who entered Fallen in Step 4. Each kill event tracks the causing Action and character. AoE kills from the same Action share a batch. See [Deferred Kill Trigger Model](#deferred-kill-trigger-model) for details.

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

A standard turn: **one Action**. All characters have access to default Actions via the hidden Combatant system Trait (see [Default Actions](#default-actions--combatant-system-trait) below). Additional Actions come from Perks.

**Default Actions** (available to all characters):
- **Attack**: Standard damage using equipped weapon's formulas. If no weapon equipped, uses a weak **unarmed attack** (Blunt damage, attribute-scaled with Might). **Costs a small amount of Stamina** (tuning value). See [Attack & Damage Formulas](#attack--damage-formulas).
- **Move**: Change zones (costs full turn). **Free** (no resource cost). **Action Speed +25** (costs only 75 Initiative).
- **Defend**: Applies the **Defending** status (1 stack, special decay: clears on turn start). While active, provides a fixed percentage boost to **both** Physical Defense and Magic Defense, and **all three** Soak families (Physical, Elemental, Magical). Also provides a Stamina recovery burst equal to Y% of max Stamina pool (tuning value). **Free** (no resource cost). **Action Speed +25** (costs only 75 Initiative). If the character is stunned and cannot take turns, Defending persists (no tick limit — accepted as thematic: a braced character who gets stunned is harder to kill while helpless).
- **Search**: Active stealth detection — searches entire map (see [Stealth & Detection](#stealth--detection)). **Free** (no resource cost). **Action Speed +50** (costs only 50 Initiative).

**Defend + Stun design note**: The interaction where Defending persists through stun (since "clears on turn start" and stunned characters never get turns) is an accepted emergent strategy. A team could deliberately stun their own Defended tank for indefinite defense — this costs the stunner's Action and deals friendly fire damage from the stun-applying ability, creating a meaningful trade-off. Content can provide anti-stun and dispel mechanics as counters.

**Perk-granted Actions**: Active abilities with variable Action Speed, cooldowns, and resource costs. All cooldowns reset fully between fights (per-combat only — see [traits-and-perks](traits-and-perks.md)).

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

### Combat Resolution Pipeline

All Actions go through a **unified pipeline** with component-driven branching. The pipeline inspects the Action's effect components to determine which steps apply — there is no explicit "action type" flag.

**Step 1: Attack Roll vs. Defense** *(enemy-targeted Damage components only)*
- Skipped entirely if the Action has no Damage components targeting enemies (heals, buffs, shields skip to Step 2)
- Attacker rolls 1 to [Attack Value]
- **AoE**: One roll for the entire Action. Each target compares the same roll against their own Defense independently — a high roll may hit all targets, a low roll may miss high-Defense targets but hit low-Defense ones.
- **Dual Defense**: Defender uses **Physical Defense** or **Magic Defense** based on the Action's **primary damage type**:
  - Physical family (Blunt, Piercing, Slashing) → Physical Defense
  - Elemental family (Fire, Cold, Lightning) → Magic Defense
  - Magical family (Poison, Shadow, Light, Psychic) → Magic Defense
- Defender's Defense = flat subtraction
- Result < 0: **Miss** — no damage for this target, but non-damage effect components (buffs, status) still resolve
- Result ≥ 0: **Hit** — the margin (roll − Defense) becomes the **hit bonus** for this target (used in Step 3)
- **Primary damage type**: Each Action with Damage components designates a primary damage type that determines which Defense to check. All Damage components in the Action share the same hit/miss result per target.
- **Friendly fire**: When an `[All]` targeting tag causes an Action to hit allies, the full pipeline applies — allies get a Defense roll, Soak, Resistance, and Shields as normal. See [Friendly Fire Rules](#friendly-fire-rules).

*Example*: 120 Attack vs. 50 Physical Defense → roll 1–120, subtract 50. Results: −49 (miss) to +70 (strong hit). Hit chance ≈ 58%.

**Step 2: Critical Hit Check** *(universal — applies to all effect components)*
- On a successful hit (or for Actions that skip Step 1), roll against the attacker's Critical Hit Chance
- Crit chance derived from Physical Crit (40% Awareness + 35% Luck + 25% Accuracy) or Magic Crit (40% Awareness + 35% Luck + 25% Intellect), based on primary damage type
- **Universal crit**: Crits apply to ALL effect component types. A crit heal heals more, a crit shield is stronger, a crit damage hits harder. Crit multiplies each component's output by `(1 + crit_multiplier)`.
- **Hit bonus is NOT amplified by crit** — it is added separately in Step 3 (see below)
- **Non-crit**: Proceed to Step 3 with base values

**Step 3: Effect Resolution (Type-Ordered)**

Effect components resolve in type order (see [Effect Resolution Order](#effect-resolution-order-within-actions)):

**3a. Status/Buff/Debuff effects** (ApplyStatus, RemoveStatus, ModifyStat)
- Resistance applies only to **harmful effects from enemies** (see [Friendly Fire Rules](#friendly-fire-rules))
- Each rider effect has its own resolution (e.g., Stunning Blow rolls against Stun Resistance)
- Status effects that modify Soak or other defensive stats take effect immediately, potentially affecting damage components in step 3b

**3b. Per-Component Damage Resolution**

Each Damage effect component resolves through its **own complete pipeline** independently:

1. **Base damage** from the component's `formula` field
2. **Crit multiplier** (if crit from Step 2): `base × (1 + crit_multiplier)`
3. **Damage modifiers**: buffs, debuffs, attrition ramp bonus
4. **Hit bonus** added as flat addition to the **primary damage type component only** (NOT amplified by crit or modifiers — raw accuracy reward). Secondary damage components in multi-component Actions do NOT receive the hit bonus. Prevents multi-component Actions from double-dipping on Attack investment.
5. **Block check**: Roll the defender's equipped shield's Block Chance %. On success, subtract Block Value proportionally across all damage components (see [Block Mechanics](#block-mechanics)). Applies to all damage types.
6. **Soak reduction**: `Reduction % = Effective_Soak / (Effective_Soak + K)`. Effective Soak = `Soak[family] - Effective_Pen` (min 0). Effective Pen = `Pen × K_pen / (Pen + K_pen)` (diminishing returns).
7. **Shield absorption**: Post-Soak damage passes through Shield HP pools (see [Shield Mechanics](#shield-mechanics)), FIFO order
8. **Minimum 1 damage**: Every component that hit deals at least 1 damage after all reductions
9. **Apply to Health**

*Example*: A "Flame Blade" Action with Slashing + Fire Damage components and primary type Slashing. One attack roll against Physical Defense. If hit, Slashing resolves against Physical Soak and Fire resolves against Elemental Soak independently. Block Value splits proportionally between them.

**3c. Healing effects** (Heal, Shield, ResourceGrant)
- No Defense or Soak — healing applies directly
- Crit multiplier applies (a crit heal heals more)

**3d. Movement effects** (Move, Summon)
- Resolve last

**Off-Hand Bonus Attack**: When dual-wielding or using weapon+focus and the main Action **targets enemies** (has any enemy-targeting component — Damage, debuff, etc.), the off-hand implement fires a **generic Attack** as a completely separate pipeline pass (own hit roll, own crit check, own damage resolution) after the main Action resolves. DW penalties apply (main-hand −15%, off-hand −35% Attack/Damage). Each implement's proc Triggers fire only on its own hits. **Does NOT fire** for non-offensive Actions: heals, self-buffs, Move, Defend, Search. A debuff-only Action targeting enemies DOES trigger the off-hand bonus attack. See [equipment](equipment.md) for full off-hand mechanics.

**Balance Implications**:
- Unified pipeline: one code path for all Actions, branching based on component presence
- Per-type Soak with diminishing returns prevents the "armor wall" problem across every damage type
- Dual Defense (Physical vs Magic) creates a meaningful split between martial and magical attackers
- Penetration has diminishing returns, preventing it from zeroing out Soak entirely
- Universal crit rewards crit investment for healers and support builds, not just attackers
- Hit bonus rewards Attack investment with raw damage that doesn't benefit from crit amplification
- Status-before-damage means debuffs applied by the same Action affect its own damage
- Per-component resolution rewards Actions with diverse damage types — each component hits a potentially different Soak value
- Minimum 1 damage per component ensures chip damage always matters

### Attack & Damage Formulas

Attack Value and Damage Value are **weapon-type-defined** — each weapon type specifies its own attribute blend formula for both. There are no fixed "Physical Attack" or "Magic Attack" derived stats.

**Weapon-type formulas** (illustrative — exact values are content data):
- **Swords**: Attack = Weapon Attack + balanced blend (Accuracy, Might, Speed). Damage = Weapon Damage + balanced blend.
- **Daggers**: Attack = Weapon Attack + Speed-heavy blend. Damage = Weapon Damage + Speed-heavy.
- **Warhammers**: Attack = Weapon Attack + Might-heavy blend. Damage = Weapon Damage + Might-heavy.
- **Bows**: Attack = Weapon Attack + Accuracy-heavy blend. Damage = Weapon Damage + Accuracy.
- **Staves/Wands**: Attack = Weapon Attack + Intellect-heavy blend. Damage = Weapon Damage + Intellect.

**Spell Actions**: Each spell Action defines its own formula in its Damage component's `formula` field (e.g., "30 base + 40% Intellect"). Spells are not dependent on weapon formulas — they define everything inline.

**Unarmed combat**: Characters with no weapon equipped have a baseline unarmed attack with Blunt damage type. Damage scales with Might (e.g., formula: `Might × 0.5`). Naturally weaker than any real weapon but functional. Perks can enhance unarmed combat (e.g., a Monk Perk could grant a new unarmed Action with better formulas and additional damage types).

**Formula modifier tags**: Perks can apply **formula modifier tags** — a Stat Adjustment subtype that substitutes one attribute for another in specific formula slots. For example:
- "Intelligent Strikes": substitute Intellect for Accuracy in all Attack formulas
- "Charismatic Hitter": substitute Charisma for Might in all Damage formulas
- **Conflict resolution**: When multiple modifiers target the same formula slot, the system evaluates all substitute candidates and uses whichever attribute value is **highest** for the character. The player always benefits from their best available option.

**Per-category scoping**: Each formula modifier tag specifies a **formula category scope** — which category of formulas it applies to (attack, damage, defense, healing, etc.). "Intelligent Strikes" applies to *attack* formulas only; "Charismatic Hitter" applies to *damage* formulas only. A modifier can target multiple categories if desired. This prevents a single modifier from universally replacing an attribute across all formulas.

Formula modifier tags are checked at formula evaluation time. They modify the attribute used in the calculation, not the formula structure itself.

### Formula Representation

Formulas throughout the combat system use a **structured weighted list** format for the common case, with an optional expression override for complex Perks/spells:

**Standard format** (~95% of formulas):
```
{
  base: <flat_value>,
  weights: { <attribute>: <weight>, ... }
}
```
Example: Physical Soak = `{ base: 0, weights: { Endurance: 0.60, Might: 0.25, Speed: 0.15 } }` (× scaling multiplier)

**Override format** (complex Perks/spells):
```
{
  formula_override: "<expression_string>"
}
```
Example: A Perk that scales with both missing HP and Willpower = `{ formula_override: "20 + (missing_hp_pct × 0.5 × Willpower)" }`

The structured format is preferred because it is machine-inspectable (AI Scorers can read weights, the balance system can analyze formulas, and modifier tags can substitute attributes). The expression override exists as an escape hatch for complex Perks or unique spell mechanics that cannot be expressed as a simple weighted blend.

### DoT Damage Rules

**DoT damage bypasses Soak entirely**. When status effects deal damage during the Effects Phase (Burning, Bleeding, Poison ticks), the damage is applied without Soak reduction.

- **Rationale**: DoTs already passed through the Resistance system when initially applied (which reduced their stack count). Double-gating DoTs through both Resistance and Soak would make them too weak to be meaningful.
- **DoTs bypass Block**: DoT tick damage occurs in the Effects Phase, outside the per-component damage pipeline where Block operates. Block only applies to incoming Actions during the Turns Phase.
- **Shields still absorb DoTs**: Shield HP sits between Soak and Health in the pipeline. Even though DoTs skip Soak and Block, they must still pass through any active Shields before reaching Health. Proactively shielding against DoTs is a valid defensive strategy.
- **DoT damage path**: DoT tick → Shield absorption → Health (bypasses both Soak and Block).
- **Resistance still applies at application**: The number of DoT stacks applied is reduced by Resistance when the status is first applied. Once applied, each stack ticks for its full per-stack damage.

### Passive Detection Timing

Passive stealth detection happens **before each character's turn** in the Turns Phase (not once per tick).

- Each character's passive detection check runs immediately before they act
- This means a character acts with the most current detection information — if an ally's turn revealed a stealth enemy earlier in the tick, the next ally to act already knows
- Passive detection remains same-zone only, using the character's Awareness stat
- Active detection (Search action) resolves during the character's turn as normal

### Summon Mechanics

The Summon effect component creates **Ephemeral Combatants** (from [characters](characters.md)):

- Summoned entities use the same stat model as all combatants but are not persistent — they exist only for the duration of this combat
- They have their own Initiative meter starting at **0** and act independently in turn order. Summoning is an investment — fast summons (high Speed) reach their first turn sooner
- They use preset AI configurations (defined by the summoning Perk/consumable)
- Team membership matches the summoner
- **Disappearance**: Summoned entities disappear when killed (enter Fallen) or when their summoner enters Fallen state (end-of-tick Fallen resolution removes both)
- Summoned entities do NOT trigger post-combat phases (no injury checks, no Perk discovery, no recruitment)

**Summon Control Capacity**: Summoners are limited by a **control capacity budget** — a Resource Pool granted by summoner Traits. Each summoner Trait that enables summoning defines a control capacity pool (e.g., "Necro Control" = 0.8×Willpower + 0.2×Charisma). Each summon type has a **control cost**. The summoner can maintain summons up to their total capacity; dying summons free capacity for new summons. This is a budget-based system, not a per-Perk cap — a summoner with 100 capacity can field five 20-cost skeletons or two 50-cost golems.

**Summon Death Triggers**: When summons are removed (killed or summoner enters Fallen), they fire full trigger events — `OnFallen` on the summon, `OnAllyFallen` on their teammates. This creates potential trigger cascades for summoner builds: a summoner with 5 active summons who enters Fallen generates 5× `OnAllyFallen` events (one for each summon removed) plus the summoner's own `OnFallen`. Content design should account for this cascade potential.

**Revived summoner**: If a summoner is revived mid-combat, their summons remain dead (they were removed during Fallen resolution). The summoner must re-summon using their restored control capacity.

### Stun Mechanics

Stun is a status effect that **prevents Initiative gain entirely**:

- While a character has Stun stacks > 0, they gain 0 Initiative per tick during the Initiative Phase
- This means stunned characters never reach Initiative ≥ 100 and never get turns — they are helpless
- **Decay**: Stun stacks decay by an amount proportional to Willpower per tick (during the Effects Phase, before Initiative Phase)
- A character with high Willpower recovers from Stun quickly; low Willpower characters may be stunned for many ticks
- Stun interacts with the one-turn-per-tick rule naturally — a partially stunned character (whose stacks decay to 0 mid-combat) resumes accumulating Initiative from wherever it was

### Friendly Fire Rules

When an Action with an `[All]` targeting tag (e.g., `[All, AoE-Zone]`, `[All, AoE-Map]`) hits allies:

- **Full pipeline applies**: Allies get their own Defense roll, Block check, Soak reduction, Shield absorption, and Resistance — same as enemies. Friendly fire is NOT free damage.
- **Resistance-skip is for BENEFICIAL effects only**: The "allies skip resistance" rule applies exclusively to beneficial effects — heals, buffs, and positive status effects. Harmful effects from allies (damage, debuffs from friendly fire) go through Resistance normally.
- **Rationale**: Without this rule, your own AoE would deal MORE damage to allies than to enemies (unresisted). The full pipeline prevents this design bug.

### Block Mechanics

**Block Chance** from equipped shield items (Buckler, Kite Shield, Tower Shield) is a damage reduction layer in the per-component damage pipeline (step 5 of damage resolution):

- **Trigger**: After a successful hit, roll the defender's shield's Block Chance percentage
- **On block**: Subtract the shield's **Block Value** (flat amount, quality-scaled) from total pre-Soak damage
- **Proportional split**: Block Value splits proportionally across all damage components in the Action based on their relative magnitudes. A Flame Blade dealing 60% Slashing / 40% Fire has Block Value split 60/40 between them.
- **All damage types**: Block applies to all damage types (Physical, Elemental, Magical). The shield physically intercepts the attack regardless of damage type.
- **One check per Action**: Block Chance is rolled once per incoming Action (not per component). If the Block succeeds, the Block Value split applies to all components.
- **Distinct from Shield HP**: Block Chance is from **equipped shield items** (Buckler/Kite/Tower in a Hand slot). Shield HP pools are from **Perks, consumables, and equipment effects** (the Shield effect component). These are two separate defensive mechanics.

See [equipment](equipment.md) for Block Chance and Block Value per shield type.

### Shield Mechanics

Shields are a **second defense layer** that absorbs damage after per-type Soak reduction:

- **Application order**: Attack Roll → Defense check (Physical or Magic) → Per-type Soak reduction → **Shield absorption** → remaining damage applied to Health
- **Shield HP**: Each Shield effect (from Perks, equipment, consumables) creates a separate Shield instance with its own HP pool and optional duration
- **Penetration bypass**: Penetration does NOT affect shields — it only reduces Soak. Shields absorb the post-Soak damage at full value
- **Stacking**: Multiple shields stack. Damage drains the **oldest shield first** (FIFO order). When a shield's HP reaches 0, remaining damage carries over to the next shield (or to Health if no shields remain)
- **Damage type filter**: Shields can optionally specify damage type filters (e.g., "absorbs Fire damage only"). Unfiltered shields absorb all damage types. Filtered shields are skipped for non-matching damage — damage passes through to the next shield or Health
- **Duration**: Shields may have a tick duration. Expired shields are removed at the start of the Effects Phase (alongside other status decay)

**Design Intent**: Shields provide a temporary defensive buffer that rewards proactive play (casting shields before taking damage). The oldest-first drain order means refreshing shields adds new HP on top of existing shields rather than overwriting them.

### Effect Resolution Order Within Actions

When an Action contains multiple effect components, they are resolved in **type order**, not simultaneously:

0. **Revive effects**: Revive components resolve first, before all other effects. This ensures hybrid heal+revive Actions work naturally: Revive brings the Fallen ally back (with clean slate — all statuses cleared), then subsequent buff and heal components apply to the now-living character.
1. **Status/Buff/Debuff effects**: ApplyStatus, RemoveStatus, ModifyStat components resolve second
2. **Damage/Healing effects**: Damage, Heal, Shield components resolve third
3. **Movement effects**: Move, Summon components resolve last

This means a single Action can debuff a target's Soak and then deal damage against the reduced Soak in the same use. Components within the same type category resolve simultaneously (no ordering within a category). Revive at Step 0 ensures that a hybrid Revive+Buff+Heal Action first revives (clearing all statuses), then applies fresh buffs, then heals — the revived character receives full benefit.

**Note**: This overrides the traits-and-perks spec's "simultaneous effect resolution within Actions" rule. The traits-and-perks spec references this spec for the authoritative resolution order.

### Critical Hit System

- **Chance**: Derived from Physical Crit (40% Awareness + 35% Luck + 25% Accuracy) or Magic Crit (40% Awareness + 35% Luck + 25% Intellect), based on the Action's primary damage type. For Actions without a primary damage type (pure heals/buffs), uses the higher of the two crit stats.
- **Trigger**: Checked after a successful hit, or for non-attack Actions, checked when the Action resolves (Step 2 of the pipeline)
- **Universal effect**: Crits apply to **all** effect component types. A crit adds a bonus amount to each component's output — bonus damage for Damage components, bonus healing for Heal components, bonus HP for Shield components, bonus stacks for ApplyStatus, etc.
- **Crit bonus formula**: `output = base × (1 + crit_multiplier)`. The `crit_multiplier` is a tuning value (e.g., 0.5 for +50% bonus on crit). This is a percentage of the base output, not a flat addition or separate roll.
- **Scaling**: Crit chance and crit multiplier can be modified by Perks and equipment

### Damage Types

**Physical**: Blunt (hammers, maces, unarmed), Piercing (spears, arrows, daggers), Slashing (swords, axes, claws)

**Elemental**: Fire (burning, explosions), Cold (freezing, chill), Lightning (shock, thunder)

**Magical**: Poison (toxins, disease), Shadow (darkness, necrotic), Light (holy, radiant), Psychic (mental, illusion)

**Philosophy**: No inherent rock-paper-scissors. Differences come from affiliated status effects and Perk/equipment interactions. Fire *tends* to inflict Burning, Cold *tends* to inflict Slow — thematic patterns, not hard rules.

**MVP Scope**: All 10 damage types are available in MVP. Each needs at least one Trait and representative Perks.

### Status Effect System

**Stack-Based Mechanics**:
- Max stacks: 9999 (effectively unlimited)
- **Minimum stacks**: Status effects always apply at least **1 stack**, even after Resistance reduction. Resistance can reduce 10 stacks to 1, but never to 0. This is an absolute rule with **no exceptions** — there is no "immunity" mechanic. High Resistance + fast decay provides functional near-immunity (1 stack that decays instantly).
- **Additive stacking**: New applications of the same status type **add stacks** to the existing count. A character with 5 stacks of Burning hit with 3 more stacks has 8 stacks. No separate duration tracking.
- **Stacks ARE duration**: There is no separate duration field. Each status type defines a **per-tick decay formula** that reduces stacks each tick. When stacks reach 0, the status is removed.
- **Frozen while Fallen**: Status effects do **not** tick down while a character is in the Fallen state. Revival clears all statuses (clean slate).
- **Character status tracking**: Just `{status_type, current_stacks}` — minimal data per active status.

**Status Decay Model**:

Each status type defines a decay formula evaluated per tick during the Effects Phase. The formula determines how many stacks are removed:

```
status_type_definition: {
  name: "Burning",
  decay_formula: <formula>,          // stacks removed per tick
  per_stack_effect: { ... },         // effect per stack per tick
  special_decay: null                // optional: "per_health_restored", "on_turn_start", etc.
}
```

Decay formulas can be:
- **Percentage**: `current_stacks × rate` (e.g., Burning: 25% per tick → 10→7→4→2→1→0). Ceiling rounding applies to decay removal: `ceil(10×0.25) = 3` stacks removed first tick, `ceil(7×0.25) = 2` next, etc. This makes percentage-decay statuses shorter-lived than they appear — Burning clears in 5 ticks, not 8.
- **Attribute-derived**: `Attribute / divisor` (e.g., Stun: `Willpower / 10` stacks removed per tick)
- **Flat**: Fixed amount per tick (e.g., Regeneration: lose 2 stacks per tick)
- **Special**: Custom conditions (e.g., Bleeding: lose 1 stack per Health restored; Defending: clears on turn start)

All decay results use **ceiling rounding** — even fractional removal rounds up to at least 1. This ensures statuses always decay and never get "stuck."

**No True Immunity**: There are no immunity Perks that prevent status application. Defense against powerful statuses comes from:
1. High Resistance (reduces applied stacks, minimum 1)
2. Fast decay (high Willpower for Stun, etc.)
3. The combination: 1 stack of Stun with Willpower 100 decays within the same tick — functionally zero impact.

**Resistance Model**:
- Status effects **always apply** — there is no binary resist/fail roll
- Resistance reduces the **number of stacks applied** AND **damage of that damage type**
- **Harmful enemy effects only**: Resistance applies only to harmful effects from enemy sources. Beneficial effects from allies (heals, buffs, positive statuses) always apply at full strength. Harmful effects from allies (friendly fire) go through Resistance normally (see [Friendly Fire Rules](#friendly-fire-rules)).
- Per-type resistance: characters can have different resistance values for different status/damage types
- Sources: Attributes (e.g., Willpower for mental statuses), Perks, equipment, other status effects

**Resistance Formulas** (percentage with diminishing returns — parallels Soak formula):
- **Stack reduction**: `stacks_applied = ceil(base_stacks × (1 - resist_ratio))` where `resist_ratio = Resistance / (Resistance + K)`. Minimum 1 stack always applies.
- **Damage reduction**: `ceil(damage × (1 - resist_ratio))` using the **same formula and same K** per damage/status type.
- Both use the same diminishing returns curve as Soak and Penetration. Higher Resistance is always valuable but never grants immunity. K is a per-type tuning constant.

*Example*: An attack applies 10 stacks of Burning. The defender has Fire Resistance 40 with K=50. `resist_ratio = 40/(40+50) = 0.44`. Stacks applied = `ceil(10 × 0.56) = ceil(5.6) = 6 stacks`. Fire damage taken is reduced by 44%.

**Status Examples** (numbers are tuning targets, not final):

| Status | Per-Stack Effect | Decay Formula |
|--------|-----------------|---------------|
| Stun | Prevents Initiative gain (0 gain while stacks > 0) | `Willpower / 10` stacks per tick |
| Burning | Fire DoT per tick | `current_stacks × 0.25` per tick |
| Bleeding | Physical DoT per tick | Special: 1 stack per Health restored |
| Slow | Reduces Initiative gain multiplier | `current_stacks × 0.20` per tick |
| Defending | % boost to Defense + Soak | Special: clears on turn start |
| Sneaking | Cannot be targeted — see Stealth | Broken by conditions (see Stealth) |
| Buff/Debuff | Stat modifications | Varies per status definition |

### Stealth & Detection

**Sneaking State**:
- While Sneaking, character cannot be targeted by single-target or selected-target enemy abilities
- **AoE hits Sneaking characters**: Zone-based AoE effects hit all characters in the zone, including Sneaking ones
- **Break conditions**:
  - Using a non-stealth-tagged Action breaks Sneaking (attacking, casting, etc.)
  - Stealth-tagged Actions can be used without breaking Sneaking (ambush attacks, stealth-specific abilities)
  - Taking damage has a **chance** to break Sneaking: `break_chance = damage_taken / (stealth_stacks × K)` where K is a tuning constant. Linear and transparent — high stealth stacks resist breaking, high damage makes it more likely
  - Being detected by an enemy does NOT automatically break Sneaking — it only allows that enemy to target the Sneaking character

**Detection System** (dual-mode):
- **Passive Detection**: Automatic, same-zone only. Before each character's turn, passive detection re-rolls against Sneaking enemies in their zone using Awareness. No action cost. **Per-turn only** — there is no persistent "this team has detected character X" state between turns. A character who detects a stealth enemy can target them during their turn only; teammates who act later in the same tick must make their own detection checks.
- **Active Detection (Search)**: A deliberate Action that searches the entire map. Detection chance decreases with range: same zone (high), adjacent zones (moderate), distant zones (low). Costs the character's turn but has Action Speed +50 (only −50 Initiative cost). Same per-turn detection persistence — Search results apply to the searching character's turn only.
- **Detection result**: Successfully detecting a Sneaking character allows the detector to target that character during their turn. Detection does not remove the Sneaking status — the Sneaking character retains other stealth benefits (damage break chance, etc.).

### Targeting

Actions use a **tag-based targeting system** (defined in [traits-and-perks](traits-and-perks.md)).

**Core Target Tags** (MVP set — extensible):

| Tag | Resolution |
|-----|-----------|
| `[Self]` | The acting character only |
| `[Enemy, Single]` | One enemy, selected by AI |
| `[Ally, Single]` | One ally, selected by AI |
| `[Enemy, Random]` | One random enemy |
| `[Ally, Random]` | One random ally |
| `[Enemy, AoE-Zone]` | All enemies in a target zone |
| `[Ally, AoE-Zone]` | All allies in a target zone |
| `[All, AoE-Zone]` | Everyone in a target zone (friendly fire) |
| `[Enemy, AoE-Map]` | All enemies on the map |
| `[All, AoE-Map]` | Everyone on the map |
| `[Fallen, Ally, Single]` | One Fallen ally, selected by AI (Revive-only targeting) |

**`[Fallen]` target tag**: Used by Revive-type Actions that exclusively target Fallen allies. Only Actions with the `[Fallen]` tag restrict themselves to Fallen targets.

**Revive on any ally**: The Revive effect component itself works on **any ally target** — if the target is Fallen, the Revive component resurrects them; if the target is not Fallen, the Revive component does nothing but other effect components in the Action still resolve normally. This enables hybrid heal+revive Actions (e.g., `[Ally, Single]` with both Heal and Revive components — heals living allies, revives Fallen ones). Actions that should **only** target Fallen allies use the `[Fallen, Ally, Single]` tag to restrict targeting.

New targeting modes are new tags — no schema changes required. Target tag resolution respects range constraints (each Action specifies its valid range band).

### AoE Resolution

When an AoE Action hits multiple targets, resolution uses a **batch-then-triggers** model:

1. **Batch resolution**: All targets in the AoE resolve through the full combat pipeline (hit roll, crit check, effect resolution, per-component damage) **simultaneously**. A target dying from the AoE does not affect other targets' resolution within the same batch.
2. **Trigger collection**: All resulting Triggers from the batch (OnHit, OnKill, OnDamageDealt, etc.) are collected but not yet fired.
3. **Trigger resolution**: All collected Triggers fire together after the batch completes. Trigger effects from multiple kills resolve simultaneously.

This means an AoE that kills 3 enemies would: (1) resolve damage against all 3 simultaneously, (2) collect three OnKill triggers, (3) fire all OnKill triggers together. A Perk like "Blood Frenzy" (OnKill: gain +10% damage) would gain its stacks from all kills at once.

### Per-Damage-Type Soak

Soak is **per-damage-type**, using three family-level base formulas. Types within a family share the same base Soak value; per-type differentiation comes from Perks and equipment only.

**Family Soak Formulas** (attribute blend weights × per-stat scaling multiplier):
- **Physical Soak** (Blunt, Piercing, Slashing): 60% Endurance + 25% Might + 15% Speed
- **Elemental Soak** (Fire, Cold, Lightning): 45% Endurance + 35% Awareness + 20% Luck
- **Magical Soak** (Poison, Shadow, Light, Psychic): 55% Willpower + 25% Intellect + 20% Luck

**Soak formula**: `Reduction % = Soak[family] / (Soak[family] + K)` where K is a global tuning constant. Higher Soak is always valuable but never grants immunity.

**Penetration**: Universal — one Penetration stat with **diminishing returns** applied against whatever family Soak is being checked. `Effective Soak = Soak[family] - Effective_Pen` (minimum 0), where `Effective_Pen = Pen × K_pen / (Pen + K_pen)` (K_pen is a tuning constant). Same harmonic diminishing returns shape as Soak — high Penetration is always valuable but can never fully zero out Soak. Penetration does NOT affect Shields (only reduces Soak).

*Example with K_pen=100*: Pen 20 → Effective 17. Pen 50 → 33. Pen 100 → 50. Pen 200 → 67. Penetration asymptotically approaches K_pen.

**Per-type bonuses**: Tag-scoped Stat Adjustments (e.g., "+5 Soak vs [Fire]") add to the family base for specific damage types. A character with 50 Elemental Soak and "+10 Soak vs [Fire]" has effective 60 Soak against Fire but 50 against Cold and Lightning.

### Dual Defense System

Two Defense values, each a derived stat:

- **Physical Defense**: 40% Speed + 35% Accuracy + 25% Awareness (× scaling multiplier). Used for physical attacks (Blunt, Piercing, Slashing). Represents dodge, evasion, and reading physical attacks.
- **Magic Defense**: 60% Willpower + 25% Awareness + 15% Luck (× scaling multiplier). Used for elemental and magical attacks (Fire, Cold, Lightning, Poison, Shadow, Light, Psychic). Represents mental fortitude and magical resistance.

The Action's **primary damage type** determines which Defense is checked (see [Combat Resolution Pipeline](#combat-resolution-pipeline)). Actions with both physical and magical damage components use the primary type for the single hit/miss roll.

### Combat Event Stream & Deterministic Simulation

**Deterministic simulation**: Combat uses a **seeded PRNG**. Same seed + same inputs = identical replay. The seed is stored alongside the event log, enabling:
- Exact fight replays
- Deterministic testing and bug reproduction
- Verifiable results (anti-cheat, tournament integrity)

**Combat event stream**: The engine emits a stream of **structured typed events** during simulation:
- Each significant combat action produces one or more typed events (e.g., AttackEvent, DamageEvent, FallenEvent, MovementEvent, HealEvent, StatusEvent, DyingBlowEvent)
- Events contain all relevant data (actor, target, values, results)
- The full event stream is stored as the authoritative combat log

**Presentation model**: Combat simulates the entire fight to completion before presenting results. The TUI offers:
- **Dramatized summary**: A condensed narrative of key moments (generated from the event stream, potentially LLM-enhanced)
- **Detailed event logs**: Full tick-by-tick event stream review for players who want depth
- **Combat Scoreboard**: Per-character stat summary (see below)

The specific event type catalog is an implementation detail — the spec establishes that a structured event stream exists and what it enables.

### Resources

**Universal** (all characters):
- **Health**: Fully recovers between battles. **No in-combat regeneration** — all healing comes from Perks, consumables, or equipment procs.
- **Stamina**: General-purpose physical action resource. **Percentage-based passive regen**: X% of max Stamina pool per tick (tuning value) + larger recovery burst from the Defend action (Y% of max pool). When Stamina reaches 0, physical Actions cost Health at a **1:1 ratio** — no penalty multiplier (exhaustion mechanic from [characters](characters.md)).

**Trait-granted** (via Perk content — see [traits-and-perks](traits-and-perks.md)):
- Mana, Faith, Spirit, Focus, and content-author-defined resource types
- **No default in-combat regen** for Trait-granted resources. Regeneration comes from Perks and Triggers — Perks that define a resource pool often bundle a regen Trigger as part of the same Perk or a companion Perk in the same Trait tree.
- Resource pools start at full capacity at the beginning of combat
- **Resource scale**: Hundreds to thousands for granular tuning (governed by per-stat scaling multipliers)

**Resource Depletion Rules**: Only Stamina has the Health substitution mechanic (Exhaustion — see [characters](characters.md)). When any other resource pool (Mana, Faith, Spirit, Focus, etc.) is depleted, Actions that require that resource are simply **unavailable** — the `resource_gate` vetoes them. There is no fallback or substitution. This makes resource management a genuine strategic concern for non-Stamina builds.

### Judgment — AI Quality Stat

Judgment is a derived combat stat that controls how effectively a character's AI evaluates and selects Actions during combat. See [combat-ai](combat-ai.md) for the full Utility AI system.

**Formula**: 40% Awareness + 35% Willpower + 25% Intellect

**Function** — controls three AI parameters:
- **Tactical-vs-Personality blend**: How much the character weighs objective effectiveness vs personality impulses. Range ~0.3 (low Judgment) to ~0.95 (high Judgment).
- **Selection sharpness**: How reliably the character picks the top-scoring Action vs making "suboptimal" choices. Range ~1 (low Judgment) to ~10 (high Judgment).
- **Lookahead depth**: How many ticks ahead the character can project deterministic game-state events (Effects Phase + Initiative timing). Range 1 tick (low Judgment) to 10 ticks (high Judgment). See [combat-ai](combat-ai.md) for the full lookahead system.

**Modifiers** beyond the base formula:
- Character Star Rating (higher stars = modest baseline boost)
- Specific Perks and equipment affixes
- Status effects (confusion, fear could lower it temporarily)

Judgment uses the same per-stat scaling multiplier model as other derived stats. The exact multiplier and the functions mapping Judgment to blend/sharpness/lookahead values are tuning parameters.

### Combat Context Flags

At the start of each fight, the combat system assembles an explicit **context flags** object and passes it to the AI system. This metadata describes the fight circumstances — tournament round, exhibition status, PvE tier, team sizes, consumable replenishment rules, attrition ramp parameters, etc. Any AI Scorer can query these flags to adjust behavior based on context.

**Canonical schema** (owned by combat.md — AI system consumes read-only):

```
combat_context: {
  is_tournament: bool,
  round: int,                    // current round (1-indexed)
  total_rounds: int,             // total rounds in this event
  consumables_replenish: bool,   // do consumables reset between rounds?
  is_exhibition: bool,
  pve_tier: int | null,          // null if not PvE
  team_sizes: [int],             // list of team sizes, own team first (e.g., [3, 5, 2])
  attrition_ramp_onset: int,     // tick at which attrition ramp activates
  attrition_ramp_rate: float,    // % damage bonus per tick after onset
  ...                            // extensible for future flags
}
```

Context flags are assembled by the combat system from the event configuration. The AI system consumes them read-only. See [combat-ai](combat-ai.md) for how Scorers use these flags.

### Resource Pool Scaling Multipliers

Each derived stat (including resource pools) has a per-stat **scaling multiplier** that converts the weighted attribute blend into game-scale values.

**Formula-based defaults**: Multipliers are derived from desired gameplay ranges rather than hand-tuned individually. Starting approach:
- Determine desired range for each stat at target attribute levels (e.g., "Health should be 400–800 for a typical 3★ character")
- Back-calculate the multiplier from the blend formula and target range
- Health Pool starts at ×10 as baseline (from [characters](characters.md))
- All multipliers are tuning values — the formula-based approach sets initial values; playtesting refines them

### Default Actions — Combatant System Trait

Every character has a hidden **"Combatant" system Trait** at level 1. This Trait is:
- **Invisible**: Not displayed in the Trait list UI. Does not occupy a Trait Slot.
- **Immutable**: Cannot be leveled, respecced, or modified.
- **Universal**: Automatically present on all characters (player-owned, Named NPCs, and Ephemeral Combatants).

The Combatant Trait contains a single Perk with the default Actions (Attack, Defend, Move, Search) as standard Perk Action components. This means:
- Default Actions are real Perk components that go through the same resolution pipeline as all other Actions
- No special-case code for "basic attacks" — everything is a Perk-granted Action
- The Combatant Trait's Perk can be extended in future phases to grant additional default capabilities

### Fallen State and Revival

- **Fallen**: When a character's HP drops below 1, they enter the Fallen sub-state (from [characters](characters.md)). They are out of the fight for its remainder (unless revived).
- **Fallen Resolution is a pure HP check**: At the end of each tick (step 4), the system checks whether HP < 1. If a character was reduced below 0 HP earlier in the tick but was healed back above 0 HP before step 4 (by an ally's Action, a self-heal on a Dying Blow turn, or a Trigger), they are **not** Fallen. Healing saves from Fallen.
- **Self-save via Dying Blow**: A character at ≤0 HP who takes their turn (Dying Blow) can use a self-heal Action. If the heal brings them above 0 HP before Fallen Resolution, they survive. Consistent rules — no special cases.
- **Revival**: Fallen characters **can** be revived mid-combat via Perks, consumables, or rare equipment effects. A revived character returns at a fraction of their maximum Health (exact fraction is a tuning value).
- **Revival: true clean slate**: **ALL** status effects are cleared on revival — both beneficial and harmful. There is no "buff vs debuff" classification; revival is a complete reset. Buffs applied before falling are lost. Hybrid revive+buff Actions work by applying fresh buffs AFTER the clean slate (via the type-ordered resolution: Revive Step 0 → Status Step 1). The revived character starts fresh — only the fractional HP remains from the pre-Fallen state.
- **Fallen Resolution order**: Characters enter Fallen in **reverse Initiative order** (lowest Initiative first). `OnFallen` Triggers fire sequentially in this order. This means higher-Initiative characters' Fallen Triggers resolve last, which can matter for chain reactions.
- **Status decay frozen while Fallen**: Status effects do not tick down while a character is in the Fallen state. If revived, statuses resume from where they were (but see "clean slate" above — revival clears all statuses, so this only matters for effects that occur between Falling and potential Revival within the same tick).
- **Overkill**: The amount of excess damage beyond 0 HP is tracked. Overkill = `abs(final_HP)` regardless of damage source — DoTs, direct damage, and environmental effects all contribute. Overkill magnitude affects injury severity in the post-combat injury roll — massive overkill increases the chance and severity of injuries.

### Attrition Ramp

An anti-stalemate mechanic that ensures all fights converge to a conclusion:

- **Onset**: After a configurable onset tick (default ~100), the ramp activates. Onset tick is a per-event-type value (specified in [Combat Context Flags](#combat-context-flags)).
- **Effect**: A **stacking global damage bonus** (+0.5–1% per tick, tuning value) applies to ALL combatants once the ramp is active. The bonus increases every tick after onset.
- **Scope**: Damage only — healing is unaffected. This means healing becomes proportionally less effective as the ramp escalates, naturally pushing fights toward resolution.
- **No hard tick limit**: There is no maximum tick count. The exponentially increasing damage bonus makes extended fights increasingly lethal, providing a soft cap on fight length.
- **Per-event-type configuration**: Both the onset tick and escalation rate are specified per event type via Context Flags. PvE fodder encounters might set onset at ~30 ticks with a faster ramp; championship finals might set onset at ~100 with a slower ramp.

**Design Intent**: The attrition ramp replaces the need for a hard time limit. Fights that naturally resolve quickly (mismatched teams, burst-heavy compositions) never trigger the ramp. Stalemate-prone compositions (double tank, pure healer builds) are gradually pushed toward a conclusion. The ramp onset tick serves as the "expected fight length" for a given event type.

### Fight Phase Signal

Combat exposes a **fight phase signal** to the AI system and Triggers:

```
fight_phase = current_tick / estimated_max_ticks
```

Where `estimated_max_ticks` = the attrition ramp onset tick (from Context Flags). This produces a 0.0–1.0+ ratio:
- **< 1.0**: Normal combat phase (before attrition ramp)
- **≥ 1.0**: Attrition phase (ramp is active, fight is running long)

The `resource_efficiency` Scorer in [combat-ai](combat-ai.md) uses this signal for fight phase detection: conserve resources early (fight_phase < 0.5), balanced mid-fight (0.5–0.8), spend freely late (> 0.8). Triggers can also reference fight_phase for phase-dependent effects.

### Post-Combat Handoff

When victory or defeat is declared (last team standing), combat ends and hands off to the [post-combat](post-combat.md) system. Combat provides:
- The **Combat Scoreboard** (see below) with per-character stats
- The list of **Fallen characters** with overkill values
- The set of **Traits that were active** during combat (for Perk Discovery rolls)
- The **Combat Context Flags** (exhibition status, event type, etc.)

See [post-combat](post-combat.md) for the full post-combat flow: injury/death checks, Perk discovery, recruitment, and loot distribution.

### Combat Scoreboard

A structured per-character stat block tracked throughout combat and available to the [post-combat](post-combat.md) spec and [meta-balance](../architecture/meta-balance.md) system:

| Stat | Description |
|------|-------------|
| Damage Dealt | Total damage dealt (post-Soak, post-Shield) |
| Damage Taken | Total damage taken (post-Soak, post-Shield) |
| Damage Healed | Total healing done to self and allies |
| Kills | Number of enemies reduced to Fallen |
| Actions Used (by type) | Count of each Action used (e.g., "Flame Burst: 3, Attack: 7") |
| Fallen Events | Number of times this character entered Fallen state (relevant with revival) |
| Revival Events | Number of times this character was revived |
| Dying Blows | Number of Actions taken while at ≤0 HP (before end-of-tick Fallen resolution) |
| Ticks Survived | Number of ticks the character was active (not Fallen) |

The scoreboard is purely observational — it does not affect combat mechanics. It feeds into post-combat Perk Discovery (determining which Traits were "active") and the meta-balance system (tracking win/loss per Trait).

### Zone Effects (Phase 4)

Zone-targeted effects (e.g., "create a fire zone that damages everyone inside") are explicitly deferred to Phase 4. MVP combat uses zones for positioning and range only — no persistent zone-based effects or terrain modifiers.

### Morale (Phase 4)

Fighters have a Morale value affected by combat events. Low Morale may cause debuffs, fleeing, or refusal to act. High Morale may grant bonuses. Full design TBD.

**Crowd Appeal**: The derived stat (60% Charisma + 25% Luck + 15% Awareness, from [characters](characters.md)) is part of the Morale system, deferred to Phase 4. For MVP, Crowd Appeal exists as a derived stat with no mechanical combat effect.

---

## Canonical Vocabularies

These vocabularies define the core set of system-recognized keywords. All are extensible — new entries can be added as content without schema changes.

### Target Tags

See [Targeting](#targeting) above for the MVP target tag set and resolution rules.

### Trigger Event Types

The authoritative list of events that can fire Perk Triggers.

**Trigger Recursion Guard**: Triggers can chain — a Trigger's effect can cause other Triggers to fire, which can chain further. However, each individual Trigger instance can fire **at most once per originating event**. Implementation: maintain a "fired set" per originating event; skip Triggers already in the set. This prevents infinite loops while allowing complex chains (e.g., OnHit → ApplyBurning → OnStatusApplied → bonus damage → OnDamageDealt → ..., but each Trigger fires at most once).

**Combat Flow Events:**
- `OnCombatStart` — at the beginning of combat (before first tick)
- `OnCombatEnd` — at the end of combat (after last tick)
- `OnTickStart` — at the start of each tick (during Effects Phase)

**Turn Events:**
- `OnTurnStart` — at the start of this character's turn
- `OnTurnEnd` — at the end of this character's turn

**Attack Events:**
- `OnHit` — when this character successfully hits an enemy (Step 1 result ≥ 0)
- `OnCrit` — when this character lands a critical hit
- `OnMiss` — when this character's attack misses
- `OnHitBy` — when this character is hit by an enemy
- `OnCritBy` — when this character is hit by a critical hit

**Damage Events:**
- `OnDamageDealt` — when this character deals damage (after Soak reduction). Fires immediately during the Action.
- `OnDamageTaken` — when this character takes damage. Fires immediately during the Action.

**Kill/Fall Events** (deferred — see [Deferred Kill Trigger Model](#deferred-kill-trigger-model)):
- `OnKill` — when this character drops an enemy to Fallen (fires in Step 5, not immediately). Event payload includes overkill value data.
- `OnAllyFallen` — when a friendly character falls in combat (fires in Step 5)
- `OnFallen` — when this character enters the Fallen state (fires in Step 5)
- `OnDyingBlow` — when this character takes an Action while at ≤0 HP (before end-of-tick Fallen resolution). Fires immediately during the Action.

**Revival Events:**
- `OnRevive` — when this character is revived (fires during Step 0 of effect resolution, immediate trigger). Fires on the revived character.

**Defense Events:**
- `OnBlock` — when this character's shield item successfully blocks (fires at pipeline step 5, joins Action trigger collection)
- `OnShieldBreak` — when a Shield HP pool on this character reaches 0 (fires at pipeline step 7, joins Action trigger collection)
- `OnDodge` — when this character successfully dodges an enemy attack (defender's perspective on a miss, roll < 0). Symmetric with OnMiss. Fires as part of the Action's trigger collection.

**Healing Events:**
- `OnHeal` — when this character heals an ally (healer's perspective, symmetric with OnDamageDealt). Fires immediately during the Action.
- `OnHealedBy` — when this character receives healing from any source (symmetric with OnDamageTaken). Fires immediately during the Action.

**Summon Events:**
- `OnSummon` — when this character creates a summon via the Summon effect component. Fires immediately during the Action.

**Status Events:**
- `OnStatusApplied` — when a status effect is applied to this character
- `OnStatusExpired` — when a status effect fully decays from this character

**Resource Events:**
- `OnResourceDepleted` — when a resource pool reaches zero
- `OnStaminaExhausted` — when Stamina hits 0 (exhaustion trigger)

**Movement Events:**
- `OnMove` — when this character changes zones
- `OnForcedMove` — when this character is pushed/pulled to a different zone

**Stealth Events:**
- `OnStealthBreak` — when this character's Sneaking status is broken
- `OnDetect` — when this character detects a Sneaking enemy

### Trigger Timing Rules

Triggers fire at different points depending on their type:
- **Immediate triggers** (fire during the Action): `OnHit`, `OnCrit`, `OnMiss`, `OnHitBy`, `OnCritBy`, `OnDamageDealt`, `OnDamageTaken`, `OnDyingBlow`, `OnBlock`, `OnShieldBreak`, `OnDodge`, `OnHeal`, `OnHealedBy`, `OnSummon`, `OnStatusApplied`, `OnStatusExpired`, `OnResourceDepleted`, `OnStaminaExhausted`, `OnMove`, `OnForcedMove`, `OnStealthBreak`, `OnDetect`. These fire as part of the Action's trigger collection (batch-then-triggers model for AoE).
- **Revive trigger**: `OnRevive` fires during Step 0 of effect resolution as an immediate trigger on the revived character.
- **Kill triggers** (deferred): `OnKill`, `OnFallen`, `OnAllyFallen` fire in Step 5 of tick resolution (after Fallen Resolution). See [Deferred Kill Trigger Model](#deferred-kill-trigger-model).
- **Flow triggers**: `OnCombatStart`, `OnCombatEnd`, `OnTickStart`, `OnTurnStart`, `OnTurnEnd` fire at their named points in the combat flow.

### Deferred Kill Trigger Model

Kill-related triggers (`OnKill`, `OnFallen`, `OnAllyFallen`) use a **deferred trigger queue** rather than firing immediately:

1. **During Actions**: When a target is reduced to HP < 0, the potential kill is recorded with the causing Action and character. The kill trigger does NOT fire yet.
2. **Step 4 (Fallen Resolution)**: Characters who actually have HP < 1 at tick end are confirmed as Fallen. Characters who were reduced below 0 HP but healed back above 0 before Step 4 do NOT generate kill triggers.
3. **Step 5 (Kill Triggers)**: Confirmed kill triggers fire as a batch. Each kill event tracks the causing Action for attribution. AoE kills from the same Action share a batch.
4. **Recursion guard**: Each kill in Step 5 is a separate originating event for trigger chain purposes (the standard per-event "fired set" applies independently per kill).

**Why deferred?** Immediate OnKill during Actions would fire for targets at HP < 0 who might later be healed back above 0 before Fallen Resolution — creating false kills. Deferring to Step 5 ensures kill triggers only fire for characters who actually enter Fallen.

**Interaction with other triggers**: `OnHit`, `OnDamageDealt`, and all other Action-phase triggers still fire immediately during the Action (unchanged). Only kill-related triggers are deferred.

### Effect Component Types

The authoritative catalog of effect component types used by Actions and Triggers:

| Type | Parameters | Example |
|------|-----------|---------|
| **Damage** | damage_type, formula, [penetration] | Deal 20 [Fire] damage |
| **Heal** | formula, [target] | Restore 30 HP |
| **ApplyStatus** | status_type, stacks, [duration] | Apply 3 stacks of Burning for 5 ticks |
| **RemoveStatus** | status_type, stacks | Remove 5 stacks of Poison |
| **ModifyStat** | stat_key, value, [duration] | Reduce target's Soak by 10 for 3 ticks |
| **Move** | target, direction/zone | Push target 1 zone away |
| **Shield** | formula, [duration], [damage_types] | Grant 50-point shield for 3 ticks |
| **Summon** | template, [zone] | Summon a Fire Elemental in current zone |
| **Revive** | health_fraction | Revive Fallen ally at 25% HP (does nothing on non-Fallen target; other components still resolve) |
| **ResourceDrain** | resource_type, amount | Drain 30 Mana from target |
| **ResourceGrant** | resource_type, amount | Restore 20 Stamina to self |

New types can be added without schema changes — the list format is inherently extensible.

**Resolution within Actions**: Effect components resolve in type order (see [Effect Resolution Order Within Actions](#effect-resolution-order-within-actions)).

**Level Scaling**: All numeric output values scale with Perk/Trait level multipliers. Resource costs, cooldowns, and requirements stay flat.

---

## Equipment Combat Mechanics (Cross-Reference)

Several combat-time mechanics are defined in [equipment](equipment.md) and affect combat resolution. This section lists them for implementers; see the equipment spec for full details.

| Mechanic | Summary | See Equipment Spec |
|----------|---------|-------------------|
| **Implement Resolution** | Actions specify physical or magical implement type. Resolution order: main-hand → off-hand → default (Unarmed/Unfocused). | Hand Slots & Implements |
| **Off-Hand Bonus Attack** | Enemy-targeted Actions (damage, debuff) trigger a generic Attack from the off-hand implement as a sequential second pipeline pass. Does NOT fire for heals, self-buffs, Move, Defend, Search. DW penalties: main-hand −15%, off-hand −35% Attack/Damage. | Off-Hand Bonus Attack |
| **Dual-Wield Penalties** | Main-hand −15% Attack/Damage, off-hand −35%. Raw DW ≈ 75% output. Perks (Ambidextrous, Two-Weapon Fighting) mitigate. | Dual-Wield |
| **Armor Speed Penalties** | Medium armor: −2 Speed per piece. Heavy armor: −5 Speed per piece. Full Heavy = −30 Speed. **Mitigated by Might** (see below). | Armor Types |
| **Might Armor Speed Mitigation** | `effective_penalty = base_penalty × (1 - Might/(Might+K))`. High Might reduces armor Speed penalties. See below. | (Defined here) |
| **Versatile Mode** | Versatile weapons auto-switch between one-hand and two-hand stat profiles based on off-hand occupancy. | Wielding Modes |
| **Equipment Combat Lock** | Equipment snapshotted at combat start for all fight types. No mid-combat gear swaps. Degradation calculated post-combat. | Equipment Combat Lock |
| **Default Attack Adaptation** | Single Attack Action adapts to main-hand implement: weapon → physical, focus → magical, empty → Unarmed. | Default Attack Adaptation |
| **Multihit Targeting** | Implements with the Multihit tag change the default Attack to hit 1–5 random enemies in the target zone (triangular distribution). One attack roll, per-target Defense check. | Implement Tags |
| **Reach Range Extension** | Reach-tagged weapons can attack at Medium range (extending from Short) with an Accuracy penalty (~20% Attack Value reduction at Medium). | Weapon Types |
| **Ranged Short Penalty** | Ranged-tagged weapons attack at Long range by default. ~20% Attack Value Accuracy penalty when used at Short range. | Weapon Types |
| **Block Chance** | Shield items (Buckler/Kite/Tower) provide Block Chance % and Block Value in the damage pipeline. See [Block Mechanics](#block-mechanics). | Shields |
| **Implement Tags** | Deadly (+crit, per-implement), Sunder (+Penetration, per-implement), Defend (+Physical Defense, passive), Deflect (+Magic Defense, passive), Multihit (AoE default Attack). | Implement Tags |

### Might Armor Speed Mitigation

Might reduces the Speed penalty from Medium and Heavy armor using the same diminishing returns formula used throughout the combat system:

```
effective_penalty = base_penalty × (1 - Might / (Might + K_armor))
```

Where `K_armor` is a tuning constant. This creates a natural Might + Heavy Armor synergy:
- A high-Might warrior in full Heavy armor (base −30 Speed) with Might 150 and K=100 reduces the penalty to about −12
- A low-Might mage in the same armor would suffer nearly the full −30 penalty
- Thematic: strong fighters wear heavy gear efficiently; mages are pushed toward Light armor

This gives Might a unique combat niche beyond weapon damage — it's the "I can wear heavy armor effectively" stat.

---

## Decisions

### Roll-vs-Static Defense Model

- **Decision**: Attacker rolls, defender provides flat threshold. Two-step: Attack vs. Defense (flat subtraction), then Damage vs. Soak (percentage reduction).
- **Rationale**: All variance on attacker's side makes defenders predictably durable. Flat Defense creates a binary hit/miss gate. Percentage Soak prevents the "armor wall" problem.
- **Implications**: Defense stacking creates hard tier gating (intentional). Soak stacking has diminishing returns. Penetration exists as a counter-stat.

### Percentage Soak with Diminishing Returns

- **Decision**: Soak uses `Reduction % = Soak / (Soak + K)` instead of flat subtraction. Penetration reduces effective Soak before the formula.
- **Rationale**: Flat Soak (the original model) made high-armor characters nearly immune to chip damage. Percentage with diminishing returns means Soak is always valuable but never makes a character invulnerable. Penetration creates meaningful build choices for attackers.
- **Implications**: K is a global tuning constant controlling how quickly Soak reaches diminishing returns. Equipment spec must define Penetration as a valid affix stat. Balance must tune K alongside damage ranges.
- **Alternatives considered**: Flat Soak (original model — rejected due to armor wall problem), threshold-based tiers (rejected — less granular than continuous formula).

### Critical Hit: Extra Damage Roll

- **Decision**: After a successful hit, a separate crit chance roll may add bonus damage. Uses Physical Crit or Magic Crit derived stats.
- **Rationale**: Separate roll after the hit keeps crits exciting without adding variance to the miss/hit determination. Extra damage roll (not a multiplier) means crits are impactful but bounded.
- **Implications**: Awareness, Luck, and Accuracy/Intellect all contribute to crit chance via the derived stat formulas. Crit-focused builds are viable but not dominant.

### Zone-Based Spatial System

- **Decision**: Discrete zones with three range bands (Short/Medium/Long), not grid-based or hex-based. Range is binary — in-range or not, with no damage falloff.
- **Rationale**: Simple enough for auto-battler AI, deep enough for positioning to matter. Zones are easy to represent in TUI. Binary range avoids complex distance math.
- **Implications**: All spatial abilities must work in terms of zones and ranges, not absolute positions or distances.

### Movement Costs Full Turn

- **Decision**: Move is a full action — no free movement by default.
- **Rationale**: Creates positioning tension. Ranged characters have a genuine advantage. "Move + attack" Perks become high-value.
- **Implications**: Melee-only characters need gap-closing Perks or forced movement to compete with ranged. AI must factor movement cost into decisions.

### Initiative: Sqrt Speed × Global Multiplier

- **Decision**: Initiative gain per tick = `sqrt(Speed) × global_initiative_multiplier`, acting at threshold ≥ 100. Global multiplier defaults to ~3.0.
- **Rationale**: Sqrt provides diminishing returns on Speed stacking. Global multiplier is a match-pacing knob independent of character balance — adjusting it speeds up or slows down combat without affecting relative character power.
- **Implications**: At default multiplier 3.0, a character with Speed 100 gains ~30 Initiative per tick → acts roughly every 3–4 ticks. Match length target of 25–150 ticks means most characters act 7–50 times per fight depending on fight type. Speed investment has clear but limited returns. The global Initiative multiplier can be varied per event type to control fight length.
- **Alternatives considered**: Pure `sqrt(Speed)` without multiplier (rejected — too few actions per character per fight), logarithmic scaling (rejected — sqrt provides sufficient diminishing returns).

### Action Speed as Flat Modifier

- **Decision**: Action Speed is a flat modifier to the base −100 Initiative cost after acting. Positive = faster recovery, negative = longer delay.
- **Rationale**: Simple and transparent. Players can easily reason about "this ability costs 70 Initiative instead of 100." Flat modifier avoids percentage-on-percentage complexity.
- **Implications**: Very fast actions (e.g., +50 Action Speed = only −50 Initiative) enable rapid-fire playstyles. Very slow actions (e.g., −50 Action Speed = −150 Initiative) can be balanced with high power.

### Tick Resolution: Effects → Initiative → Turns

- **Decision**: Each tick resolves in fixed order: per-tick effects first, then Initiative accumulation, then turns.
- **Rationale**: Effects-first means DoTs and status decay happen predictably before anyone acts. Initiative-then-turns means all characters accumulate Initiative before any turns resolve, preventing "first mover" issues within a tick.
- **Implications**: A DoT can kill a character before they get their turn on that tick. Status decay happens at a predictable point in the tick cycle.

### Status Before Damage (Type-Ordered Resolution)

- **Decision**: Within a single Action's effect components, status/buff/debuff effects apply before damage effects, then movement effects last. This overrides the "simultaneous" model from traits-and-perks.
- **Rationale**: Enables powerful combo Actions (apply armor shred, then deal damage against reduced armor). Creates interesting design space for single-Action debuff+damage patterns.
- **Implications**: A single Action CAN have an ApplyStatus that reduces Soak followed by a Damage component that benefits from the reduction. Content authors can design "setup-and-exploit" Actions. Traits-and-perks spec updated to reference combat for authoritative resolution order.

### 10 Damage Types, No Inherent Advantages

- **Decision**: 10 damage types across 3 families (Physical, Elemental, Magical) with no inherent rock-paper-scissors. All 10 available in MVP.
- **Rationale**: Advantages come from Perk/equipment interactions and status affiliations, not from hardcoded type matchups. Enables emergent meta via content, not systemic rules.
- **Implications**: All 10 types need at least one Trait and representative Perks in MVP content. Resistance is per-type.

### Resistance: Automatic with Dual Reduction

- **Decision**: Status effects always apply (no binary resist roll). Resistance reduces both the number of stacks applied and damage of that type.
- **Rationale**: "Always applies" means status effects are reliable offensive tools — you never "waste" an ability because of a resist roll. Resistance provides gradual mitigation rather than all-or-nothing, making both attacker investment and defender resistance always meaningful.
- **Implications**: High resistance reduces but never eliminates status effects. Stacking resistance to a specific type is a meaningful defensive investment. Balance must tune resistance curves to prevent either "status immunity" or "resistance uselessness."

### Haste/Slow: Dual Channel

- **Decision**: Speed modification (permanent stat changes) and Initiative gain modification (temporary multiplier) are separate channels that stack.
- **Rationale**: Two channels provide more design space — a Perk can grant temporary haste (Initiative gain buff) without permanently changing Speed. Equipment can modify base Speed. The distinction matters for stacking and dispel interactions.
- **Implications**: "Slow" statuses might reduce Initiative gain (temporary, dispellable) while injuries might reduce Speed (permanent until healed). Both pathways exist for content design.

### Stealth: AoE Hits, Damage Breaks, Stealth-Tagged Actions

- **Decision**: AoE hits Sneaking characters. Taking damage has a chance to break Sneaking. Non-stealth Actions break Sneaking; stealth-tagged Actions do not.
- **Rationale**: AoE as a stealth counter creates tactical counterplay without making stealth useless. Damage-break-chance means even indirect hits threaten stealth. Stealth-tagged Actions enable "assassin" builds that can strike from stealth at the cost of limiting their Action pool.
- **Implications**: Stealth builds need investment in damage avoidance (positioning, allies as shields). AoE abilities have anti-stealth utility beyond raw damage. Stealth-tagged Actions are a distinct tag that content authors can apply to specific Perks.

### Detection: Passive + Active

- **Decision**: Passive detection is automatic and same-zone only. Active detection (Search) is a full-turn Action that searches the entire map with range-based penalties.
- **Rationale**: Passive detection means Sneaking characters in the same zone as high-Awareness enemies are at risk. Active Search provides a dedicated counter-stealth Action at the cost of a full turn — meaningful trade-off.
- **Implications**: AI must decide when to spend a turn on Search vs. other actions. Awareness stat gains utility as the passive detection stat. Search is available via the Combatant system Trait (default Action).

### Unified Stack-Based Status System

- **Decision**: All temporary effects use stacks with per-status decay rules. No separate subsystems for buffs vs. debuffs vs. DoTs vs. CC.
- **Rationale**: Consistent, extensible. One implementation, infinite content variety.
- **Implications**: Every new status effect is just a data definition (per-stack effect + decay model), not new code.

### No Passive In-Combat Healing

- **Decision**: No natural Health regeneration during combat. Stamina has slow passive regen.
- **Rationale**: Makes healing a build investment. Teams must commit resources (Perk slots, consumable slots, equipment affixes) to sustain.
- **Implications**: Matches trend toward attrition without healing investment. Match length tuning (25–150 ticks) accounts for no passive healing — longer fights especially reward healing investment.

### No Default Regen for Trait-Granted Resources

- **Decision**: Trait-granted resource pools (Mana, Faith, Spirit, Focus, custom) have no default in-combat regeneration. Regen comes from Perks and Triggers.
- **Rationale**: Makes resource management a build investment. A Pyromancer who invests in Mana regen Perks has a genuine advantage over one who doesn't. Perks that grant a resource pool often bundle a regen Trigger as part of the same Perk or its Trait tree.
- **Implications**: Characters without regen Perks will deplete their resource pools during combat — this is intended. Resource conservation (using cheaper Actions) becomes a meaningful AI/build decision. Trait tree design should include regen options for the resources they introduce.

### Fallen Revival via Perks/Consumables

- **Decision**: Fallen characters can be revived mid-combat via Perks, consumables, or rare equipment effects.
- **Rationale**: Revival creates strategic depth — investing in revival capabilities is a meaningful team composition choice. Creates dramatic comeback moments. Consistent with Perks-grant-all-capabilities design philosophy.
- **Implications**: Revive is an effect component type in the system. AI must evaluate revival priority. Revival at fractional HP means revived characters are vulnerable.

### Overkill Affects Injury Severity

- **Decision**: Excess damage beyond 0 HP (overkill) is tracked and increases injury severity in the post-combat injury roll.
- **Rationale**: Creates meaningful stakes for getting hit hard. A character barely knocked out vs. one obliterated have different injury expectations. Encourages protecting characters from massive single hits.
- **Implications**: Equipment and Perks that reduce spike damage (shields, damage caps) have indirect injury-prevention value. Tournament strategy must consider opponent's burst damage potential.

### Tiered Injury Checks

- **Decision**: Post-combat injury uses two rolls: first determines if injury occurs, second determines severity. Both are modified by overkill, Endurance, Luck, and Perks.
- **Rationale**: Two-tier system creates more nuanced outcomes than a single roll. Most Falls result in no injury (minor events). When injuries do occur, the severity spectrum (minor → major → critical → death) provides dramatic range.
- **Implications**: Characters built for durability (high Endurance, Luck, injury-resistance Perks) have a meta-advantage beyond just surviving combat.

### Phased Post-Combat Presentation

- **Decision**: Post-combat flow is a sequential dramatic presentation: combat resolution → injury/death → Perk discovery → recruitment → loot.
- **Rationale**: Sequential presentation creates narrative moments and gives each phase appropriate attention. Bulk summaries feel rushed and don't generate player attachment.
- **Implications**: UI must support phased presentation. Each phase can be expanded with detail as the game matures.

### Hidden Combatant System Trait

- **Decision**: Default Actions (Attack, Defend, Move, Search) are provided by a hidden, immutable "Combatant" system Trait at level 1 on all characters.
- **Rationale**: Keeps all Actions in the Perk-Action framework — no special-case "basic attack" code. Hidden Trait doesn't occupy a Trait Slot or appear in UI. Extensible: future phases can add more default capabilities to this Trait.
- **Implications**: Data model must support a hidden Trait type. The Combatant Perk's Actions go through the same resolution pipeline as all Perk Actions.

### Binary Range (No Falloff)

- **Decision**: Abilities are either in-range (full effect) or out-of-range (cannot use). No damage falloff for non-optimal range.
- **Rationale**: Binary range is simplest for both implementation and AI reasoning. Zone-based spatial system already provides meaningful range dynamics through movement costs.
- **Implications**: An archer at Long range deals the same damage as one at Short range (if the Action allows both ranges). Range advantages come from not needing to move, not from damage bonuses.

### Target Match Length: 25–150 Ticks

- **Decision**: Individual fights should last 25–150 ticks. The range reflects fight variety: short PvE fodder encounters (~25 ticks), standard tournament matches (~50–80 ticks), and epic championship finals (~100–150 ticks). Adjusted via the global Initiative multiplier per event type.
- **Rationale**: The original 15–25 tick target was too narrow for the game's fight variety. Short PvE encounters need to feel fast; championship finals need enough ticks for strategic depth, resource management, and dramatic reversals. The wider range gives each fight type its own pacing identity.
- **Implications**: Balance must account for the full range. Cooldowns, resource pools, and status durations should be tuned for the middle of the range (~60 ticks) with acceptable behavior at both extremes. A 5-tick cooldown fires ~5–30 times per fight; a 20-tick cooldown fires 1–7 times. Tournament pacing assumptions must account for longer individual fights.

### MVP Scope: All Types + Default Map

- **Decision**: MVP includes all 10 damage types and the default 5-zone map. Zone variants and gimmicks are Phase 4.
- **Rationale**: All damage types need to exist for the Trait/Perk system to function properly — Traits are themed around damage types. The default 5-zone map is sufficient for validating spatial mechanics.
- **Implications**: MVP content must include at least one Trait per damage type. Zone variants are deferred content, not system work.

### Channeled Abilities and Interrupts: Deferred

- **Decision**: Channeled abilities (multi-turn Actions) and interrupt mechanics are deferred to Phase 2+.
- **Rationale**: The core Action model (one Action per turn, variable Action Speed) provides enough depth for MVP. Channeled abilities add significant system complexity (partial completion, interrupt handling, AI evaluation).
- **Implications**: No multi-turn Actions in MVP. The Action Speed system provides "fast" and "slow" ability pacing without multi-turn commitment.

### Formula-Based Scaling Multiplier Defaults

- **Decision**: Per-stat scaling multipliers are derived from desired gameplay ranges rather than hand-tuned individually.
- **Rationale**: Starting from target ranges (e.g., "Health 400–800 for a typical 3★") and back-calculating multipliers ensures consistent balance foundations. Individual hand-tuning can refine from there.
- **Implications**: Balance work starts with defining target ranges for all derived stats, then computing multipliers. Multipliers remain tuning values that can be adjusted during playtesting.

### Attrition Ramp — Anti-Stalemate Mechanic

- **Decision**: After a configurable onset tick (~100), a stacking global damage bonus (+0.5–1%/tick) applies to all combatants. Healing is unaffected. Onset tick and escalation rate are per-event-type (part of Context Flags). No hard tick limit.
- **Rationale**: Eliminates stalemates without a hard time limit. Fights that resolve naturally never trigger the ramp. Healing-heavy / double-tank compositions are gradually pushed toward resolution. The ramp onset tick doubles as the "expected fight length" signal for AI fight phase detection.
- **Implications**: Attrition ramp parameters must be defined per event type. AI `resource_efficiency` Scorer uses ramp onset as `estimated_max_ticks` for fight phase detection. Balance must ensure the ramp rate is aggressive enough to prevent true stalemates but gentle enough to allow come-from-behind moments in the attrition phase.

### Shield Mechanics — Second Defense Layer

- **Decision**: Shields absorb damage after Soak reduction (second defense layer). Penetration doesn't affect shields. Multiple shields stack; damage drains the oldest first (FIFO). Each shield has its own HP pool and optional damage type filter.
- **Rationale**: Provides a proactive defensive tool distinct from passive Soak. Oldest-first drain means refreshing shields adds HP rather than overwriting. Type filters enable specialized shields (fire ward, poison ward) without making generic shields obsolete.
- **Implications**: Combat pipeline gains a shield absorption step between Soak reduction and Health damage. Shield effect component already exists in the effect type catalog. AI must factor shield HP into target vulnerability assessments.
- **Alternatives considered**: Shields reducing before Soak (rejected — would make Penetration affect shields, blurring the two defense layers), newest-first drain (rejected — would incentivize spamming small shields to protect large ones, creating degenerate play patterns).

### Combat Scoreboard

- **Decision**: Structured per-character stat tracking: damage dealt/taken/healed, kills, Actions used by type, Fallen events, revival events, ticks survived. Available to post-combat spec and meta-balance system.
- **Rationale**: Provides the data needed for post-combat Perk Discovery (determining which Traits were "active"), meta-balance tracking, and player-facing fight statistics. Tracked transparently without affecting combat mechanics.
- **Implications**: Data model must support per-character stat accumulation during combat. Post-combat spec uses scoreboard data for discovery eligibility. Meta-balance spec uses it for win/loss tracking.

### Fight Phase Signal

- **Decision**: Combat exposes `current_tick / estimated_max_ticks` where `estimated_max_ticks` = attrition ramp onset tick. This provides a 0.0–1.0+ ratio for fight phase detection.
- **Rationale**: Gives the AI and Trigger system a clean, normalized signal for fight phase without hardcoding tick thresholds. Values < 1.0 are normal combat; ≥ 1.0 means attrition phase is active.
- **Implications**: AI `resource_efficiency` Scorer uses this for conservation strategy. Triggers can reference fight_phase for phase-dependent effects. The signal is derived from Context Flags (attrition ramp onset), not a separate configuration.

### Win Condition — Elimination Only

- **Decision**: Combat ends when one team is eliminated (last team standing). No timer, no point system, no draw conditions.
- **Rationale**: Simplest model. The attrition ramp ensures fights always converge to a conclusion, eliminating the need for a timer or draw condition. Elimination is the most dramatic and satisfying conclusion for spectators.
- **Implications**: No draw handling needed. Fight length is controlled by the attrition ramp onset and escalation rate, not by a hard timer.

### Global Initiative Order (No Team Alternating)

- **Decision**: Turn order within a tick follows global Initiative order, interleaved between teams. Characters from both teams act in Initiative order — there is no team-alternating structure.
- **Rationale**: Global ordering is simpler and more granular. Speed investment directly determines when you act relative to all combatants, not just your team. Team-alternating would add complexity without strategic depth in an auto-battler.
- **Implications**: A team with multiple high-Speed characters can act consecutively within a tick. Speed stacking has clear individual benefits (act sooner) but no team-turn-order manipulation.

### Percentage-Based Stamina Regen

- **Decision**: Stamina regenerates at X% of max pool per tick (tuning value), not a flat amount.
- **Rationale**: Percentage-based scaling means characters with larger Stamina pools (from Endurance/Willpower investment) regenerate proportionally more, making Stamina investment doubly valuable (more capacity AND more regen). Flat regen would make pool size investment feel wasted.
- **Implications**: Balance must tune the percentage alongside Stamina pool scaling multipliers. The Defend action's Stamina burst is also percentage-based (Y% of max pool).

### Defend — Fixed % Boost + Stamina Burst

- **Decision**: The Defend action provides a fixed percentage boost to Defense and Soak until the character's next turn, plus a Stamina recovery burst equal to Y% of max Stamina pool. Both percentages are tuning values.
- **Rationale**: Percentage-based Defense/Soak boost scales naturally with the character's existing defenses — a tank gets more absolute benefit from Defend than a glass cannon, which is intuitive. Percentage Stamina burst ensures meaningful recovery regardless of pool size.
- **Implications**: AI `self_hp_critical` and `resource_efficiency` Scorers factor Defend's defensive and Stamina benefits. The Combatant system Trait's Defend Perk uses these percentages as base values.

### Exhaustion — 1:1 Health Cost

- **Decision**: When Stamina = 0, physical Actions cost Health at a 1:1 ratio (no penalty multiplier). A 30-Stamina Action costs 30 Health instead.
- **Rationale**: Simple and transparent. No multiplier avoids the complexity of "is it worth paying 1.5× or 2× Health?" — the decision is clear: "do I spend Health to keep fighting?" The 1:1 ratio makes Health the true last-stand resource.
- **Implications**: Characters with high Health pools have more exhaustion runway. AI must factor Health cost into Action evaluation when Stamina is depleted. No special multiplier tracking needed.

### Forced Movement — Unrestricted by Default

- **Decision**: Forced movement is unrestricted (no resistance roll) by default. Fires `OnForcedMove` Triggers. Resistance Perks can reduce or prevent forced movement.
- **Rationale**: Unrestricted default makes forced movement a reliable tactical tool. Resistance as a Perk investment creates build choices for characters who want to hold position (tanks, zone controllers). If forced movement were resisted by default, it would be unreliable and undervalued.
- **Implications**: Content can create resistance Perks ("Immovable Stance", "Rooted") as meaningful investments. `OnForcedMove` Triggers enable reactive abilities (counter-push, damage on forced move). AI must account for forced movement reliability when evaluating push/pull Actions.

### Zone Effects — Deferred to Phase 4

- **Decision**: Zone-targeted persistent effects (fire zones, healing zones, etc.) are explicitly deferred to Phase 4. MVP combat uses zones for positioning and range only.
- **Rationale**: Zone effects add significant system complexity (duration tracking, zone state, interaction with movement) without being essential to the core combat loop. The spatial system provides enough tactical depth from positioning and range alone.
- **Implications**: No persistent zone state in MVP. Zone gimmicks and zone-targeted Actions are Phase 4 content.

### Context Flags Schema Owned by Combat

- **Decision**: The canonical Context Flags field list is defined in combat.md. The AI system (combat-ai.md) references combat.md for the schema and consumes flags read-only.
- **Rationale**: The combat system assembles the context flags from event configuration — it is the natural owner of the schema. The AI system is a consumer, not a definer. Centralizing the schema in one place prevents divergence.
- **Implications**: New context flags are added to combat.md's schema definition. Combat-ai.md's Context Flags section references combat.md. The schema includes attrition ramp parameters (`attrition_ramp_onset`, `attrition_ramp_rate`).

### Post-Combat Flow Moved to Separate Spec

- **Decision**: All post-combat resolution (injuries, Perk discovery, recruitment, loot) is moved to [post-combat](post-combat.md). Combat.md ends at victory/defeat declared and hands off scoreboard data, Fallen list, active Traits, and context flags.
- **Rationale**: Post-combat is a distinct gameplay phase with its own mechanics, UI flow, and spec dependencies (economy, roster-management, groups). Separating it reduces combat.md's scope and allows post-combat to evolve independently.
- **Implications**: Combat.md no longer owns injury checks, Perk discovery timing, recruitment, or loot. These are now in post-combat.md. Cross-references updated throughout.

### Perk Discovery — Single End-of-Combat Check

- **Decision**: Perk Discovery uses a single end-of-combat check per qualifying Trait, not per-Action/Trigger-use during combat. One discovery roll per Trait. Only Traits that had at least one Action used or Trigger fire qualify ("active Traits").
- **Rationale**: Per-use rolls during combat were implementation-complex (tracking every Action use for discovery) and created a bias toward high-Action-count builds. A single per-Trait roll at end-of-combat is simpler, fairer, and creates a cleaner handoff to the post-combat phase.
- **Implications**: Discovery probability per Trait per fight is higher than the old per-use base rate (retuned to produce similar per-fight discovery rates). The Combat Scoreboard tracks which Actions/Triggers were used, enabling the "active Traits" determination. Traits-and-perks.md updated to reflect the per-Trait model. Full resolution mechanics in [post-combat](post-combat.md).

### Multi-Team Support with Ranked Placement

- **Decision**: Combat supports 2+ teams of any sizes. "Enemy" = anyone not on your team. Teams are ranked by elimination order (last standing = 1st, last eliminated = 2nd, etc.). Teams eliminated on the same tick share placement rank (next rank skipped).
- **Rationale**: Enables free-for-all, multi-faction, and asymmetric scenarios. Per-team elimination order supports ranked multi-team tournament brackets. Tied placement handles the simultaneous elimination edge case cleanly.
- **Implications**: AI targeting must handle "enemy = not my team" with potentially multiple enemy teams. Tournament formats can use multi-team brackets. Context Flags should include team count. The data model needs per-team elimination tracking.

### Event-Type-Defined Starting Positions

- **Decision**: Starting positions are configured per event type via per-team zone lists (e.g., Team A: [North], Team B: [South]). Characters are distributed within their team's allowed zones by the system.
- **Rationale**: Different event types need different starting configurations — tournaments use opposing zones, PvE encounters might use ambush positioning, free-for-all spreads teams across all zones.
- **Implications**: Event template data model includes starting zone assignments per team. System handles character distribution within allowed zones.

### One Turn Per Tick

- **Decision**: Each character acts at most once per tick. Excess Initiative carries over to the next tick. Tick is the atomic time unit of the combat simulation.
- **Rationale**: Prevents runaway multi-action chains from extreme Speed investment. Very fast characters act in consecutive ticks before others get a turn — appearing to chain actions — but each action occurs in a separate tick. Speed investment is rewarded with frequency, not burst.
- **Implications**: No need for per-tick action limits or anti-infinite-loop safeguards. Initiative overflow naturally balances itself. Balance can focus on per-action effectiveness without worrying about action multiplication.

### Full-Tick Death Deferral and Dying Blows

- **Decision**: When a character's HP drops below 0 (from any source — DoTs, Actions, etc.), they are NOT immediately removed from play. They finish the full tick: remaining effects resolve, they accumulate Initiative, and they can take their turn. Fallen state applies at end of tick. Actions taken while at ≤0 HP are **Dying Blows** — explicitly tracked and available as a Trigger condition (`OnDyingBlow`).
- **Rationale**: Full-tick deferral creates dramatic moments — a character reduced to 0 HP by an attack can still take their turn and potentially eliminate their killer with a Dying Blow. The Dying Blow mechanic rewards Perks that trigger on the edge of death ("Last Stand", "Dying Breath").
- **Implications**: Dying Blows tracked in Combat Scoreboard. `OnDyingBlow` added to Trigger event vocabulary. Content can create "Last Stand" Perks (+100% damage on Dying Blows). AI does not need special death-checking mid-tick — all characters act normally within a tick.

### Starting Initiative: Default Zero with Offset Modifiers

- **Decision**: All characters begin combat with Initiative = 0 by default. Surprise conditions (event-configured) and Perks/affixes can grant an initial offset — either flat (+10) or scaled (+sqrt(Speed) × multiplier = one tick's head start).
- **Rationale**: Default 0 is deterministic and predictable. Offset modifiers create meaningful investment opportunities (a "Quick Draw" Perk that grants starting Initiative) and event-type flavor (ambush events give one team a head start).
- **Implications**: Initiative offset is a new Stat Adjustment target for Perks and equipment. Event templates can specify per-team Initiative offsets alongside starting zone assignments.

### Unified Combat Pipeline with Component-Driven Branching

- **Decision**: All Actions go through the same pipeline. The presence and types of effect components determine which steps apply — no explicit "action type" flag. Actions with enemy-targeted Damage components go through the full attack pipeline; heal/buff Actions skip the attack roll.
- **Rationale**: One code path reduces implementation complexity and bug surface. Component inspection is more flexible than explicit action typing — a mixed Action (damage + heal) naturally resolves the damage through the attack pipeline and the heal directly.
- **Implications**: No "action type" field in the Action data model. Pipeline code inspects effect components to determine flow. A single Action can have components that follow different pipeline paths.

### Per-Component Damage Resolution

- **Decision**: Each Damage effect component resolves through its own complete Soak + Shield + Resistance calculation independently, as if it were a separate hit. A "Flame Blade" with Slashing + Fire resolves each against its respective Soak family.
- **Rationale**: Rewards Actions with diverse damage types — each component hits a potentially different Soak family. Creates interesting build choices for multi-type attacks. Consistent with the per-type Soak system.
- **Implications**: Multi-damage Actions are mechanically stronger than equivalent single-type Actions of the same total damage (they test multiple Soak values). Content balance should account for this. Per-component resolution is more computationally expensive but bounded by component count per Action (typically 1-3).

### Per-Damage-Type Soak with Family Formulas

- **Decision**: Soak is per-damage-type with three family-level base formulas. Physical Soak (60% Endurance + 25% Might + 15% Speed), Elemental Soak (45% Endurance + 35% Awareness + 20% Luck), Magical Soak (55% Willpower + 25% Intellect + 20% Luck). Per-type bonuses via tag-scoped Stat Adjustments.
- **Rationale**: Per-type Soak creates deeper defensive build choices — a character can specialize in physical toughness vs magical resilience. Family formulas keep the formula count manageable (3 vs 10) while per-type bonuses from Perks/equipment add granularity. Each family has a distinct attribute identity.
- **Implications**: Characters have 3 base Soak values tracked. Tag-scoped bonuses add per-type refinement. The K constant in the Soak formula applies universally. Equipment and Perks that grant "+X Soak vs [Fire]" add to the Elemental Soak base for Fire damage specifically.

### Dual Defense: Physical vs Magic

- **Decision**: Two Defense values: Physical Defense (40% Speed + 35% Accuracy + 25% Awareness) and Magic Defense (60% Willpower + 25% Awareness + 15% Luck). Physical family attacks check Physical Defense; Elemental and Magical family attacks check Magic Defense.
- **Rationale**: Creates meaningful differentiation between martial and magical attackers. A character with high Speed (good Physical Defense) may be vulnerable to spells, and vice versa. Each Defense has a distinct attribute identity — Physical Defense is about evasion, Magic Defense is about mental fortitude.
- **Implications**: Physical Defense is a new derived stat (adds to the existing 16). The Action's primary damage type determines which Defense is checked. A "Flame Blade" with Slashing + Fire uses its primary type for the single hit/miss roll — both damage components benefit from the same result.
- **Alternatives considered**: Single universal Defense (original model — rejected because it didn't differentiate martial vs magical attackers), per-type Defense (rejected — 10 Defense values is too complex).

### Primary Damage Type on Actions

- **Decision**: Each Action with Damage components designates a primary damage type. This determines which Defense (Physical vs Magic) is checked in Step 1. All Damage components in the Action share the same hit/miss result.
- **Rationale**: One hit roll per Action keeps combat resolution fast. The primary type reflects the Action's "flavor" — a Flame Blade is primarily a physical attack enhanced with fire, so it checks Physical Defense. Content authors choose the primary type to match the Action's theme.
- **Implications**: Action data model gains a `primary_damage_type` field (required when Action has Damage components). Content authoring must specify primary type for mixed-damage Actions.

### Universal Penetration with Diminishing Returns

- **Decision**: Penetration is a single universal stat with **diminishing returns**: `Effective_Pen = Pen × K_pen / (Pen + K_pen)`. The effective value is then subtracted from Soak: `Effective Soak = Soak[family] - Effective_Pen` (minimum 0). Same harmonic formula shape as Soak and Resistance.
- **Rationale**: Flat Penetration without diminishing returns could completely zero out Soak at high values, creating a balance problem. Diminishing returns on Penetration match the DR on Soak, creating a symmetrical offensive-defensive investment curve. One Penetration value is simpler to track than per-type or per-family.
- **Implications**: Equipment affixes granting Penetration are valuable but with diminishing returns — the first 50 points of Penetration are much more impactful than going from 150 to 200. K_pen is a tuning constant that caps the maximum effective Penetration.
- **Alternatives considered**: Flat subtraction (rejected — allowed complete Soak bypass at high values), percentage-based Penetration (rejected — different semantic from the existing flat model).

### Universal Critical Hits

- **Decision**: Crits apply to ALL effect component types, not just Damage. A crit heal heals more, a crit shield is stronger, a crit buff applies more stacks. Uses the same Physical/Magic Crit stats.
- **Rationale**: Universal crit rewards crit investment for all build archetypes — healers, support, and tanks benefit from crit chance, not just damage dealers. Creates exciting moments for all roles.
- **Implications**: Crit stats (Awareness, Luck, Accuracy/Intellect) are valuable for every character archetype. Content balance must account for crit scaling on heals and shields. AI Scorers should factor crit chance into Action value assessment for all component types.

### Allies Skip Resistance

- **Decision**: Resistance applies only to enemy-sourced effects. Friendly buffs, heals, and beneficial status effects always apply at full strength.
- **Rationale**: Intuitive — your team's abilities help you, enemy abilities are resisted. Avoids the counterintuitive situation where a Fire-resistant character receives reduced benefit from a friendly fire buff.
- **Implications**: Effect source tracking needed (friendly vs enemy). No complex "beneficial vs harmful" tagging on status effects — the system checks source allegiance instead.

### Action-Defined Damage Formulas

- **Decision**: Each Damage effect component has its own `formula` field defining base damage. Weapon attacks reference the equipped weapon's damage; spell Actions define their own formula (e.g., "30 base + 40% Intellect"). This is already part of the effect component spec but now explicitly confirmed.
- **Rationale**: Action-defined formulas give content authors full control over damage scaling. A fire spell scales with Intellect; a physical attack scales with Might. No implicit assumptions about damage sources.
- **Implications**: The `formula` field on Damage effect components is the canonical source of base damage. Weapon damage is one possible input to a formula, not the only one.

### Deterministic Simulation with Seeded RNG

- **Decision**: Combat uses a seeded PRNG. Same seed + same inputs = identical replay. The seed is stored alongside the combat event log.
- **Rationale**: Enables exact fight replays, deterministic testing, bug reproduction, and verifiable results (tournament integrity, anti-cheat). Essential for the simulation-first presentation model.
- **Implications**: All random calls in combat must use the seeded PRNG (no system random). Seed is part of the combat record. Replays are byte-identical.

### Structured Combat Event Stream

- **Decision**: The combat engine emits structured typed events during simulation. Combat simulates the entire fight to completion, then presents results via dramatized summary and/or detailed event logs.
- **Rationale**: Simulation-first enables the MUD-style TUI to present combat in whatever format suits the player — quick summary, LLM-dramatized narrative, or tick-by-tick review. Structured events are the authoritative combat record, enabling replays, statistics, and presentation flexibility.
- **Implications**: Event type catalog is an implementation detail (not locked in spec). Combat engine produces events as output, not real-time rendering. TUI consumes events for presentation. Event logs + seed enable exact replay. LLM summarization is a presentation layer feature.

### Batch-Then-Triggers AoE Resolution

- **Decision**: AoE Actions resolve all targets through the pipeline simultaneously (batch), then all resulting Triggers fire together after the batch completes.
- **Rationale**: Clean separation between "what happens" (batch resolution) and "what it causes" (triggers). A target dying mid-AoE doesn't affect other targets. All OnKill triggers from a multi-kill AoE fire together, enabling consistent trigger stacking.
- **Implications**: AoE implementation resolves damage against all targets before checking triggers. "Blood Frenzy" (OnKill: +damage) would stack from all kills simultaneously. No ordering dependency between AoE targets.

### Weapon-Type-Defined Attack & Damage Formulas

- **Decision**: Each weapon type defines its own attribute blend formula for Attack Value and Damage Value. There are no fixed "Physical Attack" or "Magic Attack" derived stats. Spell Actions define their own formulas independently in their Damage component's `formula` field.
- **Rationale**: Content-driven formulas give maximum flexibility. A dagger uses Speed, a warhammer uses Might, a staff uses Intellect. Each weapon type has its own gameplay identity without hardcoded derived stats. Spells are fully independent of weapons.
- **Implications**: The combat system needs a formula evaluation engine that resolves weapon/spell formulas dynamically. Equipment spec must define per-weapon-type attribute formulas as content data. No fixed Attack derived stat in the characters spec.

### Zone Adjacency: Center + Ring Topology

- **Decision**: Default 5-zone map uses center + ring adjacency. Center is adjacent to all four cardinals. Cardinals are adjacent to ring neighbors (N↔E, E↔S, S↔W, W↔N). Opposite cardinals (N↔S, E↔W) are NOT adjacent = Long range.
- **Rationale**: Creates meaningful Long range on the default map — ranged characters can target across opposite sides. Center is a contested hub accessible to all. Ring adjacency provides multiple movement paths between any two zones.
- **Implications**: Long range Actions ARE useful on the default map (across N↔S or E↔W). Center is the most connected zone (adjacent to all 4), making it valuable territory. Movement from N to S requires 2 moves minimum (through C, E, or W).

### Attribute-Scaled Unarmed Combat

- **Decision**: Characters with no weapon equipped have a baseline unarmed attack (Blunt damage, Might-scaled). Functional but naturally weaker than any weapon. Perks can enhance unarmed combat (Monk-style builds).
- **Rationale**: Ensures every character can participate in combat even without equipment. Might-scaling makes physical prowess the natural unarmed damage source. Perk-enhanced unarmed creates a viable build archetype.
- **Implications**: The Combatant system Trait's Attack action must support both weapon-based and unarmed resolution. Unarmed Perks could grant new Attack actions with better formulas, tags, and effects.

### Context Flags: Team List for Multi-Team

- **Decision**: Replace `team_size`/`opponent_team_size` with `team_sizes: [int]` — a list of team sizes with own team first. AI derives all needed information from the list.
- **Rationale**: A list naturally supports 2+ teams. Own team first is a clean convention. AI can calculate total enemies, team count, outnumbered status, etc. from the list.
- **Implications**: Context Flags schema updated. AI Scorers that reference team size must use the list format.

### Formula Modifier Tags (Attribute Substitution)

- **Decision**: Perks can apply formula modifier tags — a Stat Adjustment subtype that substitutes one attribute for another in formula evaluation. When multiple modifiers target the same slot, the **highest attribute value wins** (player always benefits from their best option).
- **Rationale**: Formula modifiers enable "Intelligent Strikes" (Intellect for Accuracy) and "Charismatic Hitter" (Charisma for Might) without creating new Action variants. Highest-value-wins conflict resolution is simple and always benefits the player.
- **Implications**: Formula evaluation must check for active modifier tags and resolve conflicts. This is a new Stat Adjustment subtype alongside flat bonuses and tag-scoped bonuses. The data model needs a formula modifier representation.

### DoTs Bypass Soak

- **Decision**: DoT damage (from status effect ticks during the Effects Phase) bypasses Soak entirely and is applied directly to Health. Shields still absorb DoT damage. Resistance reduces stack count at application time.
- **Rationale**: DoTs already passed through Resistance when initially applied — double-gating through both Resistance and Soak would make them too weak. Shields proactively protecting against DoTs is a valid strategy and creates build choices.
- **Implications**: The Effects Phase damage path skips Soak calculation for status-sourced damage. Shield absorption still applies (Shields sit between Soak and Health). This makes Resistance the primary defense against DoTs, and Shields the proactive defense.
- **Alternatives considered**: DoTs through Soak (rejected — too much defensive gating, DoTs become irrelevant), DoTs bypass both Soak and Shields (rejected — no proactive defense against DoTs).

### Passive Detection Before Each Turn

- **Decision**: Passive stealth detection happens immediately before each character's turn in the Turns Phase, not once per tick. Characters act with the most current detection information.
- **Rationale**: More granular detection means an ally's turn that reveals a stealth enemy immediately benefits the next ally to act. Creates natural team coordination for stealth counterplay.
- **Implications**: Passive detection is not part of the Effects Phase — it's per-character in the Turns Phase. This adds a detection check before each turn but the check is simple (same-zone Awareness comparison).

### Summons as Ephemeral Combatants

- **Decision**: The Summon effect component creates Ephemeral Combatants with their own Initiative, preset AI, and team membership matching the summoner. They disappear when killed or when the summoner enters Fallen. They do not trigger post-combat phases.
- **Rationale**: Using the existing Ephemeral Combatant model (from characters spec) avoids special-case code for summons. Independent Initiative means summons act in global turn order like everyone else. Summoner-death-kills-summons prevents orphaned entities.
- **Implications**: Summon templates need to define stat blocks, AI configurations, and duration. Fallen resolution must check for summoner-linked entities. Post-combat explicitly excludes summons from injury/discovery/recruitment.

### Stun Prevents Initiative Gain

- **Decision**: Stun stacks prevent Initiative gain entirely (0 gain while stacks > 0). Stunned characters never reach Initiative ≥ 100, so they never get turns. Stun stacks decay proportionally to Willpower per tick.
- **Rationale**: Hard stun is the most impactful CC — losing turns entirely is devastating. Willpower-based decay provides a clear counter-stat and prevents permanent stun-lock. High Willpower characters shrug off stun quickly; low Willpower characters are vulnerable.
- **Implications**: Stun is the most powerful status effect — content must price it accordingly (high resistance, short duration, limited access). AI must factor stun vulnerability into target selection. Stun immunity Perks would be high-value defensive investments.

### Revival: Clean Slate

- **Decision**: All status effects are cleared on revival. A revived character starts fresh — no lingering debuffs, DoTs, or buffs from before they fell. Only the fractional HP remains.
- **Rationale**: Clean slate is simpler (no "which statuses survive?"), avoids punishing revived characters with pre-death debuffs, and creates a clear mental model: revival = fresh start at low HP.
- **Implications**: Revival timing matters less from a debuff perspective (no benefit to waiting for debuffs to expire before reviving). Buff-then-revive combos are not viable — the buff would be cleared. Content design should not rely on pre-Fallen status persisting through revival.

### Summon Control Capacity Budget

- **Decision**: Summoner Traits grant a Resource Pool acting as a control capacity budget (e.g., "Necro Control" = 0.8×Willpower + 0.2×Charisma). Each summon type has a control cost. Summoners maintain summons up to capacity; dying summons free capacity for new summons.
- **Rationale**: Budget-based control is more flexible than per-Perk caps. A summoner can choose many weak summons or few strong ones. Tying capacity to a Resource Pool means it scales with attributes and can be modified by Perks/equipment. Dying summons naturally free capacity, creating dynamic summoning decisions.
- **Implications**: Summoner Traits need a control capacity Resource Pool definition. Summon templates need a control cost field. AI must factor remaining capacity into summoning decisions. Balance must ensure capacity pools and summon costs produce reasonable summon counts (e.g., 2-5 active summons for a dedicated summoner).

### Mutual Elimination is a Draw

- **Decision**: When all remaining teams are eliminated on the same tick, all share 1st place (draw). Consistent with the existing same-tick shared rank rule.
- **Rationale**: The shared rank rule already handles same-tick elimination cleanly. Mutual elimination is just the extreme case where all teams eliminate each other simultaneously. No special handling needed — the general rule covers it.
- **Implications**: Tournament formats must handle draw outcomes (shared 1st). Post-combat flow must handle the case where no team "won" outright.

### Non-Stamina Resource Depletion: Action Unavailable

- **Decision**: Only Stamina has the Health substitution mechanic (Exhaustion). When any other resource pool (Mana, Faith, Spirit, Focus, etc.) is depleted, Actions requiring that resource are simply unavailable — the `resource_gate` vetoes them. No fallback or substitution.
- **Rationale**: Stamina's Health substitution creates dramatic last-stand moments for physical fighters. Extending this to all resources would eliminate resource management as a strategic concern. Mana depletion means a mage must fall back on default Actions or cheaper alternatives — that's a meaningful build constraint.
- **Implications**: AI `resource_efficiency` Scorer must conserve non-Stamina resources more aggressively (no safety net). Builds relying on a single non-Stamina resource need regen Perks or will hit a hard wall. Default Actions (Attack, Move, Defend, Search) remain available since they don't cost non-Stamina resources.

### Crit Bonus: Percentage of Base

- **Decision**: Crit bonus formula: `output = base × (1 + crit_multiplier)`. The `crit_multiplier` is a tuning value (e.g., 0.5 for +50% bonus on crit). This applies universally to all effect component types.
- **Rationale**: Percentage of base scales naturally with ability power — a strong ability crits for more than a weak one. Single multiplier tuning value is simple to balance. Avoids the variance of a separate die roll and the complexity of flat-per-component bonuses.
- **Implications**: Crit multiplier is a new tuning value. Perks/equipment that increase crit multiplier are powerful (scale with ability strength). Balance must ensure that crit builds with high-base-damage abilities don't become dominant.

### Revive Targeting: Fallen Target Tag

- **Decision**: A new `[Fallen]` target tag allows Actions to exclusively target Fallen allies (e.g., `[Fallen, Ally, Single]`). The Revive effect component itself works on any ally — on non-Fallen targets, the Revive component does nothing but other components still resolve. This enables hybrid heal+revive Actions.
- **Rationale**: Two complementary mechanisms: the `[Fallen]` tag for Revive-only Actions, and Revive-on-any-ally for hybrid Actions. A "Healing Light" Action with both Heal and Revive can target any ally — it heals the living and revives the Fallen. A "Resurrect" Action with only Revive uses `[Fallen, Ally, Single]` to restrict to Fallen targets.
- **Implications**: Target tag vocabulary gains `[Fallen]`. AI `revival_priority` Scorer evaluates both Fallen-only and hybrid heal+revive Actions. Content authors choose between dedicated and hybrid Revive Actions based on design intent.

### Revived Summoner: Summons Stay Dead

- **Decision**: If a summoner is revived mid-combat, their previously active summons remain dead. The summoner must re-summon using their restored control capacity.
- **Rationale**: Summons are removed during Fallen resolution (when the summoner falls). Revival is a fresh start — the summoner returns with clean slate (no statuses, no summons). Automatically restoring summons would be complex and potentially too powerful.
- **Implications**: Reviving a summoner is less immediately impactful than reviving a direct combatant (the summoner needs turns to rebuild their summon army). AI must factor re-summon cost into revival priority for summoner characters.

### Fallen Status Decay: Frozen

- **Decision**: Status effects do not tick down while a character is in the Fallen state. If a Fallen character were to be affected by any mechanism, their statuses remain at the stacks they had when they fell.
- **Rationale**: Frozen decay is simpler than continuing to process statuses on inactive characters. Since revival clears all statuses (clean slate), frozen decay's primary purpose is consistency — the system doesn't need to track or process status decay for Fallen characters.
- **Implications**: Minimal implementation impact — Fallen characters are simply skipped during the Effects Phase. Clean slate on revival makes frozen decay moot in practice, but the rule exists for completeness.

### Healing Saves from Fallen

- **Decision**: Fallen Resolution (step 4 of tick resolution) is a pure HP check. If a character was reduced below 0 HP during the tick but healing brought them back above 0 HP before step 4, they do not enter Fallen.
- **Rationale**: Pure HP check is the simplest rule — no tracking of "was this character ever at 0 HP?" Just check current HP at tick end. This means healing is a genuine life-saving tool, not just damage mitigation.
- **Implications**: Fast healers (high Initiative, acting before Fallen Resolution) can save allies who were hit earlier in the tick. Creates dramatic save moments. Self-healing on Dying Blow turns can prevent the character's own death.

### Resistance Formula: Percentage with Diminishing Returns

- **Decision**: Both stack reduction and damage reduction use the same formula: `value × (1 - resist_ratio)` where `resist_ratio = Resistance / (Resistance + K)`. Same K per type for both. Parallels the Soak formula shape.
- **Rationale**: Using the same diminishing returns curve as Soak creates system consistency — players learn one formula shape and understand both defense mechanisms. Per-type K allows tuning individual resistances without affecting others.
- **Implications**: K is a per-type tuning constant (or possibly a single global constant like Soak). Balance must tune K alongside stat ranges. Very high Resistance approaches but never reaches immunity. Minimum 1 stack always applies regardless of Resistance.

### Stealth Break: Damage/Stealth Ratio

- **Decision**: Stealth break chance on taking damage uses `break_chance = damage_taken / (stealth_stacks × K)` where K is a tuning constant.
- **Rationale**: Linear formula is transparent and easy to reason about. More stealth stacks = harder to break. More damage = more likely to break. K controls the overall sensitivity.
- **Implications**: Stealth break K is a new tuning value. High stealth investment (many stacks) provides genuine protection against chip damage. AoE that deals low damage per target is less likely to break stealth than a big single hit.

### Resistance Damage Reduction: Same Formula, Same K

- **Decision**: Damage reduction from Resistance uses the same `damage × (1 - resist_ratio)` formula and the same K as stack reduction for that type.
- **Rationale**: One K per type simplifies tuning. If Resistance 40 reduces applied stacks by 44%, it also reduces incoming damage of that type by 44%. Players can reason about resistance strength from a single number.
- **Implications**: No separate damage-reduction tuning per type — stack and damage reduction are always in lockstep. This means high Resistance provides comprehensive defense against a damage/status type.

### Minimum Stacks: Floor of 1

- **Decision**: Status effects always apply at least 1 stack, even after Resistance reduction. Resistance can reduce 10 stacks to 1 but never to 0.
- **Rationale**: Minimum 1 stack ensures status effects are reliable offensive tools. An attacker who lands a status-applying ability always gets *something* through, even against high-Resistance targets. This prevents Resistance stacking from creating effective immunity to statuses.
- **Implications**: Even maximally Resistance-stacked characters are affected by statuses (at reduced potency). Content balance can rely on statuses always having some effect. The 1-stack floor is a hard rule, not a tuning parameter.

### Attack Costs Stamina

- **Decision**: The default Attack action has a small Stamina cost (tuning value). Move, Defend, and Search are free (no resource cost).
- **Rationale**: A small Stamina cost on Attack creates Stamina pressure even for characters relying on the basic attack — they can't attack indefinitely without eventually triggering Exhaustion. This makes Defend (which recovers Stamina) a meaningful choice even for simple builds. Move, Defend, and Search being free ensures characters always have zero-cost options.
- **Implications**: Default Attack Stamina cost is a new tuning value. AI must factor this cost into Attack evaluation via `resource_efficiency`. Characters with very low Stamina pools will feel Exhaustion pressure faster from basic attacks alone.

### Formula Representation: Weighted List + Optional Expression

- **Decision**: Formulas use a structured `{base, weights}` format for the common case (~95% of formulas) and an optional `formula_override` expression string for complex Perks/spells.
- **Rationale**: Structured format is machine-inspectable — AI Scorers can read weights, the balance system can analyze formulas, and formula modifier tags can substitute attributes. Expression override provides an escape hatch for complex mechanics without compromising the simple common case.
- **Implications**: Formula evaluation engine must support both formats. Content authoring tools should default to the structured format and only expose expression override for advanced use cases. Modifier tags only apply to the structured format (expression overrides handle their own attribute references).

### DoT Overkill Counts

- **Decision**: Overkill = `abs(final_HP)` regardless of damage source. DoT damage, direct damage, environmental effects — all contribute to the overkill value used for post-combat injury severity.
- **Rationale**: Consistent rule — damage is damage regardless of source. A character who burns to death from DoTs should have the same injury risk as one killed by a sword strike of equal excess damage.
- **Implications**: DoT-heavy builds can generate overkill just like burst damage builds. Characters under heavy DoTs who are about to fall should be healed or shielded to reduce overkill, creating tactical depth.

### Self-Save via Dying Blow

- **Decision**: A character at ≤0 HP who takes their turn (Dying Blow) can use a self-heal Action. If the heal brings them above 0 HP before Fallen Resolution (step 4), they survive. Consistent rules — no special cases.
- **Rationale**: If Fallen Resolution is a pure HP check, then any healing that brings HP above 0 should prevent Fallen — including self-healing on a Dying Blow turn. Special-casing self-saves would add complexity for no gameplay benefit.
- **Implications**: Characters with self-heal abilities have a natural survival advantage on Dying Blow turns. AI must evaluate self-heal as a high-priority option during Dying Blow turns. Content can create "Second Wind" style Perks that are specifically designed for self-save scenarios.

### Formula Modifier Scope: Per-Category

- **Decision**: Each formula modifier tag specifies a **formula category scope** — which categories of formulas it applies to (attack, damage, defense, healing, etc.). A modifier can target multiple categories.
- **Rationale**: Per-category scoping prevents a single modifier from universally replacing an attribute across all formulas. "Intelligent Strikes" should make Intellect replace Accuracy in *attack* formulas, not in defense or healing formulas too. This is important for balance — universal attribute replacement would be too powerful.
- **Implications**: Formula modifier data model gains a `scope` field (list of formula categories). Content authors must specify which categories each modifier affects. The formula evaluation engine checks scope before applying modifiers.

### Fallen Resolution: Initiative Order

- **Decision**: Characters enter Fallen in **reverse Initiative order** (lowest Initiative first). `OnFallen` Triggers fire sequentially in this order.
- **Rationale**: Initiative order provides a deterministic, meaningful ordering for Fallen resolution. Lowest-Initiative characters fall first, which means higher-Initiative characters' Triggers fire last — potentially benefiting from the state changes caused by earlier Triggers. Sequential processing (not simultaneous) ensures clear cause-and-effect chains.
- **Implications**: High-Initiative characters who fall on the same tick get their Triggers processed after lower-Initiative characters. This creates a subtle advantage for Speed investment. OnFallen Trigger chains are deterministic and ordered.

### Revive Works on Any Ally

- **Decision**: The Revive effect component works on any ally target. On a non-Fallen target, Revive does nothing — but other effect components in the Action still resolve normally. This enables hybrid heal+revive Actions.
- **Rationale**: Allowing Revive on any target (with no-op on living allies) enables versatile hybrid Actions without requiring separate "heal" and "revive" versions. A single "Healing Light" Action with Heal + Revive components can serve both purposes. The `[Fallen]` target tag still exists for dedicated Revive-only Actions that should only be used on Fallen allies.
- **Implications**: AI must evaluate hybrid heal+revive Actions for both living and Fallen ally targets. Content authors can create versatile support Actions that combine healing and revival. The `[Fallen]` tag remains for Actions that should exclusively target Fallen allies.

### Numeric System: Ceiling Integer Rounding

- **Decision**: All combat values are integers. All fractional results use ceiling rounding (round up). Minimum 1 damage per component on successful hits. Minimum 1 stack on all status applications.
- **Rationale**: Ceiling rounding favors attackers — even tiny damage deals at least 1. Consistent with minimum 1 stack philosophy. Integer-only values simplify implementation and player reasoning.
- **Implications**: All formula evaluations produce integers via ceiling. Damage, healing, Initiative gain, status decay, Soak reduction — everything is integers.

### Block Chance in Combat Pipeline

- **Decision**: Block Chance from equipped shield items (Buckler, Kite Shield, Tower Shield) is integrated into the per-component damage resolution pipeline between damage modifiers and Soak reduction (step 5 of 9). Block applies to all damage types, splits proportionally across multi-component damage.
- **Rationale**: Block as a pre-Soak flat reduction makes it especially effective against weaker hits (removing a larger percentage of damage before Soak's diminishing returns apply). Proportional split prevents gaming by component ordering. All-type blocking is thematic — the shield physically intercepts the attack.
- **Implications**: Two separate "shield" mechanics in the system: Block Chance (from equipment shield items, flat reduction) and Shield HP (from Perks/consumables, absorption pool). Equipment spec defines Block Chance/Value per shield type; combat spec defines pipeline position.

### Off-Hand Bonus Attack as Sequential Second Pass

- **Decision**: When dual-wielding or using weapon+focus, the main Action resolves through the full pipeline, then the off-hand fires a generic Attack as a completely separate pipeline pass (own hit roll, own crit check, own Block, own Soak). Each implement's proc Triggers fire only on its own hits.
- **Rationale**: Two separate Actions is clean and consistent — no special cases in the pipeline. Per-implement Trigger firing creates interesting build choices. DW penalties (-15%/-35%) ensure the bonus attack doesn't make dual-wielding dominant.
- **Implications**: Every dual-wield turn produces two separate damage events in the combat log. AI must evaluate off-hand bonus value. Shields are more effective against dual-wielders (two Block checks per turn).

### Friendly Fire: Full Normal Pipeline

- **Decision**: When Actions with `[All]` targeting tags hit allies (friendly fire), the full combat pipeline applies — Defense roll, Block, Soak, Resistance, Shields. The "allies skip resistance" rule applies ONLY to beneficial effects (heals, buffs, positive statuses), not to friendly fire damage.
- **Rationale**: Without this rule, a character's own AoE would deal MORE damage to allies than to enemies (unresisted damage). The full pipeline prevents this design bug while keeping beneficial effects reliable.
- **Implications**: Effect source tracking needs to distinguish beneficial vs harmful effects, not just ally vs enemy. AI `aoe_net_value` Scorer accounts for friendly fire pipeline correctly.

### Additive Status Stacking with Formula-Based Decay

- **Decision**: New status applications add stacks to existing stacks (additive). No separate duration tracking — stacks ARE the duration. Each status type defines a per-tick decay formula (percentage, flat, or attribute-derived). Character status tracking is minimal: `{status_type, current_stacks}`.
- **Rationale**: Eliminates the complexity of duration timers. Percentage decay creates natural exponential falloff. Attribute-derived decay (e.g., Stun → Willpower) ties defensive stats to status recovery. Ceiling rounding ensures statuses always decay (last stack always clears).
- **Implications**: Status type definitions gain a `decay_formula` field using the standard formula representation. Content defines per-status decay behavior. Multiple sources of the same status stack naturally.

### Penetration Diminishing Returns

- **Decision**: Penetration uses the same harmonic diminishing returns formula as Soak: `Effective_Pen = Pen × K_pen / (Pen + K_pen)`. The effective value subtracts from Soak. Same formula shape everywhere in the system.
- **Rationale**: Flat Penetration without DR could completely zero out Soak at high values. Harmonic DR creates system consistency — one formula shape for Soak, Resistance, and Penetration.
- **Implications**: K_pen is a new tuning constant. Penetration affixes need rebalancing for DR. First 50 points of Penetration are much more impactful than going from 150 to 200.

### Defend Boosts All Defenses, No Tick Limit

- **Decision**: Defend boosts both Physical and Magic Defense, plus all 3 Soak families (Physical, Elemental, Magical). No maximum tick duration — persists until the character's next turn. Implemented as the "Defending" status (1 stack, special decay: clears on turn start).
- **Rationale**: A general defensive brace should protect against all attack types. The stun+Defend interaction (indefinite defense while stunned) is accepted as thematic — a braced character who gets stunned is harder to kill while helpless. Content balances stun duration.
- **Implications**: Defend is a status effect using the standard stack-based system. No special mechanic needed.

### Trigger Chaining with Recursion Guard

- **Decision**: Triggers can chain — a Trigger's effect can fire other Triggers, which can chain further. Each individual Trigger instance fires at most once per originating event (recursion guard).
- **Rationale**: Unlimited chaining enables complex "combo chain" Perks (OnHit → apply Burning → OnStatusApplied → bonus damage). The recursion guard prevents infinite loops without limiting creative content design.
- **Implications**: Implementation maintains a "fired set" per originating event. Content authors can create chain-enabling Perks. Performance: chain depth is bounded by the total number of unique Triggers on the affected characters.

### Summon Starting Initiative: Zero

- **Decision**: Summoned Ephemeral Combatants start with Initiative = 0 and accumulate from scratch.
- **Rationale**: Summoning is a long-term investment, not instant payoff. High-Speed summons (wolves, elementals) reach their first turn faster than slow summons (golems). Simple rule consistent with default starting Initiative.
- **Implications**: Summoner build evaluation must account for summon ramp-up time. AI factors Initiative ramp into summon decision timing.

### No True Status Immunity

- **Decision**: There are no immunity Perks. Minimum 1 stack always applies with no exceptions. Defense against powerful statuses comes from high Resistance (reduces to 1 stack) + fast decay (attribute-derived formula clears quickly). Functionally zero impact from 1 stack that decays in the same tick.
- **Rationale**: Removing the immunity concept simplifies the system. The combination of minimum-1-stack + fast-decay provides the same gameplay outcome (negligible status impact) without a separate binary immunity mechanic. No special cases in status application.
- **Implications**: Removes mentions of "Stun immunity Perks" from the spec. High-Willpower characters are effectively stun-proof through fast decay, not through a separate immunity gate.

### Hit Bonus: Post-Modifier Flat Addition

- **Decision**: The hit margin (roll − Defense) is added to damage AFTER crit multiplier and damage modifiers but BEFORE defensive reductions (Block, Soak, Shield). Hit bonus is NOT amplified by crit or buffs — it's raw accuracy reward. Defenses still reduce it.
- **Rationale**: Attack Value intentionally double-dips (hit chance AND damage scaling), creating high per-hit variance. This is the intended combat feel — a great attack roll is both accurate and powerful. Applying hit bonus post-crit prevents crit from amplifying accuracy reward.
- **Implications**: High Attack investment is very powerful. The 9-step per-component damage pipeline explicitly orders hit bonus at step 4.

### AoE: One Roll, Per-Target Defense

- **Decision**: For AoE Actions, the attacker rolls once (1 to Attack Value). Each target subtracts their own Defense from that single roll independently. A high roll hits everyone; a low roll may miss high-Defense targets but hit low-Defense ones. Hit bonus varies per target.
- **Rationale**: "One swing" is thematic and faster to resolve. Per-target Defense comparison means the same attack can hit some targets and miss others based on individual defenses — creates interesting AoE outcomes.
- **Implications**: One roll per AoE Action, not per target. Each target gets an independent hit/miss result and hit bonus. Multihit (from implement tags) follows the same rule.

### Might Mitigates Armor Speed Penalties

- **Decision**: Might reduces Medium/Heavy armor Speed penalties using the harmonic diminishing returns formula: `effective_penalty = base_penalty × (1 - Might/(Might+K_armor))`. K_armor is a tuning constant.
- **Rationale**: Gives Might a unique combat niche beyond weapon damage — it's the "I can wear heavy armor effectively" stat. Creates natural Might + Heavy Armor synergy. Thematic: strong fighters bear heavy gear; mages are pushed toward Light armor.
- **Implications**: Might description in characters spec needs updating. Equipment spec's armor Speed penalty section gains a cross-reference. K_armor is a new tuning constant. A high-Might warrior in full Heavy armor (base −30 Speed) might reduce the penalty to about −12.

### Equipment Combat Mechanics Cross-Reference

- **Decision**: Combat spec gains an "Equipment Combat Mechanics" cross-reference section listing all equipment-originated combat effects (DW penalties, armor Speed penalties, implement resolution, Versatile mode, Multihit, equipment combat lock, Reach/Ranged, Block, implement tags) with links to equipment spec for details.
- **Rationale**: Implementation teams can find all combat-relevant mechanics from one document without duplicating content. Equipment spec remains the authoritative source for equipment details; combat spec provides navigability.
- **Implications**: Combat spec is the "starting point" for combat implementation; equipment spec provides equipment-specific detail.

### Status Decay: Ceiling Rounding Shortens Durations (#121)

- **Decision**: Ceiling rounding applies to decay removal, making percentage-decay statuses shorter-lived than they appear. Burning 25% decay: `ceil(10×0.25)=3` removed → 10→7→4→2→1→0 (5 ticks, not 8).
- **Rationale**: Consistent with the global ceiling rounding rule. Ceiling on decay removal favors status recovery — statuses clear faster than naive calculation suggests. This is intentional: it balances the minimum-1-stack rule on application.
- **Implications**: Content authors must use the ceiling-corrected decay sequence when designing status durations. A "25% decay" status clears in roughly `ceil(log(stacks) / log(1/0.75))` ticks.

### Default Action Speeds: Non-Offensive Actions Are Faster (#122)

- **Decision**: Default Action Speeds: Attack = 0 (−100 Initiative), Move = +25 (−75), Defend = +25 (−75), Search = +50 (−50). Non-offensive actions are faster than Attack.
- **Rationale**: Non-offensive default Actions should be competitive with attacking. Search is deliberately cheapest — map-wide detection at low Initiative cost makes it a viable tactical option. Move and Defend as "quick" actions reward tactical decisions over pure aggression.
- **Implications**: AI scoring must account for variable Initiative costs when comparing default Actions. A character who Searches recovers Initiative 50% faster than one who Attacks.

### Hit Bonus: Primary Component Only (#123)

- **Decision**: The hit bonus (roll − Defense) is added as flat addition to the **primary damage type component only**. Secondary damage components in multi-component Actions do NOT receive the hit bonus.
- **Rationale**: Prevents multi-component Actions from double-dipping on Attack investment. A "Flame Blade" with Slashing (primary) + Fire gets the hit bonus only on the Slashing component. This keeps multi-component Actions balanced — their advantage is hitting multiple Soak families, not amplifying hit bonus across all components.
- **Implications**: The per-component damage pipeline step 4 must identify which component matches the Action's primary damage type. Content balance should consider that secondary components receive no hit bonus.

### Off-Hand Fires on Enemy-Targeted Actions Only (#124)

- **Decision**: Off-hand bonus attack fires when the main Action targets enemies (any enemy-targeting component — Damage, debuff). Does NOT fire for heals, self-buffs, Move, Defend, Search. A debuff-only Action targeting enemies DOES trigger off-hand.
- **Rationale**: Thematically, the off-hand swing is part of an aggressive act. Healing an ally while also swinging your off-hand weapon is not a coherent combat action. Debuffs targeting enemies ARE aggressive (poisoning someone while stabbing with the main hand).
- **Implications**: AI must distinguish offensive from non-offensive Actions for off-hand evaluation. Dual-wielding healers do NOT get bonus attacks when healing — only when using enemy-targeted Actions.

### Revive at Step 0 in Effect Resolution (#125)

- **Decision**: Revive components resolve at Step 0 (before all other effects) in the type-ordered effect resolution. Hybrid heal+revive Actions: Revive first (clean slate), then buffs (Step 1), then heals (Step 2).
- **Rationale**: Ensures hybrid Actions work naturally without special-casing. A "Healing Light" with Revive+Buff+Heal revives the character (clearing statuses), applies fresh buffs, then heals — the revived character gets full benefit of all components.
- **Implications**: Content authors can create reliable hybrid revive+buff Actions. The clean slate from Revive doesn't destroy the Action's own buff/heal components because they resolve after Revive.

### Deferred Kill Trigger Queue (#126)

- **Decision**: OnKill, OnFallen, and OnAllyFallen use a deferred trigger queue. Potential kills are recorded during Actions; confirmed at Step 4 (Fallen Resolution); triggers fire at Step 5 (Kill Triggers). Characters healed back above 0 HP before Step 4 do not generate kill triggers.
- **Rationale**: Immediate OnKill during Actions would fire for targets who might be healed back above 0 HP before Fallen Resolution — creating false kills. Deferral ensures kill triggers are accurate. Also creates a cleaner separation between Action-phase triggers (immediate) and kill-phase triggers (deferred).
- **Implications**: AI cannot rely on immediate OnKill feedback during the Turns Phase. Step 5 adds a new tick resolution phase. Recursion guard applies per-kill-event in Step 5.

### Remove OnOverkill — Merge into OnKill (#127)

- **Decision**: `OnOverkill` is removed from the trigger vocabulary. Overkill value data is available in the `OnKill` event payload. One fewer trigger type to maintain.
- **Rationale**: OnOverkill was a niche trigger that could be fully served by OnKill with overkill data in its payload. Content that wants overkill-dependent behavior can check the payload value. Reduces vocabulary size without losing functionality.
- **Implications**: All references to OnOverkill replaced with OnKill. Content using overkill-dependent triggers reads from the OnKill event payload.

### New Trigger Events: Revival, Defense, Healing, Dodge, Summon (#128)

- **Decision**: Seven new trigger events added: `OnRevive` (revival), `OnBlock` and `OnShieldBreak` (defense), `OnHeal` and `OnHealedBy` (healing), `OnDodge` (attack/defense), `OnSummon` (summon). One removed: `OnOverkill`.
- **Rationale**: Fills gaps in the trigger vocabulary that would block Perk content design. OnBlock enables shield-focused builds. OnHeal/OnHealedBy creates symmetric healing triggers (matching OnDamageDealt/OnDamageTaken). OnDodge is the defender's counterpart to OnMiss. OnRevive enables post-revival Perks. OnSummon enables summoner synergies.
- **Implications**: Traits-and-perks has 7 new trigger events available for Perk content design. AI considerations may need updating for new trigger evaluations. Total trigger vocabulary: 30+ events.

### Detection: Per-Turn Only, No Persistent State (#129)

- **Decision**: Detection is per-turn only — there is no persistent "this team has detected character X" state between turns. Before each character's turn, passive detection re-rolls. Active Search also has per-turn persistence only. Teammates must make their own detection checks.
- **Rationale**: Per-turn detection prevents a single high-Awareness character from permanently revealing all stealth enemies for the whole team. Each character must earn their own detection. This makes stealth more robust and Awareness investment more individually meaningful.
- **Implications**: No detection state tracking needed between turns (simpler implementation). AI stealth evaluation must account for per-character detection probability, not team-level detection. Search's value is per-character, not team-wide.

### DoTs Bypass Block (#130)

- **Decision**: DoTs bypass both Soak AND Block. DoT tick damage occurs in the Effects Phase, outside the per-component damage pipeline where Block operates. DoT damage path: DoT tick → Shield absorption → Health.
- **Rationale**: Block is a per-Action mechanic — the shield physically interposes against an incoming attack. DoTs are ongoing status effects, not discrete attacks. Block conceptually doesn't apply. This also simplifies the Effects Phase — DoT damage just goes straight to Shields/Health.
- **Implications**: DoT-focused builds bypass two layers of defense (Soak and Block) but are still gated by Resistance at application and Shields during ticks. Makes Resistance and Shields the primary DoT defenses.

### Summon Deaths Fire Full Triggers (#131)

- **Decision**: When summons are removed (killed or summoner enters Fallen), they fire full trigger events — OnFallen on the summon, OnAllyFallen on teammates. A summoner with 5 summons falling generates 5× OnAllyFallen.
- **Rationale**: Summons are combatants that participate fully in the trigger system. Excluding them from death triggers would create special-case code and prevent interesting summoner Perks (e.g., "Necro Empowerment: OnAllyFallen: gain stacks of Empowered"). The cascade potential is a feature, not a bug — it rewards summoner investment.
- **Implications**: Content design should account for summon death cascades. A 5-summon necromancer falling is a major trigger event (6 total Fallen events). Balance should consider this when designing OnAllyFallen Perks.

### Defend + Stun: Accepted Emergent Strategy (#132)

- **Decision**: The Defend+Stun interaction (indefinite defense via self-stun) is an accepted emergent strategy. A team deliberately stunning their own Defended tank creates a meaningful trade-off: costs the stunner's Action, deals friendly fire damage from the stun-applying ability, and the stunned tank can't act.
- **Rationale**: This interaction emerges naturally from the rules (Defending clears on turn start; stunned characters never get turns). Rather than patch it with a special case, it's accepted as an interesting tactical option with real costs. Content provides counters (anti-stun Perks, dispel mechanics, Defend-removing abilities).
- **Implications**: No rule changes needed. Content design should ensure adequate anti-stun and dispel options exist as counters. AI should be capable of evaluating self-stun strategies (likely very niche).

### Crowd Appeal: Deferred to Phase 4 (#133)

- **Decision**: Crowd Appeal (derived stat from characters spec) has no mechanical combat effect in MVP. It exists as a derived stat for future use in the Morale system (Phase 4).
- **Rationale**: Crowd Appeal feeds into the Morale system, which is entirely deferred. Including it as a stat now ensures the foundation exists, but there's no point in giving it combat mechanics before Morale is designed.
- **Implications**: Crowd Appeal can be calculated and displayed but has no combat impact. Phase 4 Morale design will define its mechanical role.

### Revival Clean Slate: True Clean Slate (#134)

- **Decision**: Revival clean slate clears ALL statuses — both beneficial and harmful. No "buff vs debuff" classification. This is a complete reset.
- **Rationale**: Binary buff/debuff classification would add complexity (what about neutral statuses? Mixed statuses?). "Clear everything" is the simplest, most predictable rule. Hybrid revive+buff Actions work naturally via type-ordered resolution: Revive (Step 0, clears all) → Status (Step 1, applies fresh buffs).
- **Implications**: Pre-fall buffs are lost on revival. Content cannot rely on buffs persisting through fall+revival cycles. Revive+buff Actions are the intended way to buff revived characters.

### Kill Triggers in Step 5 of Tick Resolution (#135)

- **Decision**: Add Step 5 "Kill Triggers" to tick resolution order, after Fallen Resolution (Step 4). Tick now has 5 steps: (1) Effects Phase, (2) Initiative Phase, (3) Turns Phase, (4) Fallen Resolution, (5) Kill Triggers.
- **Rationale**: Kill triggers need their own resolution step because they are deferred (not fired during Actions). Placing them after Fallen Resolution ensures only confirmed kills generate triggers. This is the natural home for the deferred kill trigger queue.
- **Implications**: Tick resolution gains a fifth step. Data model needs a deferred kill trigger queue that accumulates during the tick and drains in Step 5.

### Effects Phase: Internal Sub-Ordering (#136)

- **Decision**: The Effects Phase resolves in fixed internal order: (1) Decay — all stacks on all statuses decay, (2) Effects — all per-stack effects apply, (3) Shield Expiry — expired shields removed last. Per-character ordering: highest stack count first (tiebreaker: alphabetical). All characters process simultaneously.
- **Rationale**: Decay-before-effects means statuses apply at their post-decay stack count (not pre-decay). Shield expiry last means shields protect against DoTs on their final tick. Simultaneous cross-character processing prevents ordering dependencies.
- **Implications**: A 3-stack DoT that decays by 1 this tick deals damage at 2 stacks, not 3. Shields set to expire this tick still absorb DoT damage. Implementation must process all characters' decay before any character's effects.

### Step 5 Kill Triggers: AoE Batch Sharing (#137)

- **Decision**: AoE kills from the same Action share a batch in Step 5. Each kill within the batch is a separate originating event for the recursion guard.
- **Rationale**: Consistent with the batch-then-triggers model for AoE resolution. Multiple kills from one AoE should fire their kill triggers together (as a batch), matching how AoE Action triggers work. Separate originating events per kill ensures the recursion guard tracks each kill chain independently.
- **Implications**: A "Blood Frenzy" Perk (OnKill: +damage stack) would gain one stack per confirmed kill in the batch. An AoE that kills 3 enemies gives 3 OnKill events, each as a separate originating event.

### OnRevive: Immediate During Step 0 (#138)

- **Decision**: `OnRevive` fires during Step 0 of effect resolution as an immediate trigger on the revived character. Not deferred like kill triggers.
- **Rationale**: OnRevive needs to fire immediately so that Revive-triggered effects (e.g., "gain Shield on revive") can apply before other Action components resolve. Deferring it would break the type-ordered resolution model.
- **Implications**: Content can create "Phoenix" style Perks: OnRevive → gain Shield, gain buff, etc. These effects apply in the Revive step before other Action components.

### OnBlock and OnShieldBreak: Pipeline Triggers (#139)

- **Decision**: `OnBlock` fires at pipeline step 5 (Block check) and `OnShieldBreak` fires at pipeline step 7 (Shield absorption). Both join the Action's trigger collection for batch resolution.
- **Rationale**: These are combat pipeline events that naturally fire during their respective pipeline steps. Joining the Action's trigger collection is consistent with how other pipeline triggers (OnHit, OnCrit) work.
- **Implications**: Shield-focused builds can trigger effects on blocks ("Riposte: OnBlock → counter-attack") and shield breaks ("Desperate Shield: OnShieldBreak → gain Defending status").

### OnHeal/OnHealedBy: Symmetric Healing Triggers (#140)

- **Decision**: `OnHeal` (healer's perspective) and `OnHealedBy` (recipient's perspective) added as symmetric counterparts to OnDamageDealt/OnDamageTaken. Fire immediately during the Action.
- **Rationale**: The trigger vocabulary had damage triggers from both perspectives but no healing equivalents. Healing-focused builds need trigger events. OnHeal enables "Holy Fervor" (gain stacks on healing), OnHealedBy enables "Grateful Blessing" (buff when healed).
- **Implications**: Content design gains healing-triggered Perk options. AI Considerations should evaluate healing triggers when assessing heal Action value.

### OnDodge: Defender's Perspective on Miss (#141)

- **Decision**: `OnDodge` fires when this character successfully dodges an enemy attack (defender's perspective, attack roll < 0). Symmetric with `OnMiss` (attacker's perspective).
- **Rationale**: The vocabulary had OnMiss (attacker side) but no defender equivalent. High-evasion builds need a trigger for "dodge-and-counter" Perks (e.g., "Riposte: OnDodge → bonus attack").
- **Implications**: Content gains dodge-reactive Perks. Speed/Accuracy investment gains trigger synergy value beyond just hit/miss probability.

### OnSummon: Summoner Trigger (#142)

- **Decision**: `OnSummon` fires when this character creates a summon via the Summon effect component. Fires immediately during the Action.
- **Rationale**: Summoner builds need a trigger for summon-related synergies (e.g., "Pack Tactics: OnSummon → buff all existing summons", "Overcharge: OnSummon → summon gains bonus stats").
- **Implications**: Content gains summoner-synergy Perks. AI should factor OnSummon triggers into summon Action evaluation.

### Trigger Timing Classification (#143)

- **Decision**: Triggers are classified into three timing categories: immediate (fire during Action resolution), revive (fire during Step 0), and deferred (fire in Step 5). This classification determines when effects from triggers can interact with other combat events.
- **Rationale**: Different trigger types need different timing to produce correct behavior. Kill triggers must be deferred (to avoid false kills from healed targets). Revive triggers must be immediate (to work within effect resolution). All others are immediate (standard batch-then-triggers model).
- **Implications**: Implementation needs three trigger dispatch paths. Content authors should understand timing when designing trigger-dependent Perks.

---

## Open Questions

Most original questions are resolved (143 decisions across 31 rounds). Remaining items are tuning values and deferred systems:

### Tuning Values
1. **K constant for Soak formula**: What value of K in `Soak[family]/(Soak[family]+K)` produces the desired damage reduction curve? Needs testing with target stat ranges.
2. **Global Initiative Multiplier exact default**: ~3.0 is the starting point; actual value depends on desired actions-per-character-per-fight.
3. ~~**Crit bonus formula**~~: **Resolved** — percentage of base: `output = base × (1 + crit_multiplier)`. Crit multiplier is a tuning value (e.g., 0.5 for +50%).
4. ~~**Stealth break-on-damage probability formula**~~: **Resolved** — `break_chance = damage_taken / (stealth_stacks × K)`. K is a tuning value.
5. **Revival Health fraction**: What percentage of max HP does a revived character return with?
6. ~~**Resistance reduction formulas**~~: **Resolved** — percentage with diminishing returns: `value × (1 - resist_ratio)` where `resist_ratio = Resistance / (Resistance + K)`. Same formula and K for both stack reduction and damage reduction per type.
7. **Judgment scaling multiplier and mapping functions**: Per-stat scaling multiplier for Judgment, and the functions mapping Judgment to tactical-vs-personality blend (0.3–0.95), selection sharpness (1–10), and lookahead depth (1–10 ticks).
8. **Stamina regen percentage**: What X% of max pool per tick produces good Stamina pacing?
9. **Defend boost percentages**: What fixed % boost to Defense/Soak and what Y% Stamina burst feels right for the Defend action?
10. **Attrition ramp defaults**: Default onset tick (~100), escalation rate (+0.5–1%/tick), and per-event-type overrides.
11. **Perk Discovery per-Trait rate**: What base rate per qualifying Trait produces similar per-fight discovery rates to the old per-use model?
12. **Per-family Soak scaling multipliers**: What scaling multipliers for the 3 Soak families produce desired damage reduction ranges at target attribute levels?
13. **Physical Defense scaling multiplier**: What scaling multiplier for Physical Defense produces the desired hit/miss rates?
14. **Default Attack Stamina cost**: What small Stamina cost for the default Attack action creates meaningful Stamina pressure without being prohibitive?
15. **Crit multiplier value**: What `crit_multiplier` (e.g., 0.5 for +50%) produces good crit impact?
16. **Stealth break K constant**: What K in `damage_taken / (stealth_stacks × K)` produces the desired stealth fragility?
17. **Resistance K constant(s)**: What K per resistance type in `Resistance / (Resistance + K)` produces desired stack/damage reduction curves?
18. **Summon control capacity multipliers**: What per-Trait capacity formulas and per-summon control costs produce desired summon counts (2-5 for dedicated summoners)?
19. **K_pen constant for Penetration DR**: What K_pen in `Pen × K_pen / (Pen + K_pen)` produces the desired Penetration curve?
20. **K_armor constant for Might armor mitigation**: What K_armor in `Might / (Might + K_armor)` produces desired armor penalty reduction?
21. **Block Chance/Value per shield type**: Buckler (~20%/low), Kite (~35%/med), Tower (~50%/high) — exact values TBD.
22. **Status decay rates per type**: Per-status decay percentages and attribute divisors (e.g., Burning 25%, Stun Willpower/10).
23. **Effects Phase tiebreaker for equal stack counts**: When multiple statuses on a character have the same stack count, the deterministic ordering (alphabetical by status name or definition order) needs to be confirmed at implementation time.

### Deferred Systems
12. **Morale**: Full design deferred to Phase 4.
13. **Channeled abilities and interrupts**: Deferred to Phase 2+.
14. **Zone variants and gimmicks**: Deferred to Phase 4.
15. **Zone effects**: Deferred to Phase 4.

### Moved to Other Specs
- **Injury roll probability tables**: Moved to [post-combat](post-combat.md) open questions.

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [combat-ai](combat-ai.md) | AI must understand zones, ranges, action options, and target selection across 2+ teams ("enemy" = not my team). AI must handle 20–40+ Actions per character efficiently. AI must evaluate multi-resource Action costs, tag-scoped Stat Adjustment bonuses (including multi-tag additive stacking), stealth/detection trade-offs (passive + active Search), revival priority, and resource conservation. Judgment derived stat (40% Awareness + 35% Willpower + 25% Intellect) controls AI decision quality via the Utility AI system (blend, sharpness, and lookahead depth). Combat log must record AI scoring data (top 3 Actions per turn with scores). Combat system passes context flags to AI at fight start (canonical schema defined here). Characters within a tick act sequentially in Initiative order with state updates between each. `resource_efficiency` Scorer uses fight phase signal (current_tick / attrition_ramp_onset) for conservation strategy. AI must factor shield HP, Block Chance, and per-type Soak into target vulnerability and damage output assessments. AI must evaluate primary damage type choice for mixed Actions (which Defense is weaker). Dying Blow Actions need AI handling (character at ≤0 HP still acts). AI must evaluate hybrid heal+revive Actions for both living and Fallen allies. AI must factor self-heal as a high-priority Dying Blow option. AI must evaluate summon control capacity when deciding to summon. Default Attack Stamina cost feeds into `resource_efficiency` Scorer. **Round 20**: AI must evaluate off-hand bonus attack value (sequential second pass). AI `aoe_net_value` Scorer must account for friendly fire going through full pipeline. AI must factor Might-based armor Speed mitigation into equipment evaluation. **Round 26**: Off-hand fires on debuff-only enemy Actions (AI scoring update). Kill triggers are deferred to Step 5 (AI can't rely on immediate OnKill during Actions). Detection is per-turn (AI stealth evaluation must be per-character, not persistent). 7 new trigger events for AI Consideration evaluation: OnRevive, OnBlock, OnShieldBreak, OnHeal, OnHealedBy, OnDodge, OnSummon. Variable Action Speeds on default Actions (Search +50, Move/Defend +25) affect AI cost-benefit. |
| [post-combat](post-combat.md) | Combat hands off to post-combat at victory/defeat. Provides: Combat Scoreboard (per-character stats including Dying Blows), Fallen list with overkill values, active Traits list (for Perk Discovery), Context Flags (exhibition status, event type), per-team elimination order/placement. Post-combat owns: injury/death checks, Perk Discovery resolution, recruitment, loot distribution. |
| [characters](characters.md) | 9 Attributes feed all combat calculations via derived stat formulas (including Judgment, Physical Defense, 3 Soak families). **New derived stats**: Physical Defense (40% Speed + 35% Accuracy + 25% Awareness), Physical Soak (60% Endurance + 25% Might + 15% Speed), Elemental Soak (45% Endurance + 35% Awareness + 20% Luck), Magical Soak (55% Willpower + 25% Intellect + 20% Luck). Anatomical slots determine weapon/armor options. Stamina exhaustion (0→Health drain at 1:1 ratio), percentage-based Stamina regen + Defend boost. Fallen sub-state mechanics with full-tick deferral. Post-combat outcomes handled by [post-combat](post-combat.md). **Round 20**: Might description updated — also mitigates armor Speed penalties via `Might/(Might+K_armor)` formula. |
| [traits-and-perks](traits-and-perks.md) | Perks provide Actions, Stat Adjustments, Triggers. Traits unlock resource types. Effect resolution order within Actions: Revive (Step 0) → status → damage → movement (overrides simultaneous model). All cooldowns reset per-combat. Perk Discovery changed to per-Trait end-of-combat check (active Traits only) — resolved in [post-combat](post-combat.md). Tag-scoped bonuses: flat or %, additive stacking, flat→% order. Trait/Perk level amplification multipliers shared curve ×1.0/×1.2/×1.4/×1.7/×2.0. Actions with Damage components need a `primary_damage_type` field. `OnDyingBlow` added to Trigger event vocabulary for "Last Stand" style Perks. **Round 20**: Trigger chaining enabled with recursion guard — content authors should be aware Triggers can fire further Triggers. Status immunity Perks → reframe as "high Resistance + fast attribute-decay" Perks. **Round 26**: OnOverkill removed from vocabulary. 7 new triggers available for Perk content design (OnRevive, OnBlock, OnShieldBreak, OnHeal, OnHealedBy, OnDodge, OnSummon). Revive at Step 0 enables reliable hybrid heal+revive Actions. Hit bonus is primary-component-only (affects multi-component Action design). |
| [equipment](equipment.md) | Weapons provide Attack/Damage values and damage type tags. Armor provides per-type Soak bonuses (Physical/Elemental/Magical families + per-type tags). Affixes can grant Penetration (universal, now with DR), crit bonuses, status riders, stealth-tagged effects, shield effects, starting Initiative offset. Equipment affixes use the tag system for synergies. **Round 20**: Block Chance position confirmed in combat pipeline (post-hit-bonus, pre-Soak). Off-hand bonus attack confirmed as sequential second pipeline pass. Penetration affixes need rebalancing for diminishing returns. Armor Speed penalty section needs cross-reference to Might mitigation formula. **Round 26**: No direct changes. Block clarified as not applying to DoTs (Block is a per-Action mechanic, not Effects Phase). |
| [consumables](consumables.md) | Consumables are usable during combat turns (potions, bombs, scrolls). Consumables can provide revival (Revive effect component) and shields (Shield effect component). AI must decide when to use them. |
| [tournaments](tournaments.md) | Tournament matches are combat instances. Multi-team formats supported with ranked elimination order. Attrition ramp onset and rate are per-event-type (specified in Context Flags). Per-combat cooldown and resource pool resets between rounds. Star-gated entry to manage power gaps. Post-combat phases (injury, discovery, recruitment, loot) handled by [post-combat](post-combat.md). Fight length varies by event type (25–150 ticks). Event templates define starting zone assignments per team. |
| [meta-balance](../architecture/meta-balance.md) | Combat Scoreboard data (win/loss per character, per Trait, per-character stats, Dying Blows) feeds the automatic underdog balancing system. |
| [data-model](../architecture/data-model.md) | Must support: hidden system Trait type, effect component typed format (type tag + key-value parameters), target tag vocabulary (including `[Fallen]`), Trigger event vocabulary (including OnDyingBlow), per-family Soak values (3 families), universal Penetration with DR (`Pen×K/(Pen+K)`), Physical + Magic Defense as derived stats, per-component damage resolution (9-step pipeline), overkill tracking per Fallen character, Combat Scoreboard per-character stat accumulation (including Dying Blows), Shield instances (HP pool, duration, type filter, FIFO ordering), Context Flags schema, attrition ramp state, event template schema (starting zones per team, Initiative offsets), seeded PRNG per combat, structured event stream/log storage, per-team elimination order tracking, `primary_damage_type` field on Actions. Summon control capacity Resource Pool per summoner Trait, per-summon control cost field, formula representation (`{base, weights}` + optional `formula_override`), formula modifier `scope` field (list of formula categories), Resistance K values per type, default Attack Stamina cost. **Round 20**: All combat values are integers (ceiling rounding). Status type definitions gain `decay_formula` field. Block Chance as pipeline step. Off-hand bonus attack as second Action. Trigger recursion guard per-event. Might armor mitigation formula. Penetration K_pen constant. **Round 26**: Effects Phase sub-ordering (decay→effects→shield expiry). Step 5 kill trigger queue with deferred entries. Revive Step 0 in resolution pipeline. Default Action Speed values (Search +50, Move/Defend +25). Per-turn detection state (no persistent detection tracking needed). 7 new trigger event types, OnOverkill removed. |

---

_Last updated: 2026-02-22 — Rounds 26-31 interrogation: 23 additional decisions. Hit bonus primary-component-only. Deferred OnKill trigger queue (Step 5). Kill Triggers as new tick resolution step. Effects Phase internal ordering (decay→effects→shield expiry). Default Action Speeds (Search +50, Move/Defend +25, Attack 0). Revive at Step 0 in effect resolution. Per-turn detection persistence (no persistent state). Trigger vocabulary expansion (7 new: OnRevive, OnBlock, OnShieldBreak, OnHeal, OnHealedBy, OnDodge, OnSummon; 1 removed: OnOverkill). DoTs bypass Block. Summon death triggers. Defend+Stun accepted strategy. Crowd Appeal deferred. True clean slate on revival. Total: 143 decisions across 31 rounds. Previous: rounds 20-25 (20 decisions), rounds 14-19 (18 decisions), rounds 10-13 (11 decisions), round 9 (24 decisions), round 8 (20 decisions), rounds 1-7 (27 decisions)._
