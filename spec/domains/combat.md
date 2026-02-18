# Combat — Domain Specification

**Status**: 🟢 Complete (updated 2026-02-17: fight length revised, context flags added, sequential evaluation noted)
**Last interrogated**: 2026-02-17
**Last verified**: —
**Depends on**: [characters](characters.md), [traits-and-perks](traits-and-perks.md)
**Depended on by**: [combat-ai](combat-ai.md), [tournaments](tournaments.md)

---

## Overview

Combat is the core gameplay loop — automated tactical fights between teams of characters on zone-based maps. Players configure builds and AI before combat; combat executes without player input. This domain covers the spatial system, initiative, action economy, combat resolution, damage types, status effects, stealth, targeting, resources, injury mechanics, and post-combat flow. Combat AI configuration is a separate domain.

---

## Core Concepts

### Battle Format

- **Scale**: 1v1 to 20v20.
- **Automation**: Pre-configured AI orders; combat executes automatically (idle/auto-battler).
- **Player interaction**: Configure builds and AI before combat; spectate during.
- **Target match length**: 25–150 ticks. The range reflects fight variety: short PvE fodder encounters at the low end (~25 ticks), standard tournament matches in the middle (~50–80 ticks), and epic championship finals at the high end (~100–150 ticks). Controlled via the global Initiative multiplier and team composition.

### Spatial System — Zones and Ranges

**Ranges** (binary — an ability is in-range or not, no damage falloff):
- **Short**: Same zone (melee range)
- **Medium**: Adjacent zones
- **Long**: Non-adjacent zones

**Zone Maps**:
- **Default**: 5 zones (North, South, East, West, Center)
- **Variants**: Custom layouts scaled to combatant count (Phase 4)
- **Gimmicks**: Traps, timed hazards, movement modifiers, obstacles (Phase 4)
- **Zone Capacity**: No strict limits by default; can be specified per-zone per-battlefield

**Movement**:
- Basic Move action costs the character's full turn
- Forced movement effects: push to random adjacent zone, push away, pull toward, etc.

**Design Intent**: Making Move cost a full turn creates meaningful positioning. Ranged characters have a genuine advantage — melee fighters pay a real cost to close distance. Perks that grant "attack + move" or "free movement on kill" become high-value abilities.

### Tick Resolution Order

Each combat tick resolves in a fixed order:

1. **Effects Phase**: Per-tick status effects resolve (DoT damage, stack decay, regen ticks, environmental hazards)
2. **Initiative Phase**: All characters add `sqrt(Speed) × global_initiative_multiplier` to their Initiative meters
3. **Turns Phase**: Characters whose Initiative ≥ 100 take their turns in Initiative order (highest first, with tie-breaking rules). Characters evaluate and act **sequentially** — board state updates after each character acts, so later characters react to the updated state. See [combat-ai](combat-ai.md) for team coordination details.

This order means DoTs and status decay happen before any character acts, and Initiative accumulates before turns resolve.

### Initiative System

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
- **Attack**: Standard damage using equipped weapon
- **Move**: Change zones (costs full turn)
- **Defend**: Increase defensive stats until next turn; provides a Stamina recovery burst
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

Resolution follows a fixed pipeline for each attack Action:

**Step 1: Attack Roll vs. Defense**
- Attacker rolls 1 to [Attack Value]
- Defender's Defense = flat subtraction
- Result < 0: **Miss** — no further steps
- Result ≥ 0: **Hit** — excess becomes the hit bonus (carried to Step 2)

*Example*: 120 Attack vs. 50 Defense → roll 1–120, subtract 50. Results: −49 (miss) to +70 (strong hit). Hit chance ≈ 58%.

**Step 2: Critical Hit Check**
- On a successful hit, roll against the attacker's Critical Hit Chance (derived from Physical Crit or Magic Crit stats)
- **Non-crit**: Proceed to Step 3 with base damage
- **Crit**: Roll an additional bonus damage die and add it to the damage pool in Step 4

**Step 3: Status/Rider Application**
- All status effects and rider effects from the Action are applied **before** the damage step
- Each rider has its own resolution (e.g., Stunning Blow rolls against Stun Resistance)
- Status effects that modify Soak or other defensive stats take effect immediately, potentially affecting the upcoming damage step
- Rider effects defined per-Perk and per-equipment-affix; system supports arbitrary rider hooks

**Step 4: Damage Roll vs. Soak**
- Base weapon damage + hit bonus (from Step 1) + crit bonus (from Step 2), rolled against Soak
- **Soak is percentage-based with diminishing returns**: `Reduction % = Soak / (Soak + K)` where K is a global tuning constant
- **Penetration**: Some Perks and equipment affixes grant Penetration, which reduces the defender's effective Soak before the formula is applied. `Effective Soak = Soak - Penetration` (minimum 0)
- Damage after reduction: `Final Damage = Damage Roll × (1 - Reduction%)`
- Final Damage < 1: Fully absorbed
- Final Damage ≥ 1: Applied to Health

**Balance Implications**:
- Percentage Soak with diminishing returns prevents the "armor wall" problem — high Soak is always valuable but never makes a character immune to chip damage
- Penetration creates a meaningful counter-stat to Soak stacking
- Defense (Step 1) remains flat subtraction — still a powerful binary gate between "can hit" and "can't hit"
- Low Attack is completely ineffective vs. high Defense (intentional hard tier gating)
- All variance on attacker's side; defenders have predictable durability
- Status-before-damage means debuffs applied by the same attack affect its own damage (e.g., an armor-shredding rider reduces Soak before the damage step)

### Effect Resolution Order Within Actions

When an Action contains multiple effect components, they are resolved in **type order**, not simultaneously:

1. **Status/Buff/Debuff effects**: ApplyStatus, RemoveStatus, ModifyStat components resolve first
2. **Damage/Healing effects**: Damage, Heal, Shield components resolve second
3. **Movement effects**: Move, Summon components resolve last

This means a single Action can debuff a target's Soak and then deal damage against the reduced Soak in the same use. Components within the same type category resolve simultaneously (no ordering within a category).

**Note**: This overrides the traits-and-perks spec's "simultaneous effect resolution within Actions" rule. The traits-and-perks spec references this spec for the authoritative resolution order.

### Critical Hit System

- **Chance**: Derived from Physical Crit (40% Awareness + 35% Luck + 25% Accuracy) or Magic Crit (40% Awareness + 35% Luck + 25% Intellect), depending on attack type
- **Trigger**: Checked after a successful hit (Step 2 of the pipeline)
- **Effect**: An additional bonus damage roll is added to the damage pool
- **Scaling**: Crit chance and crit bonus damage can be modified by Perks and equipment

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
- Per-type resistance: characters can have different resistance values for different status/damage types
- Resistance is tunable per-status-type — some statuses may weight resistance more toward stack reduction, others toward damage reduction
- Sources: Attributes (e.g., Willpower for mental statuses), Perks, equipment, other status effects

*Example*: An attack applies 10 stacks of Burning. The defender has Fire Resistance 40. The resistance formula reduces the applied stacks (e.g., 10 → 6 stacks) AND reduces Fire damage taken (e.g., 40% damage reduction from Fire sources). Exact formulas are tuning values.

**Status Examples** (numbers are tuning targets, not final):

| Status | Effect | Decay Model |
|--------|--------|-------------|
| Stun | Reduces or prevents Initiative gain | Decays by Willpower per tick |
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

### Resources

**Universal** (all characters):
- **Health**: Fully recovers between battles. **No in-combat regeneration** — all healing comes from Perks, consumables, or equipment procs.
- **Stamina**: General-purpose physical action resource. **Slow passive regen per tick** + larger recovery burst from the Defend action. When Stamina reaches 0, physical actions cost Health instead (exhaustion mechanic from [characters](characters.md)).

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

At the start of each fight, the combat system assembles an explicit **context flags** object and passes it to the AI system. This metadata describes the fight circumstances — tournament round, exhibition status, PvE tier, team sizes, consumable replenishment rules, etc. Any AI Scorer can query these flags to adjust behavior based on context.

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

### Injury and Death Mechanics

After combat resolves, Fallen characters in **non-exhibition** events undergo a tiered injury check:

**Tier 1: Injury Roll**
- Roll to determine if an injury occurs at all
- Factors: overkill magnitude, character Endurance, Luck, Perks that improve injury resistance
- **No injury**: Character returns to Available state
- **Injury**: Proceed to Tier 2

**Tier 2: Severity Roll**
- Roll to determine injury severity
- Factors: same as Tier 1, plus overkill magnitude further increases severity chance
- **Outcomes**: Minor injury (temporary stat penalty, short recovery), major injury (significant stat penalty, long recovery), critical injury (permanent Potential reduction or anatomical slot loss), death (character enters Dead state)

Exhibition events skip injury checks entirely — Fallen characters return to Available.

### Post-Combat Flow

Post-combat resolves as a **phased dramatic presentation**:

1. **Combat Resolution**: Final blow, victory/defeat declared
2. **Injury & Death Phase**: Fallen characters' fates revealed (injury rolls, severity)
3. **Perk Discovery Phase**: Collected discovery events presented one by one (accept/reject per discovery — see [traits-and-perks](traits-and-perks.md))
4. **Recruitment Phase**: If applicable, defeated Named NPCs or generated NPCs offered for recruitment (Charisma/Luck/Awareness check)
5. **Loot Phase**: Equipment, gold, and other rewards distributed

Each phase is presented sequentially for dramatic effect — the player experiences the aftermath as a narrative sequence, not a bulk summary.

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

---

## Open Questions

Most original questions are resolved. Remaining items are tuning values and deferred systems:

### Tuning Values
1. **K constant for Soak formula**: What value of K in `Soak/(Soak+K)` produces the desired damage reduction curve? Needs testing with target stat ranges.
2. **Global Initiative Multiplier exact default**: ~3.0 is the starting point; actual value depends on desired actions-per-character-per-fight.
3. **Crit damage bonus formula**: How much bonus damage does a crit add? Flat bonus, percentage of base damage, or separate die?
4. **Stealth break-on-damage probability formula**: How does damage taken vs. stealth stacks determine break chance?
5. **Revival Health fraction**: What percentage of max HP does a revived character return with?
6. **Injury roll probability tables**: Exact thresholds for tier 1 (injury yes/no) and tier 2 (severity) rolls.
7. **Resistance reduction formulas**: Exact formulas for how resistance reduces stacks and damage (per-type tuning curves).
8. **Judgment scaling multiplier and mapping functions**: Per-stat scaling multiplier for Judgment, and the functions mapping Judgment to tactical-vs-personality blend (0.3–0.95), selection sharpness (1–10), and lookahead depth (1–10 ticks).

### Deferred Systems
8. **Morale**: Full design deferred to Phase 4.
9. **Channeled abilities and interrupts**: Deferred to Phase 2+.
10. **Zone variants and gimmicks**: Deferred to Phase 4.

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [combat-ai](combat-ai.md) | AI must understand zones, ranges, action options, and target selection. AI must handle 20–40+ Actions per character efficiently. AI must evaluate multi-resource Action costs, tag-scoped Stat Adjustment bonuses (including multi-tag additive stacking), stealth/detection trade-offs (passive + active Search), revival priority, and resource conservation. Judgment derived stat (40% Awareness + 35% Willpower + 25% Intellect) controls AI decision quality via the Utility AI system (blend, sharpness, and lookahead depth). Combat log must record AI scoring data (top 3 Actions per turn with scores). Combat system passes context flags to AI at fight start. Characters within a tick act sequentially in Initiative order with state updates between each. |
| [characters](characters.md) | 9 Attributes feed all combat calculations via 16 derived stat formulas (including Judgment for AI). Anatomical slots determine weapon/armor options. Stamina exhaustion (0→Health drain), Stamina regen + Defend boost. Fallen sub-state mechanics implemented. Post-combat flow includes Perk Discovery resolution (accept/reject) alongside injury/death, recruitment, loot. |
| [traits-and-perks](traits-and-perks.md) | Perks provide Actions, Stat Adjustments, Triggers. Traits unlock resource types. Effect resolution order within Actions: status → damage → movement (overrides simultaneous model). All cooldowns reset per-combat. Perk Discovery rolls collected during combat, resolved post-combat. Tag-scoped bonuses: flat or %, additive stacking, flat→% order. Trait/Perk level amplification multipliers shared curve ×1.0/×1.2/×1.4/×1.7/×2.0. |
| [equipment](equipment.md) | Weapons provide Attack/Damage values and damage type tags. Armor provides Defense/Soak. Affixes can grant Penetration, crit bonuses, status riders, stealth-tagged effects. Equipment affixes use the tag system for synergies. |
| [consumables](consumables.md) | Consumables are usable during combat turns (potions, bombs, scrolls). Consumables can provide revival (Revive effect component). AI must decide when to use them. |
| [tournaments](tournaments.md) | Tournament matches are combat instances. Exhibition events skip injury checks. Non-exhibition events trigger tiered injury rolls with overkill severity. Per-combat cooldown and resource pool resets between rounds. Star-gated entry to manage power gaps. Post-combat flow phases apply per round. Fight length varies by event type (25–150 ticks) — tournament pacing must account for longer championship fights. |
| [meta-balance](../architecture/meta-balance.md) | Combat outcomes (win/loss per character, per Trait) feed the automatic underdog balancing system. |
| [data-model](../architecture/data-model.md) | Must support: hidden system Trait type, effect component typed format (type tag + key-value parameters), target tag vocabulary, Trigger event vocabulary, Soak/Penetration/Crit as stat categories, tiered injury roll data, overkill tracking per Fallen character. |

---

_Last updated: 2026-02-17 — Combat AI round 2 integration: revised target match length from 15–25 to 25–150 ticks (PvE fodder to championship finals), added Combat Context Flags (metadata passed to AI at fight start), noted sequential team evaluation in Turns Phase (board state updates between characters), updated Judgment to control three AI parameters (blend, sharpness, lookahead depth). Previous: Added Judgment derived stat for combat AI. 2026-02-16: full interrogation (7 rounds, 27 decisions)._
