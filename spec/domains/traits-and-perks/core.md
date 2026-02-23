# Core Concepts

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
- **Trait Level Amplification**: A multiplier per level applied to all child Perks (slightly accelerating curve: ×1.0 / ×1.2 / ×1.4 / ×1.7 / ×2.0). Exact values are tuning — deferred to [combat](../combat/index.md) spec.
- **Trait Level Effects**: Modifies ALL Perks within that Trait (higher Trait level = stronger Perks).
- **Requirements**: Can require specific Attributes, other Traits/Perks, derived stats, or achievements.
- **Requirement Checking**: Acquisition-only gates. Once a Trait is owned, it is never deactivated by subsequent stat changes — requirements are only checked at the moment of acquisition.
- **Cross-Category Sharing**: Identical Perks can appear in different Traits across different categories.
- **Acquisition Channels**: Trainers (primary), quest/event rewards, rare combat events (deferred to later phase).
- **Respeccing**: Expensive vendor service from a Group to remove Traits. All XP invested in the removed Trait and its Perks is lost. Gold cost to the Group — formula deferred to [economy](../economy.md) spec. Intentionally expensive: encourages roster turnover over single-character iteration.

### Perk Components

Perks are multi-component packages. Any single Perk can include any combination of:

1. **Actions**: Active combat abilities the character can use on their turn. Action cooldowns are **per-combat only** — all cooldowns reset fully between fights.
2. **Stat Adjustments**: Permanent bonuses applied while the Perk is owned.
3. **Triggers**: Conditional automatic effects that fire when specific combat events occur.

No hard system limit on component count per Perk. Content guidelines suggest 1–3 total components; balance is the content author's responsibility.

**Tradeoff Perks**: Stat Adjustments can have negative values and Triggers can have harmful effects, enabling Perks with built-in tradeoffs (e.g., a Perk granting +30% melee damage via Stat Adjustment but also -20% defense, or a Trigger that drains Health on turn start). No new "Drawback" component type is needed — the existing component system supports negative values natively.

All three component types share the same **effect component list** model for their outputs (see [Effect Component Model](mechanics.md#effect-component-model) below).

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
| **Stamina** | 50% Willpower + 50% Endurance (defined in [characters](../characters.md)) | Advanced combat techniques, weapon mastery | Martial |

Content authors can define additional resource types with custom pool formulas and auto-tags beyond this default set.

### Pool Mechanics

- **Activation**: A resource pool becomes available when a character first owns a Perk that references that resource (costs it or grants pool capacity). The attribute-derived formula provides the base pool size.
- **Bonus capacity**: Perk Stat Adjustment components can grant bonus pool capacity (e.g., "+50 Mana pool") on top of the attribute-derived base. Bonus capacity is added **pre-multiplier** — it adds to the weighted attribute blend before the per-stat scaling multiplier is applied. This means bonus capacity is amplified by the scaling multiplier, making it a powerful investment.
- **Emergent ownership**: Resource pool ownership is emergent from Perk content, not a Trait-level property. A single Trait can have Perks that reference multiple resource types.
- **Auto-tagging**: A Perk that grants or costs a resource type automatically gains the corresponding tag (e.g., a Perk that costs Mana gains the Arcane tag). Traits inherit these tags through the standard union-of-Perk-tags rule.
- **Shared pool**: All Perks that reference the same resource type share one pool per character. The pool grows via Perk-granted bonus capacity, not by creating separate pools per Perk.
- **Multi-resource Actions**: A single Action can cost resources from multiple pools (e.g., 30 Mana + 20 Stamina for a hybrid "Spellblade Strike").
- **Combat start**: Pools start at full capacity at the beginning of combat and regenerate slowly per tick. Exact regen rate deferred to [combat](../combat/index.md) spec.
- **Pool removal**: A resource pool disappears when the character no longer owns any Perks that reference it (e.g., after respeccing the relevant Traits).
- **Martial exception**: Martial-tagged Perks use the universal Stamina pool (already defined on all characters). Martial Perks may add bonus capacity to Stamina rather than granting a new pool.
- **Scaling multipliers**: Like other derived stats, pool formulas use per-stat scaling multipliers. Exact multipliers are tuning values deferred to [combat](../combat/index.md) spec.
