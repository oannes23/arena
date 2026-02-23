# Resolution

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

**Off-Hand Bonus Attack**: When dual-wielding or using weapon+focus and the main Action **targets enemies** (has any enemy-targeting component — Damage, debuff, etc.), the off-hand implement fires a **generic Attack** as a completely separate pipeline pass (own hit roll, own crit check, own damage resolution) after the main Action resolves. DW penalties apply (main-hand −15%, off-hand −35% Attack/Damage). Each implement's proc Triggers fire only on its own hits. **Does NOT fire** for non-offensive Actions: heals, self-buffs, Move, Defend, Search. A debuff-only Action targeting enemies DOES trigger the off-hand bonus attack. See [equipment](../equipment/index.md) for full off-hand mechanics.

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

The Summon effect component creates **Ephemeral Combatants** (from [characters](../characters.md)):

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

See [equipment](../equipment/index.md) for Block Chance and Block Value per shield type.

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
