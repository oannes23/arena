# Combat — Domain Specification

**Status**: 🟢 Complete (updated 2026-02-18: rounds 10-13 — 11 additional decisions: weapon-type formulas, zone adjacency, DoTs bypass Soak, stun, summons)
**Last interrogated**: 2026-02-18
**Last verified**: —
**Depends on**: [characters](characters.md), [traits-and-perks](traits-and-perks.md)
**Depended on by**: [combat-ai](combat-ai.md), [post-combat](post-combat.md), [tournaments](tournaments.md)

---

## Overview

Combat is the core gameplay loop — automated tactical fights between teams of characters on zone-based maps. Players configure builds and AI before combat; combat executes without player input. This domain covers the spatial system, initiative, action economy, combat resolution, damage types, status effects, stealth, targeting, resources, shields, attrition ramp, win conditions, and the combat scoreboard. Post-combat resolution (injuries, Perk discovery, recruitment, loot) is in [post-combat](post-combat.md). Combat AI configuration is in [combat-ai](combat-ai.md).

---

## Core Concepts

### Battle Format

- **Scale**: 1v1 to 20v20. **2+ teams supported** — free-for-all, multi-faction, and asymmetric sizes (3v5, 1v10, etc.) are all valid configurations. "Enemy" = anyone not on your team.
- **Automation**: Pre-configured AI orders; combat executes automatically (idle/auto-battler). **Simulation-first**: the combat engine runs the entire fight to completion, then presents results (see [Combat Event Stream](#combat-event-stream--deterministic-simulation)).
- **Player interaction**: Configure builds and AI before combat; review results after (dramatized summary + detailed event logs).
- **Win condition**: Elimination with **per-team elimination order**. Last team standing = 1st place. Teams are ranked by elimination order (last eliminated = 2nd, second-to-last = 3rd, etc.). Teams eliminated on the same tick share the same placement rank (next rank skipped). The Attrition Ramp (see below) ensures fights always converge to a conclusion.
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
4. **Fallen Resolution**: All characters whose HP dropped below 1 at any point during this tick enter the Fallen state. Overkill values are recorded. `OnFallen` Triggers fire. Dying Blow Actions taken during the Turns Phase are recorded in the Combat Scoreboard.

**One turn per tick**: Each character acts **at most once per tick**. If a character's Initiative remains ≥ 100 after acting (e.g., fast Action with high Speed), the excess carries over to the next tick. Very fast characters can act in consecutive ticks before anyone else gets a turn — appearing to chain actions — but each action is in a separate tick.

**Full-tick death deferral**: When a character's HP drops below 0 — whether from the Effects Phase (DoTs) or from another character's Action during the Turns Phase — they are **not** immediately removed. They finish the full tick: remaining effects resolve, they accumulate Initiative, and they can even take their turn if they have Initiative ≥ 100. The Fallen state is applied at the end of the tick (step 4). Actions taken by a character at ≤0 HP are **Dying Blows** (see below).

This order means DoTs and status decay happen before any character acts, Initiative accumulates before turns resolve, and death is always deferred to the end of the tick for maximum drama.

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
- **Attack**: Standard damage using equipped weapon's formulas. If no weapon equipped, uses a weak **unarmed attack** (Blunt damage, attribute-scaled with Might). See [Attack & Damage Formulas](#attack--damage-formulas).
- **Move**: Change zones (costs full turn)
- **Defend**: Fixed percentage boost to Defense and Soak until next turn; provides a Stamina recovery burst equal to Y% of max Stamina pool (tuning value)
- **Search**: Active stealth detection — searches entire map (see [Stealth & Detection](#stealth--detection))

**Perk-granted Actions**: Active abilities with variable Action Speed, cooldowns, and resource costs. All cooldowns reset fully between fights (per-combat only — see [traits-and-perks](traits-and-perks.md)).

Perks may grant bonus actions, free movement, or combined movement+attack. These are exceptions, not baseline.

### Action Speed

Action Speed is a flat modifier to the base −100 Initiative cost after acting.

- **Base cost**: −100 Initiative
- **Fast action**: Positive Action Speed modifier. Example: Action Speed +30 → character loses only 70 Initiative (100 − 30 = 70)
- **Slow action**: Negative Action Speed modifier. Example: Action Speed −40 → character loses 140 Initiative (100 − (−40) = 140)
- **Default Actions**: Attack, Move, Defend, and Search have Action Speed 0 (standard −100)

Fast actions let a character act again sooner; slow actions impose a longer delay. This creates meaningful trade-offs between powerful-but-slow abilities and weaker-but-fast ones.

### Combat Resolution Pipeline

All Actions go through a **unified pipeline** with component-driven branching. The pipeline inspects the Action's effect components to determine which steps apply — there is no explicit "action type" flag.

**Step 1: Attack Roll vs. Defense** *(enemy-targeted Damage components only)*
- Skipped entirely if the Action has no Damage components targeting enemies (heals, buffs, shields skip to Step 3)
- Attacker rolls 1 to [Attack Value]
- **Dual Defense**: Defender uses **Physical Defense** or **Magic Defense** based on the Action's **primary damage type**:
  - Physical family (Blunt, Piercing, Slashing) → Physical Defense
  - Elemental family (Fire, Cold, Lightning) → Magic Defense
  - Magical family (Poison, Shadow, Light, Psychic) → Magic Defense
- Defender's Defense = flat subtraction
- Result < 0: **Miss** — no further steps for damage, but non-damage effect components (buffs, status) still resolve
- Result ≥ 0: **Hit** — excess becomes the hit bonus (carried to Step 2)
- **Primary damage type**: Each Action with Damage components designates a primary damage type that determines which Defense to check. All Damage components in the Action benefit from the same hit/miss result.

*Example*: 120 Attack vs. 50 Physical Defense → roll 1–120, subtract 50. Results: −49 (miss) to +70 (strong hit). Hit chance ≈ 58%.

**Step 2: Critical Hit Check** *(universal — applies to all effect components)*
- On a successful hit (or for Actions that skip Step 1), roll against the attacker's Critical Hit Chance
- Crit chance derived from Physical Crit (40% Awareness + 35% Luck + 25% Accuracy) or Magic Crit (40% Awareness + 35% Luck + 25% Intellect), based on primary damage type
- **Universal crit**: Crits apply to ALL effect component types. A crit heal heals more, a crit shield is stronger, a crit damage hits harder. Crit adds a bonus amount to each component's output.
- **Non-crit**: Proceed to Step 3 with base values

**Step 3: Effect Resolution (Type-Ordered)**
- Effect components resolve in type order (see [Effect Resolution Order](#effect-resolution-order-within-actions)):
  1. Status/Buff/Debuff effects (ApplyStatus, RemoveStatus, ModifyStat)
  2. Damage/Healing effects (Damage, Heal, Shield) — each Damage component resolves independently (see below)
  3. Movement effects (Move, Summon)
- **Resistance**: Applies only to **enemy-sourced** effects. Friendly buffs and heals always apply at full strength — allies skip resistance entirely.
- Each rider effect has its own resolution (e.g., Stunning Blow rolls against Stun Resistance)
- Status effects that modify Soak or other defensive stats take effect immediately, potentially affecting damage components in the same step

**Step 4: Per-Component Damage Resolution**
- Each Damage effect component resolves through its **own complete Soak + Shield + Resistance calculation** independently, as if it were a separate hit:
  - **Base damage**: Defined by the component's `formula` field (weapon attacks reference equipped weapon damage; spell Actions define their own formula). Hit bonus from Step 1 and crit bonus from Step 2 are added.
  - **Soak**: Per-damage-type Soak with diminishing returns: `Reduction % = Soak[family] / (Soak[family] + K)` where `Soak[family]` is the defender's Soak for the damage component's family (Physical, Elemental, or Magical)
  - **Penetration**: Universal Penetration reduces the defender's effective Soak (for whichever family applies) before the formula: `Effective Soak = Soak[family] - Penetration` (minimum 0)
  - **Shield absorption**: Post-Soak damage passes through Shields (see [Shield Mechanics](#shield-mechanics))
  - **Final Damage**: Applied to Health
- A "Flame Blade" Action with both Slashing and Fire Damage components resolves Slashing against Physical Soak and Fire against Elemental Soak separately

**Balance Implications**:
- Unified pipeline: one code path for all Actions, branching based on component presence
- Per-type Soak with diminishing returns prevents the "armor wall" problem across every damage type
- Dual Defense (Physical vs Magic) creates a meaningful split between martial and magical attackers
- Universal Penetration is a single stat that counters Soak regardless of damage type
- Universal crit rewards crit investment for healers and support builds, not just attackers
- Status-before-damage means debuffs applied by the same Action affect its own damage
- Per-component resolution rewards Actions with diverse damage types — each component hits a potentially different Soak value
- Allies skip resistance: no counterintuitive buff reduction on your own team

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

Formula modifier tags are checked at formula evaluation time. They modify the attribute used in the calculation, not the formula structure itself.

### DoT Damage Rules

**DoT damage bypasses Soak entirely**. When status effects deal damage during the Effects Phase (Burning, Bleeding, Poison ticks), the damage is applied without Soak reduction.

- **Rationale**: DoTs already passed through the Resistance system when initially applied (which reduced their stack count). Double-gating DoTs through both Resistance and Soak would make them too weak to be meaningful.
- **Shields still absorb DoTs**: Shield HP sits between Soak and Health in the pipeline. Even though DoTs skip Soak, they must still pass through any active Shields before reaching Health. Proactively shielding against DoTs is a valid defensive strategy.
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
- They have their own Initiative meter and act independently in turn order
- They use preset AI configurations (defined by the summoning Perk/consumable)
- Team membership matches the summoner
- **Disappearance**: Summoned entities disappear when killed (enter Fallen) or when their summoner enters Fallen state (end-of-tick Fallen resolution removes both)
- Summoned entities do NOT trigger post-combat phases (no injury checks, no Perk discovery, no recruitment)

### Stun Mechanics

Stun is a status effect that **prevents Initiative gain entirely**:

- While a character has Stun stacks > 0, they gain 0 Initiative per tick during the Initiative Phase
- This means stunned characters never reach Initiative ≥ 100 and never get turns — they are helpless
- **Decay**: Stun stacks decay by an amount proportional to Willpower per tick (during the Effects Phase, before Initiative Phase)
- A character with high Willpower recovers from Stun quickly; low Willpower characters may be stunned for many ticks
- Stun interacts with the one-turn-per-tick rule naturally — a partially stunned character (whose stacks decay to 0 mid-combat) resumes accumulating Initiative from wherever it was

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

1. **Status/Buff/Debuff effects**: ApplyStatus, RemoveStatus, ModifyStat components resolve first
2. **Damage/Healing effects**: Damage, Heal, Shield components resolve second
3. **Movement effects**: Move, Summon components resolve last

This means a single Action can debuff a target's Soak and then deal damage against the reduced Soak in the same use. Components within the same type category resolve simultaneously (no ordering within a category).

**Note**: This overrides the traits-and-perks spec's "simultaneous effect resolution within Actions" rule. The traits-and-perks spec references this spec for the authoritative resolution order.

### Critical Hit System

- **Chance**: Derived from Physical Crit (40% Awareness + 35% Luck + 25% Accuracy) or Magic Crit (40% Awareness + 35% Luck + 25% Intellect), based on the Action's primary damage type. For Actions without a primary damage type (pure heals/buffs), uses the higher of the two crit stats.
- **Trigger**: Checked after a successful hit, or for non-attack Actions, checked when the Action resolves (Step 2 of the pipeline)
- **Universal effect**: Crits apply to **all** effect component types. A crit adds a bonus amount to each component's output — bonus damage for Damage components, bonus healing for Heal components, bonus HP for Shield components, bonus stacks for ApplyStatus, etc.
- **Scaling**: Crit chance and crit bonus can be modified by Perks and equipment

### Damage Types

**Physical**: Blunt (hammers, maces, unarmed), Piercing (spears, arrows, daggers), Slashing (swords, axes, claws)

**Elemental**: Fire (burning, explosions), Cold (freezing, chill), Lightning (shock, thunder)

**Magical**: Poison (toxins, disease), Shadow (darkness, necrotic), Light (holy, radiant), Psychic (mental, illusion)

**Philosophy**: No inherent rock-paper-scissors. Differences come from affiliated status effects and Perk/equipment interactions. Fire *tends* to inflict Burning, Cold *tends* to inflict Slow — thematic patterns, not hard rules.

**MVP Scope**: All 10 damage types are available in MVP. Each needs at least one Trait and representative Perks.

### Status Effect System

**Stack-Based Mechanics**:
- Max stacks: 9999 (effectively unlimited)
- Decay: Variable per status type
- Application: From rider effects, Triggers, environmental hazards, Perks

**Resistance Model**:
- Status effects **always apply** — there is no binary resist/fail roll
- Resistance reduces the **number of stacks applied** AND **damage of that damage type**
- **Enemy-sourced only**: Resistance applies only to effects from enemy sources. Friendly buffs, heals, and beneficial status effects **always apply at full strength** — allies skip resistance entirely
- Per-type resistance: characters can have different resistance values for different status/damage types
- Resistance is tunable per-status-type — some statuses may weight resistance more toward stack reduction, others toward damage reduction
- Sources: Attributes (e.g., Willpower for mental statuses), Perks, equipment, other status effects

*Example*: An attack applies 10 stacks of Burning. The defender has Fire Resistance 40. The resistance formula reduces the applied stacks (e.g., 10 → 6 stacks) AND reduces Fire damage taken (e.g., 40% damage reduction from Fire sources). Exact formulas are tuning values.

**Status Examples** (numbers are tuning targets, not final):

| Status | Effect | Decay Model |
|--------|--------|-------------|
| Stun | **Prevents** Initiative gain entirely (0 gain while stacks > 0) | Decays by Willpower per tick |
| Burning | Fire DoT per tick | Fixed per-tick reduction |
| Bleeding | Physical DoT | Exotic: reduces by 1 per Health restored |
| Slow | Reduces Initiative gain (temporary channel) | Fixed duration or per-tick |
| Sneaking | Cannot be targeted — see Stealth | Broken by conditions (see Stealth) |
| Buff/Debuff | Stat modifications | Various decay rates |

### Stealth & Detection

**Sneaking State**:
- While Sneaking, character cannot be targeted by single-target or selected-target enemy abilities
- **AoE hits Sneaking characters**: Zone-based AoE effects hit all characters in the zone, including Sneaking ones
- **Break conditions**:
  - Using a non-stealth-tagged Action breaks Sneaking (attacking, casting, etc.)
  - Stealth-tagged Actions can be used without breaking Sneaking (ambush attacks, stealth-specific abilities)
  - Taking damage has a **chance** to break Sneaking (probability based on damage taken vs. stealth stacks)
  - Being detected by an enemy does NOT automatically break Sneaking — it only allows that enemy to target the Sneaking character

**Detection System** (dual-mode):
- **Passive Detection**: Automatic, same-zone only. Each tick, characters automatically attempt to detect Sneaking enemies in their zone using their Awareness stat. No action cost.
- **Active Detection (Search)**: A deliberate Action that searches the entire map. Detection chance decreases with range: same zone (high), adjacent zones (moderate), distant zones (low). Costs the character's full turn (default Action via Combatant Trait).
- **Detection result**: Successfully detecting a Sneaking character allows the detector (and their team) to target that character. Detection does not remove the Sneaking status — the Sneaking character retains other stealth benefits (damage break chance, etc.).

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

**Penetration**: Universal — one Penetration stat applied against whatever family Soak is being checked. `Effective Soak = Soak[family] - Penetration` (minimum 0). Penetration does NOT affect Shields (only reduces Soak).

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

- **Fallen**: When a character's HP drops below 1, they enter the Fallen sub-state (from [characters](characters.md)). They are out of the fight for its remainder.
- **Revival**: Fallen characters **can** be revived mid-combat via Perks, consumables, or rare equipment effects. A revived character returns at a fraction of their maximum Health (exact fraction is a tuning value).
- **Overkill**: The amount of excess damage beyond 0 HP is tracked. Overkill magnitude affects injury severity in the post-combat injury roll — massive overkill increases the chance and severity of injuries.

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

---

## Canonical Vocabularies

These vocabularies define the core set of system-recognized keywords. All are extensible — new entries can be added as content without schema changes.

### Target Tags

See [Targeting](#targeting) above for the MVP target tag set and resolution rules.

### Trigger Event Types

The authoritative list of events that can fire Perk Triggers:

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
- `OnDamageDealt` — when this character deals damage (after Soak reduction)
- `OnDamageTaken` — when this character takes damage
- `OnOverkill` — when this character's attack reduces a target below 0 HP

**Kill/Fall Events:**
- `OnKill` — when this character drops an enemy to Fallen
- `OnAllyFallen` — when a friendly character falls in combat
- `OnFallen` — when this character enters the Fallen state
- `OnDyingBlow` — when this character takes an Action while at ≤0 HP (before end-of-tick Fallen resolution)

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
| **Revive** | health_fraction | Revive Fallen ally at 25% HP |
| **ResourceDrain** | resource_type, amount | Drain 30 Mana from target |
| **ResourceGrant** | resource_type, amount | Restore 20 Stamina to self |

New types can be added without schema changes — the list format is inherently extensible.

**Resolution within Actions**: Effect components resolve in type order (see [Effect Resolution Order Within Actions](#effect-resolution-order-within-actions)).

**Level Scaling**: All numeric output values scale with Perk/Trait level multipliers. Resource costs, cooldowns, and requirements stay flat.

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

### Universal Penetration

- **Decision**: Penetration is a single universal stat. When checking Soak, Penetration reduces the relevant family Soak regardless of damage type: `Effective Soak = Soak[family] - Penetration` (minimum 0).
- **Rationale**: One Penetration value is simpler to track and invest in. Per-type or per-family Penetration would create 3-10 separate stats. Universal Penetration is a clean "armor piercing" stat that benefits all damage output.
- **Implications**: Equipment affixes that grant Penetration are universally valuable. A character with high Penetration is effective against all Soak types. Balance: Penetration is powerful because it reduces whichever Soak applies — content should price it accordingly.

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

---

## Open Questions

Most original questions are resolved. Remaining items are tuning values and deferred systems:

### Tuning Values
1. **K constant for Soak formula**: What value of K in `Soak[family]/(Soak[family]+K)` produces the desired damage reduction curve? Needs testing with target stat ranges.
2. **Global Initiative Multiplier exact default**: ~3.0 is the starting point; actual value depends on desired actions-per-character-per-fight.
3. **Crit bonus formula**: How much bonus does a crit add to each effect component type? Flat bonus, percentage of base value, or separate die? (Now applies universally to all component types, not just damage.)
4. **Stealth break-on-damage probability formula**: How does damage taken vs. stealth stacks determine break chance?
5. **Revival Health fraction**: What percentage of max HP does a revived character return with?
6. **Resistance reduction formulas**: Exact formulas for how resistance reduces stacks and damage (per-type tuning curves).
7. **Judgment scaling multiplier and mapping functions**: Per-stat scaling multiplier for Judgment, and the functions mapping Judgment to tactical-vs-personality blend (0.3–0.95), selection sharpness (1–10), and lookahead depth (1–10 ticks).
8. **Stamina regen percentage**: What X% of max pool per tick produces good Stamina pacing?
9. **Defend boost percentages**: What fixed % boost to Defense/Soak and what Y% Stamina burst feels right for the Defend action?
10. **Attrition ramp defaults**: Default onset tick (~100), escalation rate (+0.5–1%/tick), and per-event-type overrides.
11. **Perk Discovery per-Trait rate**: What base rate per qualifying Trait produces similar per-fight discovery rates to the old per-use model?
12. **Per-family Soak scaling multipliers**: What scaling multipliers for the 3 Soak families produce desired damage reduction ranges at target attribute levels?
13. **Physical Defense scaling multiplier**: What scaling multiplier for Physical Defense produces the desired hit/miss rates?

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
| [combat-ai](combat-ai.md) | AI must understand zones, ranges, action options, and target selection across 2+ teams ("enemy" = not my team). AI must handle 20–40+ Actions per character efficiently. AI must evaluate multi-resource Action costs, tag-scoped Stat Adjustment bonuses (including multi-tag additive stacking), stealth/detection trade-offs (passive + active Search), revival priority, and resource conservation. Judgment derived stat (40% Awareness + 35% Willpower + 25% Intellect) controls AI decision quality via the Utility AI system (blend, sharpness, and lookahead depth). Combat log must record AI scoring data (top 3 Actions per turn with scores). Combat system passes context flags to AI at fight start (canonical schema defined here). Characters within a tick act sequentially in Initiative order with state updates between each. `resource_efficiency` Scorer uses fight phase signal (current_tick / attrition_ramp_onset) for conservation strategy. AI must factor shield HP and per-type Soak into target vulnerability and damage output assessments. AI must evaluate primary damage type choice for mixed Actions (which Defense is weaker). Dying Blow Actions need AI handling (character at ≤0 HP still acts). |
| [post-combat](post-combat.md) | Combat hands off to post-combat at victory/defeat. Provides: Combat Scoreboard (per-character stats including Dying Blows), Fallen list with overkill values, active Traits list (for Perk Discovery), Context Flags (exhibition status, event type), per-team elimination order/placement. Post-combat owns: injury/death checks, Perk Discovery resolution, recruitment, loot distribution. |
| [characters](characters.md) | 9 Attributes feed all combat calculations via derived stat formulas (including Judgment, Physical Defense, 3 Soak families). **New derived stats**: Physical Defense (40% Speed + 35% Accuracy + 25% Awareness), Physical Soak (60% Endurance + 25% Might + 15% Speed), Elemental Soak (45% Endurance + 35% Awareness + 20% Luck), Magical Soak (55% Willpower + 25% Intellect + 20% Luck). Anatomical slots determine weapon/armor options. Stamina exhaustion (0→Health drain at 1:1 ratio), percentage-based Stamina regen + Defend boost. Fallen sub-state mechanics with full-tick deferral. Post-combat outcomes handled by [post-combat](post-combat.md). |
| [traits-and-perks](traits-and-perks.md) | Perks provide Actions, Stat Adjustments, Triggers. Traits unlock resource types. Effect resolution order within Actions: status → damage → movement (overrides simultaneous model). All cooldowns reset per-combat. Perk Discovery changed to per-Trait end-of-combat check (active Traits only) — resolved in [post-combat](post-combat.md). Tag-scoped bonuses: flat or %, additive stacking, flat→% order. Trait/Perk level amplification multipliers shared curve ×1.0/×1.2/×1.4/×1.7/×2.0. Actions with Damage components need a `primary_damage_type` field. `OnDyingBlow` added to Trigger event vocabulary for "Last Stand" style Perks. |
| [equipment](equipment.md) | Weapons provide Attack/Damage values and damage type tags. Armor provides per-type Soak bonuses (Physical/Elemental/Magical families + per-type tags). Affixes can grant Penetration (universal), crit bonuses, status riders, stealth-tagged effects, shield effects, starting Initiative offset. Equipment affixes use the tag system for synergies. |
| [consumables](consumables.md) | Consumables are usable during combat turns (potions, bombs, scrolls). Consumables can provide revival (Revive effect component) and shields (Shield effect component). AI must decide when to use them. |
| [tournaments](tournaments.md) | Tournament matches are combat instances. Multi-team formats supported with ranked elimination order. Attrition ramp onset and rate are per-event-type (specified in Context Flags). Per-combat cooldown and resource pool resets between rounds. Star-gated entry to manage power gaps. Post-combat phases (injury, discovery, recruitment, loot) handled by [post-combat](post-combat.md). Fight length varies by event type (25–150 ticks). Event templates define starting zone assignments per team. |
| [meta-balance](../architecture/meta-balance.md) | Combat Scoreboard data (win/loss per character, per Trait, per-character stats, Dying Blows) feeds the automatic underdog balancing system. |
| [data-model](../architecture/data-model.md) | Must support: hidden system Trait type, effect component typed format (type tag + key-value parameters), target tag vocabulary, Trigger event vocabulary (including OnDyingBlow), per-family Soak values (3 families), universal Penetration, Physical + Magic Defense as derived stats, per-component damage resolution, overkill tracking per Fallen character, Combat Scoreboard per-character stat accumulation (including Dying Blows), Shield instances (HP pool, duration, type filter, FIFO ordering), Context Flags schema, attrition ramp state, event template schema (starting zones per team, Initiative offsets), seeded PRNG per combat, structured event stream/log storage, per-team elimination order tracking, `primary_damage_type` field on Actions. |

---

_Last updated: 2026-02-18 — Rounds 10-13 interrogation: 11 additional decisions. Weapon-type-defined Attack & Damage formulas (no fixed derived stats). Zone adjacency: center + ring topology with Long range across opposite cardinals. Attribute-scaled unarmed combat (Might-based, Blunt). Formula modifier tags for attribute substitution (highest value wins). DoTs bypass Soak (Shields still absorb). Passive detection before each turn. Summons as Ephemeral Combatants (own Initiative, preset AI, die with summoner). Stun prevents Initiative gain (Willpower decay). Context Flags team_sizes list for multi-team. Total: 82 decisions across 13 rounds. Previous: round 9 (24 decisions), round 8 (20 decisions), rounds 1-7 (27 decisions)._
