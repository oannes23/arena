# Traits and Perks — Domain Specification

**Status**: 🟢 Complete
**Last interrogated**: 2026-02-16
**Last verified**: —
**Depends on**: [characters](characters.md)
**Depended on by**: [combat](combat.md), [equipment](equipment.md), [consumables](consumables.md), [groups](groups.md), [roster-management](roster-management.md)

---

## Overview

Traits and Perks are the growth and identity system. Traits are the core building blocks of character identity — attached to Trait Slots, they unlock trees of Perks. Perks are the atomic unit of character capability: multi-component packages of Actions, Stat Adjustments, and Triggers. This domain is separate from Characters (which defines the chassis) because Traits/Perks represent the entire growth system with its own leveling, requirements, and acquisition mechanics.

---

## Core Concepts

### Three Trait Categories

| Category | Represents | Examples |
|----------|-----------|---------|
| **Core** | Who the character is — innate identity | Orc, Berserker's Fury, Fated Hero, Touched by Shadow |
| **Role** | What the character can do — functional capabilities | Warrior, Pyromancer, Blacksmith, Medic, Trickster |
| **Bond** | Who the character is connected to — affiliations | Pyromancer's Guild, Priest of Tharzul, Nanna-Sin Monastery |

### Trait Mechanics

- **Minimum Star Level**: Each Trait has a minimum purchase level (1–5★). Acquiring a Trait at a higher minimum star level costs cumulative XP (e.g., minimum 3★ costs 100 + 300 + 600 = 1000 XP).
- **Leveling**: All Traits can be leveled up to 5★ regardless of minimum.
- **XP Cost Schedule**: 0→1★ = 100, 1→2★ = 300, 2→3★ = 600, 3→4★ = 1000, 4→5★ = 1500 XP. Same schedule for Traits and Perks.
- **Leveling Cost**: XP + trainer service fee (gold paid to the Group providing the training).
- **Trait Level Amplification**: A multiplier per level applied to all child Perks (slightly accelerating curve: ×1.0 / ×1.2 / ×1.4 / ×1.7 / ×2.0). Exact values are tuning — deferred to [combat](combat.md) spec.
- **Trait Level Effects**: Modifies ALL Perks within that Trait (higher Trait level = stronger Perks).
- **Requirements**: Can require specific Attributes, other Traits/Perks, derived stats, or achievements.
- **Requirement Checking**: Acquisition-only gates. Once a Trait is owned, it is never deactivated by subsequent stat changes — requirements are only checked at the moment of acquisition.
- **Cross-Category Sharing**: Identical Perks can appear in different Traits across different categories.
- **Acquisition Channels**: Trainers (primary), quest/event rewards, rare combat events (deferred to later phase).
- **Respeccing**: Expensive vendor service from a Group to remove Traits. All XP invested in the removed Trait and its Perks is lost. Gold cost to the Group — formula deferred to [economy](economy.md) spec. Intentionally expensive: encourages roster turnover over single-character iteration.

### Perk Components

Perks are multi-component packages. Any single Perk can include any combination of:

1. **Actions**: Active combat abilities the character can use on their turn. Action cooldowns are **per-combat only** — all cooldowns reset fully between fights.
2. **Stat Adjustments**: Permanent bonuses applied while the Perk is owned.
3. **Triggers**: Conditional automatic effects that fire when specific combat events occur.

No hard system limit on component count per Perk. Content guidelines suggest 1–3 total components; balance is the content author's responsibility.

**Tradeoff Perks**: Stat Adjustments can have negative values and Triggers can have harmful effects, enabling Perks with built-in tradeoffs (e.g., a Perk granting +30% melee damage via Stat Adjustment but also -20% defense, or a Trigger that drains Health on turn start). No new "Drawback" component type is needed — the existing component system supports negative values natively.

All three component types share the same **effect component list** model for their outputs (see [Effect Component Model](#effect-component-model) below).

**Example — Flame Aura Perk:**
- Action: Activate "Flame Aura" (grants 10 stacks, costs 50 Mana, 5-turn cooldown)
- Stat Adjustment: +15 Willpower (permanent while owned)
- Trigger: When struck in melee, gain 1 stack of Flame Aura

### Perk Mechanics

- **Requirements**: Can require Attributes, Traits, other Perks, derived stats, achievements. Requirements are acquisition-only gates (consistent with Trait requirements) — once a Perk is owned, it is never deactivated by stat changes.
- **Minimum Star Level**: 1–5★ minimum purchase level.
- **Leveling**: All Perks can level to 5★.
- **Perk Level Cap**: A Perk's level cannot exceed its parent Trait's level. The Trait must be leveled first to unlock higher Perk levels. **Exception**: Perks acquired via Perk Discovery override this cap at acquisition — they are usable immediately at their minimum star level even if it exceeds the parent Trait level. However, further manual leveling of a discovered Perk is still capped by the parent Trait level.
- **XP Cost Schedule**: Same as Traits — 0→1★ = 100, 1→2★ = 300, 2→3★ = 600, 3→4★ = 1000, 4→5★ = 1500 XP.
- **Perk Level Effects**: Modifies only that specific Perk. Uses the same scaling curve as Trait level amplification (×1.0 / ×1.2 / ×1.4 / ×1.7 / ×2.0). Stacks multiplicatively with Trait level amplification — a 5★ Trait (×2.0) with a 5★ Perk (×2.0) produces a 4.0× total multiplier on base values.
- **Perk Scaling**: Output values (damage, healing, Stat Adjustment bonuses, Trigger effect magnitudes) scale with Perk level via the multiplier curve. Resource costs, cooldowns, and requirements stay flat at all levels.
- **Acquisition**: Purchase from Trainers using XP (must own the parent Trait first).
- **Starter Perk**: Every Trait has exactly one designated Starter Perk that is auto-granted when the Trait is acquired. No XP cost for this initial Perk.
- **Perk Stacking**: When the same Perk is available from multiple Traits, it does not stack. The second source provides redundant access only — insurance if the first source Trait is respecced away.
- **Automatic Redundant Access**: When a character acquires a new Trait that contains a Perk they already own from another Trait, redundant access is granted automatically — no separate purchase required. The Perk is the same entity regardless of source. This also applies in reverse: if Perk X is shared between Trait A and Trait B, purchasing Perk X from either source grants access through both.
- **Shared Perk Amplification**: When the same Perk is owned from two parent Traits at different levels, the higher parent Trait's amplification multiplier applies. The player always benefits from their highest-leveled source.
- **Perk Count per Trait**: 3–5 Perks per Trait (defined per individual Trait), including the Starter Perk. No Perk slot limit per Trait — characters can eventually own all 3–5 Perks in a Trait's tree.
- **No Individual Perk Removal**: Individual Perks cannot be removed. The only way to lose a Perk is to respec the entire parent Trait.

### Perk Prerequisites

- Perks can require other specific Perks as prerequisites, forming a flexible graph (not a linear chain).
- **Cross-Trait prerequisites** are allowed (rare edge case): a Perk in Trait B can require a Perk from Trait A.
- **No cycles**: Circular prerequisites are forbidden (A requires B, B requires A). This is the only hard constraint on graph structure — no maximum depth or branching limit. Content validation catches cycles at authoring time.
- Prerequisites are acquisition-only gates (consistent with all other requirements). Losing the prerequisite Perk (via respeccing its source Trait) does not deactivate dependent Perks.

### Role Trait Dual Purpose

Characters with support Role Traits can fight AND provide household support:
- A Blacksmith Trait includes combat Perks (Strength boost, Smith's Strike) AND support Perks (craft weapons/armor).
- Same character fights in tournaments and provides crafting bonuses between fights.

### Bond Trait Vendor Access

Bond Traits provide vendor access as progression with **hybrid scaling**:
- **Smooth scaling**: Some benefits scale continuously with Bond Trait level — price discounts, inventory breadth, selection quality.
- **Discrete tier unlocks**: Key features unlock at specific Bond Trait levels — exclusive items at 3★, unique services at 5★.
- Bonds unlock themed equipment, consumables, training, and services.

---

## Resource Pools

Resource pools are non-universal resources granted to characters through Perk content. The system is open and extensible — new resource types can be defined by content authors.

### Default Resource Types

Five resource types ship as the default set. Each has a canonical attribute-derived pool formula and an auto-tag:

| Resource | Pool Formula | Typical Perk Types | Auto-Tag |
|----------|-------------|-------------------|----------|
| **Mana** | 60% Intellect + 25% Willpower + 15% Awareness | Elemental magic, arcane spellcasting | Arcane |
| **Faith** | 60% Willpower + 25% Charisma + 15% Luck | Healing, holy/unholy, blessings/curses | Divine |
| **Spirit** | 50% Endurance + 30% Awareness + 20% Willpower | Nature magic, shapeshifting, totems | Primal |
| **Focus** | 60% Awareness + 30% Willpower + 10% Endurance | Telekinesis, mind control, illusions | Psychic |
| **Stamina** | 50% Willpower + 50% Endurance (defined in [characters](characters.md)) | Advanced combat techniques, weapon mastery | Martial |

Content authors can define additional resource types with custom pool formulas and auto-tags beyond this default set.

### Pool Mechanics

- **Activation**: A resource pool becomes available when a character first owns a Perk that references that resource (costs it or grants pool capacity). The attribute-derived formula provides the base pool size.
- **Bonus capacity**: Perk Stat Adjustment components can grant bonus pool capacity (e.g., "+50 Mana pool") on top of the attribute-derived base. Bonus capacity is added **pre-multiplier** — it adds to the weighted attribute blend before the per-stat scaling multiplier is applied. This means bonus capacity is amplified by the scaling multiplier, making it a powerful investment.
- **Emergent ownership**: Resource pool ownership is emergent from Perk content, not a Trait-level property. A single Trait can have Perks that reference multiple resource types.
- **Auto-tagging**: A Perk that grants or costs a resource type automatically gains the corresponding tag (e.g., a Perk that costs Mana gains the Arcane tag). Traits inherit these tags through the standard union-of-Perk-tags rule.
- **Shared pool**: All Perks that reference the same resource type share one pool per character. The pool grows via Perk-granted bonus capacity, not by creating separate pools per Perk.
- **Multi-resource Actions**: A single Action can cost resources from multiple pools (e.g., 30 Mana + 20 Stamina for a hybrid "Spellblade Strike").
- **Combat start**: Pools start at full capacity at the beginning of combat and regenerate slowly per tick. Exact regen rate deferred to [combat](combat.md) spec.
- **Pool removal**: A resource pool disappears when the character no longer owns any Perks that reference it (e.g., after respeccing the relevant Traits).
- **Martial exception**: Martial-tagged Perks use the universal Stamina pool (already defined on all characters). Martial Perks may add bonus capacity to Stamina rather than granting a new pool.
- **Scaling multipliers**: Like other derived stats, pool formulas use per-stat scaling multipliers. Exact multipliers are tuning values deferred to [combat](combat.md) spec.

---

## Species / Ancestry Core Traits

Species are defined entirely via Core Traits — there is no hardcoded species list in the system. A character's biology, anatomy, and innate nature emerge from whichever ancestry Core Traits they possess.

### Anatomy Modification

- Ancestry Core Traits can modify Anatomical Slots: add slots, remove slots, or change slot counts.
- Multiple ancestry Core Traits are allowed and stackable (e.g., "Orc" + "Fey-Touched").
- **Conflict resolution**: Per-slot maximum — take the highest count from any ancestry Trait. Example: if one Trait gives 4 Hand slots and another gives 2, the character has 4.
- A character with 0 ancestry Core Traits gets default humanoid anatomy (as defined in [characters](characters.md)).

### Personality Archetype Tags on Core Traits

Core Traits that represent personality or temperament can be tagged with one or more **personality archetypes**. These tags feed the combat AI's Personality Score track (see [combat-ai](combat-ai.md)), influencing Action selection based on character temperament.

| Archetype | AI Bias | Example Core Trait |
|-----------|---------|-------------------|
| **Aggressive** | Offensive, damage-dealing Actions | Berserker's Fury |
| **Cautious** | Defensive, positioning, evasive Actions | Vigilant Mind |
| **Protective** | Ally-supporting Actions (heals, buffs, shields) | Guardian's Oath |
| **Vindictive** | Prioritize whoever last hurt self or allies | Vengeful Spirit |
| **Showoff** | Flashy, high-impact, expensive Actions | Born Performer |

A character can have multiple personality archetypes from multiple Core Traits. Characters with no personality-tagged Core Traits have neutral AI personality (purely tactical decisions, modulated by Judgment). The Judgment derived stat (see [combat](combat.md)) controls how much personality influences decisions vs tactical optimality.

Personality archetypes are a content-level tag on Core Trait definitions — not a new component type. New archetypes can be added by extending the vocabulary in the combat-ai spec.

### Design Intent

This pattern means:
- New species are added by creating new Core Traits — no schema changes needed.
- Hybrid/mixed ancestry is naturally supported by stacking multiple ancestry Traits.
- Not all Core Traits are ancestry — personality Traits (Berserker's Fury) and destiny Traits (Fated Hero) are also Core but don't modify anatomy.

---

## Tag System

Tags are a shared keyword vocabulary connecting Perks, Equipment, and Consumables. They enable emergent synergies across the build system.

### Where Tags Live

- Tags live on individual **Perks** and their component **Actions**.
- **Traits derive their tags** from the union of all their Perks' tags — Traits have no independent tags.
- **Auto-tags from resources**: Perks that grant or cost a resource type automatically gain the corresponding resource tag (e.g., Mana → Arcane, Faith → Divine). This is in addition to any manually-assigned tags.
- Equipment and Consumables also carry tags (defined in their respective specs).

### How Tags Create Synergies

Tags enable emergent interactions rather than explicit set bonuses:
- A Perk might grant "+10% damage to all [Fire]-tagged Actions" — any [Fire] Action benefits, regardless of source Trait.
- Equipment might grant "+5 Soak vs. [Poison]-tagged damage" — works against any [Poison] source.
- A consumable tagged [Healing] might interact with a Perk that says "when you use a [Healing] effect, also gain 5 Stamina."

### No Explicit Set Bonuses

There are no "if you have Trait X and Trait Y, gain bonus Z" mechanics. All cross-Trait synergies are tag-driven and emergent. This keeps the system open-ended — new Traits automatically interact with existing ones through shared tags.

---

## Trait Generation Loot Tables

When a character is generated at recruitment, their starting Traits are determined by loot table rolls (see [characters](characters.md) for the roll mechanics). This section defines the loot table structure.

### Table Structure

- **Weighted probability pools** with nestable subtables.
- Top-level table can branch into subtables (e.g., Ancestry 20%, Personality 30%, Gift 30%, Background 20%).
- Tables are nestable — one archetype table can include another archetype's table as a subtable outcome.
- **Per-archetype definition**: Each archetype's loot table lives with the Group that offers recruitment (defined in the [groups](groups.md) spec).

### Starting Level

- Generated Traits start at their minimum required star level. A min-3★ Trait on a newly generated character starts at 3★ (with cumulative XP cost already "paid" by generation).

### Roll Modifiers

- **Per-star-level roll overrides**: Different probability distributions for the 1st roll vs. 2nd roll vs. 3rd roll within a category. Example: 1st Core roll weighted toward ancestry; 2nd weighted toward personality; 3rd toward gifts.
- **Openness parameter**: A stacking percentage chance per star level to zero out a slot before rolling on the table.
  - Low openness = veteran archetype (most slots pre-filled with Traits).
  - High openness = open-potential archetype (many empty slots for the player to fill later).
  - Example: 15% openness means each roll has a 15% chance of becoming empty. For a 3★ character, this applies independently to each of the 3 rolls.
- **Duplicate handling**: Only discard if the exact same Trait is rolled again (same-category, different-Trait is always fine). Duplicate Trait rolls unlock a random Perk from that Trait's tree instead (see [characters](characters.md)).

---

## Perk Discovery

A rare mechanic that rewards continued use of existing Perks.

### Discovery Rolls (During Combat)

- **Trigger conditions**: Discovery triggers on **Actions** (when actively used) and **Triggers** (when they fire). Passive Stat Adjustments do not trigger discovery rolls.
- When a qualifying Perk component fires in combat, there is a ~0.1% base chance to trigger a discovery event for the parent Trait's tree.
- **Rarity weighting**: The discovery roll is weighted by rarity — lower-minimum-star Perks are more likely to be discovered than higher-minimum-star ones.
- Discovery is per-qualifying-component-use, so characters who use many Actions and have many Triggers have more chances.
- **Full tree**: If all Perks in the Trait's tree are already owned, the discovery roll is silently wasted.
- Even rare 5★ Perks can be discovered this way — though they are much less likely due to rarity weighting.

### Discovery Resolution (Post-Combat)

- Discovery events are **collected during combat but resolved post-combat**, in the same phase as injury/death checks, recruitable NPC generation, and loot distribution.
- The discovered Perk is presented to the player, who can **accept or reject** it.
- **Accept**: The Perk is acquired at its minimum star level, free of XP cost.
- **Reject**: The Perk is not acquired. It remains in the discovery pool for future rolls. No penalty for rejection.
- **Level cap override**: Accepted discovered Perks override the Perk level cap at the moment of acquisition — they are acquired at their minimum star level even if it exceeds the parent Trait level. However, further manual leveling of the discovered Perk is still capped by the parent Trait level.
- Multiple discoveries can occur in a single combat (from different Traits or even the same Trait). Each is presented individually for accept/reject.

### Design Intent

- Post-combat resolution avoids interrupting combat flow and gives the player time to consider.
- Accept/reject adds meaningful player agency to the random system — especially important now that Perks can have tradeoffs (negative Stat Adjustments or harmful Triggers).
- Rejection keeps the Perk in the pool, so players can't "clear out" undesirable Perks by rejecting them permanently — they'll keep appearing until the tree is full.
- Proactively buying desired Perks via Trainers shrinks the discovery pool, reducing the chance of discovering unwanted tradeoff Perks.

**Trait discovery** (acquiring entirely new Traits via combat, rather than just new Perks from existing Traits) is deferred to a later phase.

---

## Effect Component Model

Actions, Triggers, and Stat Adjustments all express their outputs using a shared **effect component list** — an ordered list of generic effect components, each with a type tag and key-value parameters.

### Component Types (Illustrative)

The canonical catalog of effect component types is owned by the [combat](combat.md) spec. The following are the core categories for content authoring reference:

| Type | Parameters | Example |
|------|-----------|---------|
| **Damage** | damage_type, formula | Deal 20 [Fire] damage |
| **Heal** | formula | Restore 30 HP |
| **ApplyStatus** | status_type, stacks, duration | Apply 3 stacks of Burning for 5 ticks |
| **RemoveStatus** | status_type, stacks | Remove 5 stacks of Poison |
| **Move** | target, zones | Push target 1 zone away |
| **Summon** | template | Summon a Fire Elemental |
| **Shield** | formula, duration | Grant 50-point shield for 3 ticks |

New effect component types can be added to the combat spec without schema changes — the list format is inherently extensible.

### Effect Resolution Order

Effect components within a single Action or Trigger are resolved in **type order** as defined by the [combat](combat.md) spec: status/buff/debuff effects first, then damage/healing effects, then movement effects. Components within the same type category resolve simultaneously. This enables Actions that debuff a target and then damage them against the reduced stats in a single use. Multi-Trigger resolution remains simultaneous — all qualifying Triggers fire together with no ordering dependency.

### Level Scaling

All numeric output values in effect components (damage amounts, healing, status stacks, shield values, movement range, etc.) scale with the Perk/Trait level multiplier. Resource costs, cooldowns, and requirement thresholds stay **flat** at all levels. This is a universal rule — there is no per-component opt-in/opt-out for scaling.

---

## Action Targeting Model

Actions use a **tag-based targeting system**. Each Action specifies a set of target tags that describe what the Action can target. The [combat](combat.md) spec defines how target tags resolve to valid targets on the battlefield.

### AI Considerations on Actions

Each Action includes an `ai` block that defines how the combat AI evaluates that Action. This is part of the Action data model — content authors define AI behavior alongside ability mechanics. See [combat-ai](combat-ai.md) for the full Utility AI system.

**Structure:**
- **Gates**: Binary prerequisites (resource_gate, cooldown_gate, range_gate, target_exists_gate). Any Gate failing vetoes the Action.
- **Tactical Scorers**: Continuous 0.01–1.0 scores evaluating objective effectiveness (damage_output, healing_urgency, resource_efficiency, etc.).
- **Personality Scorers**: Continuous 0.01–1.0 scores evaluating fit with character temperament (aggression_preference, protective_preference, etc.).

Content authors select from the standard Consideration library defined in [combat-ai](combat-ai.md). Custom Considerations can be added for unique Actions.

**Example:**
```
Action: Flame Burst
  ...
  ai:
    gates: [resource_gate, cooldown_gate, range_gate, target_exists_gate]
    tactical_scorers:
      - aoe_clustering: { min_targets: 2, ideal_targets: 4 }
      - target_vulnerability: { tag: Fire }
      - resource_efficiency: { resource: Mana }
    personality_scorers:
      - aggression_preference: { weight: 0.6 }
```

Default Actions (Attack, Defend, Move, Search) from the Combatant system Trait also have embedded Considerations, designed as viable gap-fillers that naturally lose to Perk Actions as characters scale (Perk Actions benefit from Trait × Perk level multipliers up to ×4.0).

### Target Tag Examples

- `[Self]` — targets the acting character only
- `[Enemy, Single]` — targets one enemy
- `[Ally, Single]` — targets one ally
- `[Enemy, AoE-Zone]` — targets all enemies in a zone
- `[Ally, AoE-Zone]` — targets all allies in a zone
- `[All, AoE-Zone]` — targets everyone in a zone (friend and foe)

New targeting modes are new tags — no schema changes required. Combat spec owns the canonical target tag vocabulary and resolution rules.

---

## Stat Adjustment Model

Stat Adjustments use a **flat + tag-scoped** modifier system with two subtypes:

### Flat Bonuses

Direct numeric bonuses to any stat in the system — primary Attributes, derived stats, resource pool capacities, combat stats (crit chance, resistances, etc.).

**Examples:**
- `+5 Might`
- `+50 Mana pool capacity`
- `+3% Crit Chance`
- `+10 Physical Soak`

### Tag-Scoped Bonuses

Conditional bonuses that apply only when the matching tag is present. Can modify any numeric stat in the system when the tag condition is met. Tag-scoped bonuses can be **flat or percentage** — both subtypes are valid.

**Percentage stacking**: When multiple percentage bonuses apply to the same stat, they stack **additively** — both within the same tag and across different tags. Example: `+10% [Fire] damage` and `+15% [Fire] damage` combine to `+25%`. If an Action is tagged both [Fire] and [Arcane], bonuses from both tags stack additively: `+10% [Fire]` + `+15% [Arcane]` = `+25% total damage` on that Action.

**Application order**: Flat bonuses are applied first (to the base value), then the combined additive percentage bonus is applied as a single multiplier to the flat-adjusted total. Perk/Trait level scaling multipliers are applied before Stat Adjustment bonuses — Stat Adjustments modify the already-scaled output.

**Examples:**
- `+10% damage to [Fire] Actions` — percentage boost to Fire-tagged damage
- `-15% resource cost for [Martial] Actions` — percentage reduction for Martial-tagged costs
- `+5 Soak vs [Poison] damage` — flat conditional defense against Poison-tagged damage
- `+20% healing from [Divine] effects` — percentage boost to Divine-tagged healing
- `-1 tick cooldown for [Arcane] Actions` — flat conditional reduction to Arcane cooldowns

Tag-scoped bonuses interact with the Tag System — the same tags used for emergent synergies drive conditional modifiers.

---

## Trigger Resolution

When multiple Triggers from different Perks all fire on the same combat event (e.g., "when struck in melee"), all qualifying Triggers fire simultaneously. Effects are collected and applied together with no ordering dependency. Resolution is deterministic from the set of active Triggers, not from acquisition or slot order.

### Trigger Event Types (Illustrative)

The canonical vocabulary of Trigger event types is owned by the [combat](combat.md) spec. The following are example event categories for content authoring reference:

- **OnHit** — when this character successfully hits an enemy
- **OnHitBy** — when this character is hit by an enemy
- **OnKill** — when this character kills an enemy
- **OnAllyFallen** — when a friendly character falls in combat
- **OnCombatStart** — at the beginning of combat
- **OnTurnStart** — at the start of this character's turn
- **OnTurnEnd** — at the end of this character's turn
- **OnStatusApplied** — when a status effect is applied to this character
- **OnResourceDepleted** — when a resource pool reaches zero

These are illustrative — combat spec defines the full set and resolution semantics.

---

## Bond Trait Perk Design

Bond Trait Perks follow a **hybrid utility + combat** pattern. Bond Perks grant both out-of-combat utility (discounts, service access, crafting bonuses) and combat capabilities (themed Actions, Stat Adjustments, Triggers).

### Design Rationale

- Bond Traits represent deep Group affiliation — it's natural that a Pyromancer's Guild member gains fire-themed combat abilities alongside vendor access.
- Keeps Bond Trait investment meaningful for combat-focused players — Bonds aren't pure utility tax.
- Higher-level Bond Perks (4–5★) tend to have stronger combat components, rewarding deep Group investment.

### Examples

- **Pyromancer's Guild Bond**: Starter Perk grants member vendor access + a basic fire buff. Higher Perks might add a fire spell Action, fire resistance Stat Adjustment, and a fire-reactive Trigger.
- **Temple of Tharzul Bond**: Starter Perk grants healing service access. Higher Perks might add a divine healing Action and a faith-boosting Stat Adjustment.

---

## Perk Discovery Modifiers

The base ~0.1% Perk Discovery chance is modified by multiple sources using **additive flat bonuses** and a **Trait Level multiplier**.

### Formula

`Discovery Chance = (Base Rate (0.1%) + Luck Bonus + Perk Bonuses) × Trait Level Multiplier`

### Modifier Sources

- **Luck**: Small flat bonus from the Luck Attribute. Exact formula is a tuning value (e.g., +0.05% at 100 Luck).
- **Trait Level Multiplier**: The parent Trait's level acts as a multiplier on the discovery chance, using the standard amplification curve (×1.0 / ×1.2 / ×1.4 / ×1.7 / ×2.0). A 5★ Trait doubles the discovery chance for its tree. This applies universally to all Trait categories (Core, Role, Bond).
- **Specific Perks**: Certain Perks can grant flat bonuses to Perk Discovery chance (e.g., a "Scholar's Insight" Perk might add +0.1% to all discovery rolls). Since these are numeric output values, they scale with the Perk/Trait level multiplier like all other outputs.

### Design Intent

- Multiple modifier sources reward diverse investment — Luck, Trait leveling, and specific Perks all contribute.
- Trait Level as a multiplier (not additive) creates a meaningful incentive to level Traits: higher Trait level = meaningfully higher discovery rate for that tree.
- Perk-granted discovery bonuses being subject to level scaling creates a compounding loop — a high-level "Scholar's Insight" Perk in a high-level Trait becomes increasingly potent.
- Additive base model (before the multiplier) is transparent and easy for players to reason about.
- Exact tuning values are deferred to balance — the system should keep discovery rare enough that trainers remain the primary acquisition path.

---

## Respec and Equipment Interaction

When a Trait is respecced, equipment that required that Trait for equipping **stays equipped**. This is consistent with the acquisition-only gate philosophy — equipment requirements are checked at equip time only, not maintained continuously.

- Equipment bonuses remain active even if the underlying Trait requirement is no longer met.
- The player would need to unequip and re-equip to trigger a new requirement check.
- This prevents frustrating cascade effects where respeccing a Trait causes gear to fall off, which could further reduce stats and cascade into other failures.

---

## Trainer Availability

- **Group-specific**: Each Group's trainers only teach Traits and Perks aligned with that Group's theme.
- **Generalist Groups**: Combat academies, gladiatorial trainers, and similar generalist Groups exist that teach diverse, broad, low-level basic Traits. These provide baseline access without requiring membership in a specialized Group.

---

## Trait Slot Constraints

- Traits can only be acquired into **empty slots**. No overflow, no queue, no replacement shortcut.
- If all slots in a category are full, the player must respec an existing Trait to make room before acquiring a new one.
- **No star gate**: A character's Star Rating does not restrict which Traits they can acquire. A 1★ character can acquire a 5★-minimum Trait if they have an empty slot, meet the Trait's requirements, and can afford the XP. Star Rating only determines slot count.
- **Empty categories allowed**: Characters can respec down to zero Traits in any category. A character with no Role Traits is valid — they simply lack Role-granted capabilities.
- **One Bond per Group**: A character can hold only one Bond Trait per Group. The Bond Trait levels (1–5★) to represent deepening affiliation. Acquiring a deeper relationship with a Group means leveling the existing Bond Trait, not acquiring a second one.

---

## Decisions

### Three-Category Trait System

- **Decision**: Traits are divided into Core (identity), Role (capability), and Bond (affiliation) categories with separate slot pools.
- **Rationale**: Creates character identity along three axes. Prevents "all combat" builds by requiring investment across identity, capability, and affiliation.
- **Implications**: Character generation must produce Traits from all three pools. Each category needs enough content for meaningful choice at every star level.

### Perk as Multi-Component Package

- **Decision**: A single Perk can contain any mix of Actions, Stat Adjustments, and Triggers rather than being one component type. No hard system limit on component count — content guidelines suggest 1–3 total components per Perk.
- **Rationale**: Enables rich, thematic abilities (Flame Aura has an active component, a passive bonus, AND a reactive trigger). Reduces Perk count while increasing Perk depth. Soft guidelines keep content manageable without constraining design space.
- **Implications**: Perk balance must consider the combined value of all components. UI must display multi-component Perks clearly.

### Trait Level Amplifies All Child Perks

- **Decision**: Leveling a Trait (1–5★) amplifies ALL Perks within that Trait via a per-level multiplier (slightly accelerating: ×1.0 / ×1.2 / ×1.4 / ×1.7 / ×2.0). Leveling a specific Perk (1–5★) amplifies only that Perk using the same curve. Both stack multiplicatively — max total at 5★/5★ = 4.0× base values.
- **Rationale**: Creates a "wide vs. deep" investment choice — level the Trait for broad improvement, or level specific Perks for focused power. Accelerating curve rewards full commitment. Shared curve keeps the system simple.
- **Implications**: Balance must account for double-stacking (high Trait level × high Perk level). Exact multiplier values are tuning — deferred to combat spec.

### Fixed Escalating XP Schedule

- **Decision**: Both Traits and Perks use the same XP cost schedule: 0→1★ = 100, 1→2★ = 300, 2→3★ = 600, 3→4★ = 1000, 4→5★ = 1500 XP. Total to max (0→5★) = 3500 XP.
- **Rationale**: Escalating costs create natural diminishing returns at higher levels. Shared schedule between Traits and Perks keeps the system simple. Combined with trainer service fees (gold), creates dual-currency pressure.
- **Implications**: Economy spec must define trainer service fee scaling. A character needs 3500 XP to max one Trait + 3500 per Perk — full investment in one 5-Perk Trait costs ~21,000 XP.

### Cross-Category Perk Sharing — No Stack

- **Decision**: Identical Perks can appear in different Traits across categories. When a character has the same Perk from two sources, it does not stack — the second source provides redundant access only. The higher parent Trait's amplification multiplier applies.
- **Rationale**: Redundant access is insurance: if you respec away one Trait, you keep the Perk from the other. No stacking prevents degenerate double-dipping builds. Using the higher amplification is simple and always benefits the player.
- **Implications**: Players may intentionally overlap Perks across Traits for safety, but there is no power incentive to do so.

### Starter Perk Auto-Grant

- **Decision**: Every Trait has exactly one designated Starter Perk. This Perk is auto-granted (free, no XP cost) when the Trait is acquired.
- **Rationale**: Ensures every Trait is immediately useful upon purchase — no Trait is a blank slate requiring additional XP investment before it does anything. The Starter Perk is the Trait's "signature move."
- **Implications**: Trait design must always include a compelling Starter Perk. The Starter Perk defines the Trait's first impression and basic capability.

### Expensive Respec

- **Decision**: Removing Traits is an expensive vendor service from a Group. All XP invested in the removed Trait and its Perks is lost. Gold cost to the Group — formula deferred to [economy](economy.md) spec.
- **Rationale**: Encourages roster turnover — it's often better to hire a new character than endlessly respec an existing one. Creates attachment to characters and investment in hiring decisions.
- **Implications**: Players need enough income to hire replacements. The meta shifts toward roster breadth over single-character perfection.

### Open Extensible Resource Pool System

- **Decision**: Resource pools are emergent from Perk content, not a Trait-level property. Five default resource types ship (Mana, Faith, Spirit, Focus, Stamina) with attribute-derived pool formulas. Content authors can define additional resource types with custom formulas and auto-tags. A resource pool activates when a character first owns a Perk that references it; the attribute formula provides the base, Perk Stat Adjustments add bonus capacity.
- **Rationale**: Emergent ownership is simpler than explicit Trait-to-family assignments and naturally handles multi-family Traits. Open extensibility means new resource types don't require system changes. Attribute-derived base provides meaningful variation between characters; Perk bonuses reward investment.
- **Implications**: Combat spec must implement resource pool formulas, activation, and scaling. Content authoring tool must support defining new resource types. No hard cap on how many resource pools a character can have — bounded by Perk content. AI must handle multi-resource Action costs.
- **Alternatives considered**: Fixed 5 families with Trait-level assignment (rejected — too rigid, prevented multi-family Traits), per-Perk-only pools with no attribute base (rejected — removed character stat relevance from pools).

### Species as Core Traits

- **Decision**: Species/ancestry are defined entirely via Core Traits with anatomy modification support. Multiple ancestry Traits are stackable. Conflict resolution uses per-slot maximum. Default humanoid anatomy for characters with no ancestry Traits.
- **Rationale**: Maximally extensible — new species are just new Core Traits. Supports hybrid ancestries naturally. No hardcoded species list means content can grow without schema changes.
- **Implications**: Core Trait definitions must specify any anatomical slot modifications. Equipment system must handle variable anatomy.

### Tag-Driven Synergies

- **Decision**: All cross-Trait synergies are emergent via shared tags. No explicit set bonuses between specific Traits. Tags live on Perks/Actions; Traits derive tags from the union of their Perks. Resource-referencing Perks auto-gain the corresponding resource tag (e.g., Mana → Arcane).
- **Rationale**: Tag-driven synergies are open-ended and scale with content — new Traits automatically interact with existing ones through shared tags. Explicit set bonuses would require manual curation and create a combinatorial explosion as content grows. Auto-tagging from resources ensures consistent categorization without manual effort.
- **Implications**: Tag vocabulary must be well-curated to enable meaningful but not overpowered synergies. Equipment and consumable specs share the same tag vocabulary.

### Perk Level Cap by Trait Level

- **Decision**: A Perk's level cannot exceed its parent Trait's level. Exception: Perks acquired via Perk Discovery override this cap at acquisition, but further manual leveling remains capped.
- **Rationale**: Creates a natural investment order — Trait first (broad power), then Perks (specific power). Prevents characters from having max-level Perks in a low-level Trait. Discovery exception rewards the rarity of the event at the moment of discovery, while maintaining the normal progression path afterward.
- **Implications**: UI must communicate the cap clearly. Leveling a Trait unlocks higher Perk level ceilings for all its Perks, including previously-discovered Perks.

### Acquisition-Only Requirement Gates

- **Decision**: All requirements (Trait, Perk, prerequisite) are checked only at acquisition. Once owned, items are never deactivated by subsequent stat changes or prerequisite loss.
- **Rationale**: Prevents frustrating "stat juggling" where equipping/unequipping items could cascade-deactivate build components. Simple mental model: once you have it, you keep it.
- **Implications**: Players can safely respec a Trait without worrying about deactivating Perks in other Traits that had cross-Trait prerequisites from it.

### Cross-Trait Interaction — Tags Only

- **Decision**: No Perk can name or reference another specific Trait or Perk. All cross-Trait interaction is via shared tags. The system is fully decoupled.
- **Rationale**: Tag-only interaction is maximally extensible — new Traits automatically interact with existing ones through shared tags without requiring manual cross-references. Prevents combinatorial explosion of explicit Trait-to-Trait interactions.
- **Implications**: Content authors must design Perks using tag references, not Trait/Perk names. Tag vocabulary curation becomes a key design task.
- **Alternatives considered**: Explicit cross-Trait synergies (rejected — scales poorly), hybrid tag + named references (rejected — complexity for little gain).

### Trainer Availability by Group

- **Decision**: Training is Group-specific — each Group's trainers teach only Traits/Perks aligned with their theme. Generalist Groups (combat academies, gladiatorial trainers) teach diverse low-level basic Traits.
- **Rationale**: Makes Group membership and Bond Traits more meaningful — players need connections to the right Groups to access desired training. Generalist Groups ensure baseline accessibility.
- **Implications**: Groups spec must define trainer menus per Group. Economy spec must handle pricing differences between specialist and generalist trainers.

### Perk Discovery via Combat

- **Decision**: ~0.1% base chance per qualifying Perk component use (Actions and Triggers only — not passive Stat Adjustments) to discover a random unowned Perk from the same Trait's tree. Base rate modified by additive bonuses (Luck, Perks) then multiplied by Trait level (×1.0–×2.0). Acquired at minimum star level, free of XP cost. Discovery roll is rarity-weighted (lower-minimum-star Perks more likely). Discovered Perks override the Perk level cap at acquisition only; further leveling remains capped. Full tree = silently wasted roll.
- **Rationale**: Rewards continued engagement with existing Perks. Creates exciting surprise moments. The low rate prevents it from undermining the trainer economy. Rarity weighting makes discovery of powerful Perks appropriately rare. Trait level multiplier rewards deep investment.
- **Implications**: Combat spec must trigger Perk discovery checks on Action use and Trigger fire events. Economy impact is minimal at base 0.1% but should be monitored — at 5★ Trait with Luck and Perk bonuses, effective rate could approach ~0.5%.

### No Star Gate for Trait Acquisition

- **Decision**: A character's Star Rating does not restrict which Traits they can acquire. A 1★ character can acquire a 5★-minimum Trait if they have an empty slot, meet the Trait's requirements, and can afford the XP. Star Rating determines slot count only.
- **Rationale**: The XP cost is already a natural gate — a min-5★ Trait costs 3500 XP cumulative, which is prohibitively expensive for a low-star character early on. Adding a star gate would be redundant and reduce build flexibility for players who want to invest heavily in a single Trait on a low-star character.
- **Implications**: Economy is the primary gate for high-star Traits on low-star characters. The cumulative XP cost (100/300/600/1000/1500) naturally prevents abuse.

### Simultaneous Trigger Resolution

- **Decision**: When multiple Triggers fire on the same combat event, all qualifying Triggers fire simultaneously. Effects are collected and applied together with no ordering dependency.
- **Rationale**: Simplest model. Eliminates order-dependent interactions, which would be confusing for players and difficult to balance. No incentive to acquire Perks in a specific order.
- **Implications**: Combat spec implements collected-and-applied-together resolution. No Trigger can depend on the result of another Trigger firing on the same event.

### Multi-Resource Action Costs

- **Decision**: A single Action can cost resources from multiple pools (e.g., 30 Mana + 20 Stamina for a hybrid "Spellblade Strike"). Any combination of resource types is allowed.
- **Rationale**: Enables hybrid-fantasy abilities and creates interesting resource management tension. Characters who invest in multiple resource families can access powerful cross-discipline Actions.
- **Implications**: Combat AI must evaluate multi-resource costs when choosing Actions. Content authors must balance multi-resource Actions carefully — they should be powerful enough to justify the broader resource investment.

### One Bond Trait per Group

- **Decision**: A character can hold only one Bond Trait per Group. The Bond Trait levels (1–5★) to represent deepening affiliation. No separate "Apprentice" vs. "Master" Bond Traits for the same Group.
- **Rationale**: Simplifies the Bond–Group relationship to a clean 1:1 mapping per character. Bond Trait leveling already provides progression depth through hybrid scaling (smooth benefits + discrete tier unlocks). Multiple Bond Traits per Group would dilute the leveling system.
- **Implications**: Groups spec: each Group defines exactly one Bond Trait. Bond Trait level is the single axis of Group relationship depth.

### Empty Category Respec

- **Decision**: Characters can respec down to zero Traits in any category. A character with no Role Traits, for example, is valid — they simply lack Role-granted capabilities.
- **Rationale**: Maximum flexibility. The guaranteed-first-roll rule at generation ensures characters start with at least one Trait per category, but post-generation respec should not be artificially constrained. An empty category is a meaningful (if unusual) player choice.
- **Implications**: UI should clearly communicate when a category is empty. Characters with empty Role slots may have limited combat utility but are not system-invalid.

### Tag-Based Action Targeting

- **Decision**: Actions use a tag-based targeting system. Each Action specifies target tags (e.g., `[Enemy, Single]`, `[Ally, AoE-Zone]`, `[Self]`). Combat spec defines how tags resolve to valid targets.
- **Rationale**: Extensible — new targeting modes are new tags, no schema changes. Tags are already the universal interaction vocabulary in the system. Consistent with the tag-driven design philosophy.
- **Implications**: Combat spec must define canonical target tag vocabulary and resolution rules. AI must parse target tags when evaluating valid Actions.

### Effect Component List Model

- **Decision**: Actions and Triggers express their outputs as an ordered list of generic effect components, each with a type tag and key-value parameters. Core component types (illustrative): Damage, Heal, ApplyStatus, RemoveStatus, Move, Summon, Shield. Full catalog owned by combat spec.
- **Rationale**: Extensible — new effect types don't require schema changes, just new type tags. Structured enough for validation and AI reasoning, flexible enough for content authors.
- **Implications**: Data model needs a component-list container for Actions and Triggers. Combat spec defines the canonical effect type catalog. Content authoring tools must validate component parameters.

### Type-Ordered Effect Resolution Within Actions

- **Decision**: Effect components within a single Action or Trigger resolve in type order: status/buff/debuff effects first, damage/healing effects second, movement effects last. Components within the same type category resolve simultaneously. This is defined authoritatively by the [combat](combat.md) spec.
- **Rationale**: Enables powerful "setup-and-exploit" Actions (e.g., apply armor shred then deal damage against reduced Soak in the same Action). Creates richer design space for content authors while maintaining deterministic resolution.
- **Implications**: An Action CAN have an ApplyStatus that reduces Soak followed by a Damage component that benefits from the reduction. Content authors can design debuff+damage combo Actions. Multi-Trigger resolution remains simultaneous (no ordering between different Triggers firing on the same event).
- **Alternatives considered**: Fully simultaneous resolution (original model — rejected in favor of type-ordered for richer design space), fully sequential per-component ordering (rejected — too complex, ordering-dependent).

### Shared Effect Components Across Actions and Triggers

- **Decision**: Triggers use the exact same effect component list as Actions (Damage, Heal, ApplyStatus, Move, Summon, Shield, etc.). The only difference is activation: Actions are chosen by player/AI, Triggers fire automatically.
- **Rationale**: Unified effect model simplifies the system — one set of effect types, one scaling model, one resolution model. Content authors learn one system for both Actions and Triggers.
- **Implications**: Trigger balance must account for automatic activation — a Trigger that Summons on every hit would be more impactful than an Action that does the same. Content guidelines should address this.

### Flat + Tag-Scoped Stat Adjustments

- **Decision**: Stat Adjustments use two subtypes: flat bonuses (direct numeric bonus to any stat — Attributes, derived stats, resource pools, crit chance, etc.) and tag-scoped bonuses (conditional bonus when matching tag is present — any numeric stat, any tag scope).
- **Rationale**: Flat bonuses cover simple passive power. Tag-scoped bonuses create build depth by rewarding tag concentration without requiring explicit cross-Trait references. Covers both simple and conditional modifiers in a unified system.
- **Implications**: Data model needs two Stat Adjustment representations: `{stat_key, value}` for flat, `{tag, stat_key, value}` for tag-scoped. Tag-scoped bonuses interact with Tag System — same tags drive synergies and conditional modifiers.

### Bond Trait Hybrid Perk Pattern

- **Decision**: Bond Trait Perks grant both out-of-combat utility (discounts, service access, crafting bonuses) and combat capabilities (themed Actions, Stat Adjustments, Triggers).
- **Rationale**: Deep Group affiliation naturally grants combat knowledge and power alongside practical benefits. Keeps Bond investment meaningful for combat-focused players — Bonds aren't pure utility tax.
- **Implications**: Bond Perks create another source of combat power, giving Groups direct combat relevance beyond training access. Content balance must ensure Bond combat Perks complement (not replace) Role Trait combat power.

### Perk Discovery Modifiers: Additive Base × Trait Level

- **Decision**: Multiple sources modify the base ~0.1% Perk Discovery chance. Additive flat bonuses (Luck Attribute, specific Perks) combine into a base rate, which is then multiplied by the parent Trait's level using the standard amplification curve. Formula: `(Base Rate + Luck Bonus + Perk Bonuses) × Trait Level Multiplier`.
- **Rationale**: Multiple modifier sources reward diverse investment. Additive base model is transparent. Trait Level as a multiplier creates meaningful incentive to level Traits and uses existing mechanics. Perk-granted bonuses scale with level (consistent with all-numeric-outputs-scale rule), creating compounding optimization for discovery builds.
- **Implications**: Tuning must keep total discovery rate low enough that trainers remain primary acquisition. Luck gains another meaningful use. Trait level now has a secondary benefit beyond Perk amplification. At 5★ Trait (×2.0), effective discovery rate doubles. Exact bonus values are tuning — deferred to balance.

### Tag-Scoped Bonuses: Both Flat and Percentage, Additive Stacking

- **Decision**: Tag-scoped Stat Adjustment bonuses can be flat (+5 Soak vs [Poison]) OR percentage (+10% [Fire] damage). When multiple percentage bonuses apply to the same stat and tag, they stack additively. Application order: flat bonuses first, then combined percentage as a single multiplier. Level scaling multipliers apply before Stat Adjustments.
- **Rationale**: Allowing both flat and percentage tag-scoped bonuses maximizes design space — flat bonuses are better for small conditional buffs, percentages for scaling conditional multipliers. Additive percentage stacking is transparent and prevents exponential blowup from stacking many small percentage sources.
- **Implications**: Combat resolution must apply tag-scoped bonuses in the correct order (flat → percentage). Content authors should be aware that percentage bonuses become more powerful on high-base-damage Actions. Balance must monitor percentage accumulation across many Perks.

### Pre-Multiplier Bonus Pool Capacity

- **Decision**: Bonus pool capacity from Stat Adjustments (e.g., "+50 Mana pool") adds to the weighted attribute blend before the per-stat scaling multiplier is applied. The scaling multiplier amplifies both the attribute-derived base and the bonus capacity together.
- **Rationale**: Pre-multiplier placement makes bonus pool capacity a powerful and valuable investment — it scales with the same multiplier as the base pool. This rewards deep Perk investment in pool capacity and creates meaningful differentiation between characters who invest in pool bonuses and those who don't.
- **Implications**: Content authors must account for the scaling multiplier when designing bonus pool capacity values — a "+50" bonus becomes much larger after the multiplier (e.g., ×10 → +500 effective). Balance must ensure bonus capacity values are tuned for pre-multiplier placement. Combat spec's per-stat scaling multipliers directly affect the power of bonus pool capacity.

### Trait Level Multiplier on Perk Discovery

- **Decision**: Replace the "Bond Level Bonus" in the Perk Discovery formula with a universal Trait Level Multiplier. The parent Trait's level multiplies the discovery chance using the standard amplification curve (×1.0/×1.2/×1.4/×1.7/×2.0). Applies to all Trait categories (Core, Role, Bond). Perk-granted discovery bonuses are numeric outputs and thus also scale with level.
- **Rationale**: Simpler and more universal than a Bond-to-Trait association lookup. Eliminates the ambiguity of which Group is "thematically associated" with Core/Role Traits. Uses existing mechanics (the standard amplification curve) rather than introducing a new concept. Creates a natural incentive to level Traits beyond just Perk amplification.
- **Implications**: The Bond Level Bonus is removed from the formula — Bond Traits benefit from this equally via their own Trait level, not through a special association mechanism. Discovery chance at 5★ Trait is 2× the base rate, creating meaningfully higher discovery for deeply invested Traits. Perk-granted discovery bonuses compound with Trait level, creating interesting build optimization for discovery-focused characters.
- **Alternatives considered**: Bond Level Bonus with per-Trait Group association metadata (rejected — added data complexity, ambiguous for many Core/Role Traits), trainer-derived association (rejected — many-to-many relationship made it unclear which Bond level to use).

### Per-Combat Cooldown Reset

- **Decision**: All Action cooldowns reset fully between fights. Every combat encounter starts with a clean slate — no cooldown state persists across combats, even within multi-round tournament events.
- **Rationale**: Simplest model. Eliminates cross-combat state tracking. Ensures every fight is self-contained and fair — no player is disadvantaged by having used powerful abilities in a previous round. Consistent with resource pools starting at full capacity at combat start.
- **Implications**: Tournament balance relies on per-fight resource management, not cross-fight attrition. Powerful long-cooldown Actions are usable every fight. If cross-fight attrition is desired for tournaments, it should be expressed through other mechanics (injuries, Stamina, etc.) rather than cooldown carry-over.

### Automatic Redundant Access for Shared Perks

- **Decision**: When a character acquires a new Trait that contains a Perk they already own from another Trait, redundant access is granted automatically with no separate purchase. The Perk is the same entity regardless of which Trait tree it appears in.
- **Rationale**: A Perk is a single identity — "Perk X from Trait A" and "Perk X from Trait B" are the same Perk. Requiring separate purchase would create confusing UX (paying XP for something you already have) and undermine the "redundant access as insurance" design. Auto-access is the simplest model consistent with the no-stacking rule.
- **Implications**: When displaying Trait trees, Perks already owned from other sources should be visually marked as "already owned." Perk Discovery rolls in a new Trait should skip already-owned shared Perks (they're not "unowned"). Respeccing one source Trait leaves the Perk intact via the other source — no re-purchase needed.

### Acquisition-Only Equipment Requirements

- **Decision**: Equipment that required a specific Trait for equipping stays equipped when that Trait is respecced away. Equipment requirements are checked at equip time only, consistent with the acquisition-only gate philosophy.
- **Rationale**: Prevents frustrating cascade effects where respeccing a Trait causes gear to unequip, which could further reduce stats. Consistent with the universal "once you have it, you keep it" model.
- **Implications**: Equipment spec must implement equip-time-only requirement checks. Players can exploit this by equipping gear before respeccing — this is intended behavior (a form of planning reward).

### All Numeric Outputs Scale With Level

- **Decision**: Every numeric value in an effect component that represents an output (damage, healing, status stacks, shield amount, movement range) scales with the Perk/Trait level multiplier. Resource costs, cooldowns, and requirement thresholds stay flat at all levels. This is a universal rule with no per-component exceptions.
- **Rationale**: Simple universal rule that's easy to understand and balance. Scaling outputs + flat costs means higher levels are always worth it — more power for the same resource investment. No per-component opt-in keeps content authoring simple.
- **Implications**: Content authors design base values knowing they'll be multiplied up to 4.0× at max level (5★ Trait × 5★ Perk). Combat balance must account for this range.

### Discovery Rate: No Cap, Small Trees Sufficient

- **Decision**: No hard cap on effective Perk Discovery rate. At high investment (5★ Trait + Luck + discovery-boosting Perks), per-roll rates can reach ~1%, giving ~30-40% chance per combat of discovering a Perk. The natural limit is the small tree size (2-4 discoverable Perks per Trait). Discovery-optimized builds filling trees faster is the intended reward for that investment.
- **Rationale**: The tree size is the natural cap. Once all Perks are owned, rolls are wasted. A character investing heavily in discovery is choosing to use Perk slots and Trait levels for discovery bonuses rather than direct combat power — that's a legitimate build choice with meaningful opportunity cost.
- **Implications**: Trainer economy remains viable because discovery requires deep investment to reach high rates, and each Trait tree has very few undiscovered slots. Monitoring recommended: if discovery rates make trainers irrelevant, tuning the base rate (0.1%) is the lever.

### Post-Combat Discovery Resolution with Rejection

- **Decision**: Discovery rolls happen during combat but resolution occurs post-combat (alongside injury checks, NPC recruitment, loot). The player can accept or reject each discovered Perk. Rejected Perks remain in the discovery pool for future rolls.
- **Rationale**: Post-combat resolution avoids interrupting combat flow. Accept/reject adds player agency, especially important now that Perks can have tradeoffs (negative components). Rejection keeping the Perk in the pool prevents players from "clearing out" undesirable Perks — they must buy them from trainers or accept the recurring discovery risk.
- **Implications**: Combat spec must collect discovery events during combat and pass them to the post-combat phase. Post-combat UI must present each discovered Perk with full details for informed accept/reject decisions. Characters spec's post-combat phase now includes discovery resolution alongside injury/death, NPC recruitment, and loot.

### Tradeoff Perks via Existing Components

- **Decision**: Perks can include built-in tradeoffs using negative Stat Adjustment values and harmful Triggers. No new "Drawback" component type is needed. Content authors design tradeoff Perks using the existing Stat Adjustment (negative values) and Trigger (harmful effects) systems.
- **Rationale**: The component system already supports negative values — adding a separate Drawback type would be redundant complexity. Keeping tradeoffs within the existing framework means they benefit from all existing mechanics (level scaling, tag interaction, simultaneous resolution). Content guidelines can recommend tradeoff patterns without requiring system changes.
- **Implications**: Content authors can create Perks like "Berserker's Rage" (+30% melee damage, -20% defense) or Triggers like "Blood Frenzy" (gain damage on kill, lose Health each turn). Since all numeric outputs scale with level, both the positive and negative components of tradeoff Perks scale together — a 5★ tradeoff Perk has a bigger bonus AND a bigger penalty.

### Multi-Tag Additive Stacking

- **Decision**: When a multi-tagged Action (e.g., [Fire, Arcane]) is affected by tag-scoped bonuses from different tags, all percentage bonuses stack additively into a single total. `+10% [Fire] damage` + `+15% [Arcane] damage` = `+25% total damage` on that Action.
- **Rationale**: Additive cross-tag stacking is the simplest model and consistent with same-tag additive stacking. It rewards diverse tag investment without creating multiplicative explosions. Easy for players to reason about: add up all matching percentages.
- **Implications**: Multi-tag Actions benefit more from broad tag investment — a character with bonuses across multiple tags gets more value from Actions that match many tags. Content balance should be aware that Actions with many tags accumulate more bonuses. This is intentional — multi-tag Actions represent hybrid capabilities that reward broad builds.

### Equal Category Combat Power

- **Decision**: No Trait category "owns" combat power. Core, Role, and Bond Traits can all provide combat-relevant Actions, Stat Adjustments, and Triggers. A character built primarily around Core combat passives + Bond combat abilities (with no Role Traits) is a valid and potentially strong build.
- **Rationale**: Maximum build flexibility. Restricting combat power to Role Traits would make Core and Bond categories feel like taxes. Each category provides combat power through its own lens: Core = innate/biological advantages, Role = trained/specialized capabilities, Bond = group-taught thematic abilities.
- **Implications**: Build diversity comes from content variety and economic constraints, not from category restrictions. Combat-ai spec must handle characters with widely varying Action sources and compositions. Content design should ensure each category has a distinct flavor of combat contribution even though none is restricted.

### No Action Count Limit

- **Decision**: No mechanical limit on how many Actions a character can have available in combat. A fully built 5★ character with many Traits and Perks could have 20-40+ Actions. The combat AI evaluates all available Actions; situational filtering (targeting constraints, resource costs, cooldowns, tag-scoped bonuses) naturally narrows the effective choice set.
- **Rationale**: Limiting Actions would require an "equipped Actions" management layer that adds complexity without clear benefit. The AI auto-battles, so the player doesn't need to manually navigate large Action lists. Having many options is the reward for deep build investment.
- **Implications**: Combat-ai spec must handle large Action pools efficiently without performance issues or poor decision-making. The AI's ability to evaluate and prioritize among many Actions is a critical design challenge. UI for reviewing a character's Action list must handle large counts gracefully.

### Intentional Power Gap Across Star Ratings

- **Decision**: The power gap between low-star and high-star characters is large and intentional. A 5★ character has 5× the Trait slots of a 1★ character and potentially 4.0× multipliers on each. However, Trait/Perk star levels are NOT capped by character star level — a 1★ character can have 5★ Traits with 5★ Perks. The gap is breadth (slot count), not depth (individual Trait/Perk power).
- **Rationale**: Large power gaps create aspiration and reward long-term investment. Star-gated tournaments (1★-only, 3★-only, open) prevent unfair mismatches. A well-built 2★ beating a poorly-built 4★ is a design goal — build quality should matter. Roguelike turnover means reaching true max on a 5★ character is a rare achievement, not the norm.
- **Implications**: Tournament spec must support star-gated entry. Meta-balance spec (see below) provides automatic underdog bonuses for systematically disadvantaged builds. The combination of star gates + underdog bonuses + build quality creates a competitive landscape where investment matters but isn't insurmountable.

### Build Diversity Philosophy

- **Decision**: Build diversity is sustained by three complementary forces: economic scarcity (can't afford everything), content variety (design space too large to "solve"), and roster pressure (need different characters for different situations). Additionally, an automatic **underdog balancing** system tracks win/loss ratios per Trait and applies bonuses to underperforming Traits, preventing any single dominant strategy from persisting.
- **Rationale**: No single mechanism is sufficient for diversity — economic scarcity alone leads to "save up for the best" behavior, content variety alone can still have a solved meta, roster pressure alone doesn't address 1v1 balance. The combination, plus automatic meta-balance, creates a self-correcting ecosystem.
- **Implications**: **New spec needed**: a meta-balance spec to define the automatic underdog balancing system (win/loss tracking scope, bonus calculation, update frequency, visibility to players). Economy spec must create genuine scarcity. Content must be broad enough that no single optimal path exists. Roster-management spec must ensure multi-character investment is necessary.

### Prerequisite Graph: No Cycles Only

- **Decision**: The only hard constraint on Perk prerequisite graphs is no circular dependencies (A requires B, B requires A). No maximum depth or branching limit. Cross-Trait prerequisites remain allowed (rare). Content validation tools catch cycles at authoring time.
- **Rationale**: Minimal constraints maximize content design flexibility. Deep prerequisite chains create meaningful progression within a Trait. Cross-Trait prerequisites (rare) enable "prestige" Perks that require broad investment. Cycles are mathematically impossible to satisfy and must be prevented.
- **Implications**: Content authoring tools must include cycle detection for prerequisite graphs. No runtime enforcement needed — validation is purely at content creation time.

---

## Open Questions

All structural, mechanical, and game design questions are resolved. Three non-blocking items are deferred:

1. **MVP content examples** (deferred to implementation): What are the specific ~3 Core, ~4 Role, ~2 Bond Traits with their 3–5 Perks each? This is content authoring, not design.
2. **Combat-triggered Trait discovery** (deferred to later phase): Can characters acquire entirely new Traits (not just Perks) through combat events?
3. **Automatic underdog balancing** (deferred to new meta-balance spec): Win/loss ratio tracking per Trait with automatic bonuses for underperforming Traits. Cross-cutting system requiring its own specification.

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [characters](characters.md) | Trait Slot count defined by Character Star Rating. Core Traits may modify anatomical slots. Species = Core Traits pattern (no hardcoded species list). Post-combat phase now includes Perk Discovery resolution (accept/reject) alongside injury/death checks, NPC recruitment, and loot. |
| [combat](combat.md) | Perks provide Actions (combat abilities), Stat Adjustments (combat stats), and Triggers (combat reactions). Simultaneous Trigger resolution (all qualifying Triggers fire together, no ordering). **Type-ordered** effect resolution within Actions: status/debuff → damage/healing → movement (defined by combat spec). Resource pool system: open/extensible, attribute-derived base + Perk bonus capacity (pre-multiplier), pools activate when first Perk references a resource. Multi-resource Action costs (any combo of pools). Resource pools start at full capacity; no default regen for Trait-granted resources (regen comes from Perks/Triggers). Per-stat scaling multipliers for resource pools (also amplify bonus capacity). Trait/Perk level amplification multipliers — shared curve ×1.0/×1.2/×1.4/×1.7/×2.0 (tuning values). Tag-scoped Stat Adjustments can be flat or percentage; percentages stack additively across different tags on the same Action; application order: flat → percentage on already-scaled values. All cooldowns reset per-combat (no cross-fight persistence). Perk Discovery: rolls collected during combat, resolved post-combat (accept/reject). Base rate modified by Luck/Perks (additive) then ×Trait level (×1.0–×2.0). No rate cap — small trees are natural limit. Tradeoff Perks (negative Stat Adjustments / harmful Triggers) possible via existing components. Combat spec defines: canonical target tag vocabulary and resolution rules, canonical Trigger event type vocabulary, canonical effect component type catalog. |
| [equipment](equipment.md) | Equipment can require specific Traits/Perks (acquisition-only check — stays equipped if requirement later unmet). Affix bonuses can enhance specific Perks. Tags create Perk–equipment synergies (shared tag vocabulary, including auto-tags from resources). Variable anatomy from ancestry Core Traits affects available slots. |
| [consumables](consumables.md) | Specialist Perks (Bomber, Apothecary) amplify consumable effectiveness. Scribe Perk enables Scroll creation from known Perks. Tags create Perk–consumable synergies. |
| [combat-ai](combat-ai.md) | Personality archetype tags on Core Traits feed the AI Personality Score track. Each Perk Action includes an `ai` block (gates, tactical_scorers, personality_scorers) that defines how the Utility AI evaluates it. AI must evaluate multi-resource Action costs, tag-scoped Stat Adjustment bonuses (including multi-tag additive stacking), and tag-based targeting. Default Actions (Combatant system Trait) have embedded Considerations. **Key challenge**: No Action count limit — Utility AI must efficiently score 20-40+ Actions per character. |
| [economy](economy.md) | Trainer service fees (gold) for Trait/Perk leveling. Respec gold cost formula. XP cost schedule (100/300/600/1000/1500) interacts with economy balance. Group-specific trainer fee model (specialist vs. generalist pricing differences). XP cost as natural gate for high-star Traits on low-star characters. Discovery modifier stacking should be monitored — trainers must remain primary acquisition path. Economic scarcity is a primary driver of build diversity. |
| [groups](groups.md) | Trait generation loot table definitions per Group/archetype. Respec is a Group vendor service. Trainer services for Trait/Perk acquisition and leveling. Bond Traits map to specific Groups — one Bond Trait per Group per character. Trainer availability model: Group-specific theme alignment, generalist Groups for baseline access. Bond Trait hybrid scaling: smooth benefits (price, breadth) + discrete tier unlocks (exclusive items at 3★, unique services at 5★). Bond Trait Perks include combat capabilities (hybrid utility + combat pattern). |
| [tournaments](tournaments.md) | Star-gated tournament entry (1★-only, 3★-only, open) to manage power gaps. Per-combat cooldown reset means no cross-fight cooldown attrition in multi-round events. Power gap between star levels is intentional — build quality should enable underdog victories. |
| [roster-management](roster-management.md) | Trait composition drives character hiring value. Respec is a vendor service in the economy. Starter Perks affect character readiness at recruitment. Empty Trait categories are valid post-respec states. Roster pressure is a primary driver of build diversity — no single character can cover all situations. |
| [meta-balance](../architecture/meta-balance.md) | **NEW SPEC NEEDED**. Automatic underdog balancing: win/loss ratio tracking per Trait, automatic bonuses for underperforming Traits. Prevents solved meta and ensures build diversity. Needs to define: tracking scope (per-Trait? per-combination?), bonus calculation, update frequency, player visibility. |
| [data-model](../architecture/data-model.md) | Trait/Perk data shape definition needed: Trait → Perk tree, tag lists, loot table weights. Resource type definitions are content data (name, pool formula, auto-tag) — not hardcoded schema. Perk components need structured representation: Actions (target tags, effect component list, speed, cooldown, resource costs), Stat Adjustments (two subtypes: flat `{stat_key, value}` and tag-scoped `{tag, stat_key, value, value_type: "flat"|"percentage"}`), Triggers (event type, effect component list). Effect components use generic typed format (type tag + key-value parameters). Prerequisite graph must be acyclic (content validation). |

---

_Last updated: 2026-02-17 — Added AI Considerations as Perk Action component (ai block with gates, tactical_scorers, personality_scorers). Added personality archetype tags on Core Traits for combat AI Personality Score track. Updated combat-ai implications for Utility AI system. Previous: 2026-02-16 effect resolution order update._
