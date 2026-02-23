# Mechanics

## Effect Component Model

Actions, Triggers, and Stat Adjustments all express their outputs using a shared **effect component list** — an ordered list of generic effect components, each with a type tag and key-value parameters.

### Component Types (Illustrative)

The canonical catalog of effect component types is owned by the [combat](../combat/index.md) spec. The following are the core categories for content authoring reference:

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

Effect components within a single Action or Trigger are resolved in **type order** as defined by the [combat](../combat/index.md) spec: status/buff/debuff effects first, then damage/healing effects, then movement effects. Components within the same type category resolve simultaneously. This enables Actions that debuff a target and then damage them against the reduced stats in a single use. Multi-Trigger resolution remains simultaneous — all qualifying Triggers fire together with no ordering dependency.

### Level Scaling

All numeric output values in effect components (damage amounts, healing, status stacks, shield values, movement range, etc.) scale with the Perk/Trait level multiplier. Resource costs, cooldowns, and requirement thresholds stay **flat** at all levels. This is a universal rule — there is no per-component opt-in/opt-out for scaling.

---

## Action Targeting Model

Actions use a **tag-based targeting system**. Each Action specifies a set of target tags that describe what the Action can target. The [combat](../combat/index.md) spec defines how target tags resolve to valid targets on the battlefield.

### AI Considerations on Actions

Each Action includes an `ai` block that defines how the combat AI evaluates that Action. This is part of the Action data model — content authors define AI behavior alongside ability mechanics. See [combat-ai](../combat-ai.md) for the full Utility AI system.

**Structure:**
- **Gates**: Binary prerequisites (resource_gate, cooldown_gate, range_gate, target_exists_gate). Any Gate failing vetoes the Action.
- **Tactical Scorers**: Continuous 0.01–1.0 scores evaluating objective effectiveness (damage_output, healing_urgency, resource_efficiency, etc.).
- **Personality Scorers**: Continuous 0.01–1.0 scores evaluating fit with character temperament (aggression_preference, protective_preference, etc.).

Content authors select from the standard Consideration library defined in [combat-ai](../combat-ai.md). Custom Considerations can be added for unique Actions.

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

The canonical vocabulary of Trigger event types is owned by the [combat](../combat/index.md) spec. The following are example event categories for content authoring reference:

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

The per-Trait Perk Discovery chance is modified by multiple sources using **additive flat bonuses** and a **Trait Level multiplier**. The base rate is a tuning value (higher than the old per-use 0.1% to produce similar per-fight discovery rates with the per-Trait model).

### Formula

`Discovery Chance per Trait = (Base Rate + Luck Bonus + Perk Bonuses) × Trait Level Multiplier`

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
