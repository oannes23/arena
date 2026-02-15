# Traits and Perks — Domain Specification

**Status**: 🟢 Complete
**Last interrogated**: 2026-02-14
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
- **Trait Level Amplification**: A multiplier per level applied to all child Perks (slightly accelerating curve, e.g., ×1.0 / ×1.2 / ×1.4 / ×1.7 / ×2.0). Exact values are tuning — deferred to [combat](combat.md) spec.
- **Trait Level Effects**: Modifies ALL Perks within that Trait (higher Trait level = stronger Perks).
- **Requirements**: Can require specific Attributes, other Traits/Perks, derived stats, or achievements.
- **Requirement Checking**: Acquisition-only gates. Once a Trait is owned, it is never deactivated by subsequent stat changes — requirements are only checked at the moment of acquisition.
- **Cross-Category Sharing**: Identical Perks can appear in different Traits across different categories.
- **Acquisition Channels**: Trainers (primary), quest/event rewards, rare combat events (deferred to later phase).
- **Respeccing**: Expensive vendor service from a Group to remove Traits. All XP invested in the removed Trait and its Perks is lost. Gold cost to the Group — formula deferred to [economy](economy.md) spec. Intentionally expensive: encourages roster turnover over single-character iteration.

### Perk Components

Perks are multi-component packages. Any single Perk can include any combination of:

1. **Actions**: Active combat abilities the character can use on their turn.
2. **Stat Adjustments**: Permanent attribute bonuses applied while the Perk is owned.
3. **Triggers**: Conditional automatic effects that fire when specific combat events occur.

**Example — Flame Aura Perk:**
- Action: Activate "Flame Aura" (grants 10 stacks, costs 50 Mana, 5-turn cooldown)
- Stat Adjustment: +15 Willpower (permanent while owned)
- Trigger: When struck in melee, gain 1 stack of Flame Aura

### Perk Mechanics

- **Requirements**: Can require Attributes, Traits, other Perks, derived stats, achievements. Requirements are acquisition-only gates (consistent with Trait requirements) — once a Perk is owned, it is never deactivated by stat changes.
- **Minimum Star Level**: 1–5★ minimum purchase level.
- **Leveling**: All Perks can level to 5★.
- **Perk Level Cap**: A Perk's level cannot exceed its parent Trait's level. The Trait must be leveled first to unlock higher Perk levels. **Exception**: Perks acquired via Perk Discovery override this cap — they are usable immediately at their minimum star level even if it exceeds the parent Trait level.
- **XP Cost Schedule**: Same as Traits — 0→1★ = 100, 1→2★ = 300, 2→3★ = 600, 3→4★ = 1000, 4→5★ = 1500 XP.
- **Perk Level Effects**: Modifies only that specific Perk (independent of Trait level, but stacks multiplicatively with it).
- **Perk Scaling**: Output values scale with Perk level (damage, healing, Stat Adjustment bonuses, Trigger effect magnitudes). Resource costs, cooldowns, and requirements stay flat at all levels.
- **Acquisition**: Purchase from Trainers using XP (must own the parent Trait first).
- **Starter Perk**: Every Trait has exactly one designated Starter Perk that is auto-granted when the Trait is acquired. No XP cost for this initial Perk.
- **Perk Stacking**: When the same Perk is available from multiple Traits, it does not stack. The second source provides redundant access only — insurance if the first source Trait is respecced away.
- **Perk Count per Trait**: 3–5 Perks per Trait (defined per individual Trait), including the Starter Perk. No Perk slot limit per Trait — characters can eventually own all 3–5 Perks in a Trait's tree.
- **No Individual Perk Removal**: Individual Perks cannot be removed. The only way to lose a Perk is to respec the entire parent Trait.

### Perk Prerequisites

- Perks can require other specific Perks as prerequisites, forming a flexible graph (not a linear chain).
- **Cross-Trait prerequisites** are allowed (rare edge case): a Perk in Trait B can require a Perk from Trait A.
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

## Resource Families

Each Resource Family defines a non-universal resource pool unlocked by owning Traits in that family. There are 5 families:

| Family | Resource | Pool Formula | Typical Trait Types |
|--------|----------|-------------|---------------------|
| **Arcane** | Mana | 60% Intellect + 25% Willpower + 15% Awareness | Elemental magic, arcane spellcasting |
| **Divine** | Faith | 60% Willpower + 25% Charisma + 15% Luck | Healing, holy/unholy, blessings/curses |
| **Primal** | Spirit | 50% Endurance + 30% Awareness + 20% Willpower | Nature magic, shapeshifting, totems |
| **Psychic** | Focus | 60% Awareness + 30% Willpower + 10% Endurance | Telekinesis, mind control, illusions |
| **Martial** | Stamina | Uses Stamina pool (50% Willpower + 50% Endurance, defined in [characters](characters.md)) | Advanced combat techniques, weapon mastery |

### Pool Mechanics

- **Hybrid sizing**: Base pool from the attribute-derived formula above + bonus from Trait levels in that family.
- **Ownership**: Each Trait belongs to at most one Resource Family. Owning any Trait in a family grants that family's resource pool.
- **Shared pool**: Multiple Traits in the same family share one pool — the pool grows via Trait-level bonuses, not by creating separate pools per Trait.
- **Combat start**: Pools start at full capacity at the beginning of combat and regenerate slowly per tick. Exact regen rate deferred to [combat](combat.md) spec.
- **Pool growth**: Emergent from Perk content (Stat Adjustment components that add to the pool), not a system-level bonus for stacking Traits.
- **Pool removal**: A resource pool disappears immediately when the last Trait in that Resource Family is removed via respec.
- **Martial exception**: Martial family uses the universal Stamina pool (already defined on all characters). Martial Traits may add bonus capacity to Stamina rather than granting a new pool.
- **Scaling multipliers**: Like other derived stats, pool formulas use per-stat scaling multipliers. Exact multipliers are tuning values deferred to [combat](combat.md) spec.

---

## Species / Ancestry Core Traits

Species are defined entirely via Core Traits — there is no hardcoded species list in the system. A character's biology, anatomy, and innate nature emerge from whichever ancestry Core Traits they possess.

### Anatomy Modification

- Ancestry Core Traits can modify Anatomical Slots: add slots, remove slots, or change slot counts.
- Multiple ancestry Core Traits are allowed and stackable (e.g., "Orc" + "Fey-Touched").
- **Conflict resolution**: Per-slot maximum — take the highest count from any ancestry Trait. Example: if one Trait gives 4 Hand slots and another gives 2, the character has 4.
- A character with 0 ancestry Core Traits gets default humanoid anatomy (as defined in [characters](characters.md)).

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

- **Trigger conditions**: Discovery triggers on **Actions** (when actively used) and **Triggers** (when they fire). Passive Stat Adjustments do not trigger discovery rolls.
- When a qualifying Perk component fires in combat, there is a ~0.1% chance to **discover** a random unowned Perk from the same Trait's tree.
- **Rarity weighting**: The discovery roll is weighted by rarity — lower-minimum-star Perks are more likely to be discovered than higher-minimum-star ones.
- The discovered Perk is acquired at its minimum star level, free of XP cost.
- **Level cap override**: Discovered Perks override the Perk level cap — they are acquired active at their minimum star level even if it exceeds the parent Trait level.
- Even rare 5★ Perks can be discovered this way — though they are much less likely due to rarity weighting.
- Discovery is per-qualifying-component-use, so characters who use many Actions and have many Triggers have more chances.

**Trait discovery** (acquiring entirely new Traits via combat, rather than just new Perks from existing Traits) is deferred to a later phase.

---

## Trainer Availability

- **Group-specific**: Each Group's trainers only teach Traits and Perks aligned with that Group's theme.
- **Generalist Groups**: Combat academies, gladiatorial trainers, and similar generalist Groups exist that teach diverse, broad, low-level basic Traits. These provide baseline access without requiring membership in a specialized Group.

---

## Trait Slot Constraints

- Traits can only be acquired into **empty slots**. No overflow, no queue, no replacement shortcut.
- If all slots in a category are full, the player must respec an existing Trait to make room before acquiring a new one.

---

## Decisions

### Three-Category Trait System

- **Decision**: Traits are divided into Core (identity), Role (capability), and Bond (affiliation) categories with separate slot pools.
- **Rationale**: Creates character identity along three axes. Prevents "all combat" builds by requiring investment across identity, capability, and affiliation.
- **Implications**: Character generation must produce Traits from all three pools. Each category needs enough content for meaningful choice at every star level.

### Perk as Multi-Component Package

- **Decision**: A single Perk can contain any mix of Actions, Stat Adjustments, and Triggers rather than being one component type.
- **Rationale**: Enables rich, thematic abilities (Flame Aura has an active component, a passive bonus, AND a reactive trigger). Reduces Perk count while increasing Perk depth.
- **Implications**: Perk balance must consider the combined value of all components. UI must display multi-component Perks clearly.

### Trait Level Amplifies All Child Perks

- **Decision**: Leveling a Trait (1–5★) amplifies ALL Perks within that Trait via a per-level multiplier (slightly accelerating: ×1.0 / ×1.2 / ×1.4 / ×1.7 / ×2.0). Leveling a specific Perk (1–5★) amplifies only that Perk. Both stack multiplicatively.
- **Rationale**: Creates a "wide vs. deep" investment choice — level the Trait for broad improvement, or level specific Perks for focused power. Accelerating curve rewards full commitment.
- **Implications**: Balance must account for double-stacking (high Trait level × high Perk level). Exact multiplier values are tuning — deferred to combat spec.

### Fixed Escalating XP Schedule

- **Decision**: Both Traits and Perks use the same XP cost schedule: 0→1★ = 100, 1→2★ = 300, 2→3★ = 600, 3→4★ = 1000, 4→5★ = 1500 XP. Total to max (0→5★) = 3500 XP.
- **Rationale**: Escalating costs create natural diminishing returns at higher levels. Shared schedule between Traits and Perks keeps the system simple. Combined with trainer service fees (gold), creates dual-currency pressure.
- **Implications**: Economy spec must define trainer service fee scaling. A character needs 3500 XP to max one Trait + 3500 per Perk — full investment in one 5-Perk Trait costs ~21,000 XP.

### Cross-Category Perk Sharing — No Stack

- **Decision**: Identical Perks can appear in different Traits across categories. When a character has the same Perk from two sources, it does not stack — the second source provides redundant access only.
- **Rationale**: Redundant access is insurance: if you respec away one Trait, you keep the Perk from the other. No stacking prevents degenerate double-dipping builds.
- **Implications**: Players may intentionally overlap Perks across Traits for safety, but there is no power incentive to do so.

### Starter Perk Auto-Grant

- **Decision**: Every Trait has exactly one designated Starter Perk. This Perk is auto-granted (free, no XP cost) when the Trait is acquired.
- **Rationale**: Ensures every Trait is immediately useful upon purchase — no Trait is a blank slate requiring additional XP investment before it does anything. The Starter Perk is the Trait's "signature move."
- **Implications**: Trait design must always include a compelling Starter Perk. The Starter Perk defines the Trait's first impression and basic capability.

### Expensive Respec

- **Decision**: Removing Traits is an expensive vendor service from a Group. All XP invested in the removed Trait and its Perks is lost. Gold cost to the Group — formula deferred to [economy](economy.md) spec.
- **Rationale**: Encourages roster turnover — it's often better to hire a new character than endlessly respec an existing one. Creates attachment to characters and investment in hiring decisions.
- **Implications**: Players need enough income to hire replacements. The meta shifts toward roster breadth over single-character perfection.

### Five Resource Families

- **Decision**: Five Resource Families (Arcane/Mana, Divine/Faith, Primal/Spirit, Psychic/Focus, Martial/Stamina) with attribute-derived pool formulas. Each Trait belongs to at most one family. Multiple Traits in the same family share one pool.
- **Rationale**: Five families cover the major fantasy archetype spaces. Shared pools within a family reward specialization (deeper investment = larger pool) without creating overwhelming resource tracking. Martial reuses Stamina to avoid a redundant pool.
- **Implications**: Combat spec must implement resource pool formulas and scaling. Characters can have at most 5 non-universal resource pools (one per family). Most characters will have 1–2.

### Species as Core Traits

- **Decision**: Species/ancestry are defined entirely via Core Traits with anatomy modification support. Multiple ancestry Traits are stackable. Conflict resolution uses per-slot maximum. Default humanoid anatomy for characters with no ancestry Traits.
- **Rationale**: Maximally extensible — new species are just new Core Traits. Supports hybrid ancestries naturally. No hardcoded species list means content can grow without schema changes.
- **Implications**: Core Trait definitions must specify any anatomical slot modifications. Equipment system must handle variable anatomy.

### Tag-Driven Synergies

- **Decision**: All cross-Trait synergies are emergent via shared tags. No explicit set bonuses between specific Traits. Tags live on Perks/Actions; Traits derive tags from the union of their Perks.
- **Rationale**: Tag-driven synergies are open-ended and scale with content — new Traits automatically interact with existing ones through shared tags. Explicit set bonuses would require manual curation and create a combinatorial explosion as content grows.
- **Implications**: Tag vocabulary must be well-curated to enable meaningful but not overpowered synergies. Equipment and consumable specs share the same tag vocabulary.

### Perk Level Cap by Trait Level

- **Decision**: A Perk's level cannot exceed its parent Trait's level. Exception: Perks acquired via Perk Discovery override this cap.
- **Rationale**: Creates a natural investment order — Trait first (broad power), then Perks (specific power). Prevents characters from having max-level Perks in a low-level Trait. Discovery exception rewards the rarity of the event.
- **Implications**: UI must communicate the cap clearly. Leveling a Trait unlocks higher Perk level ceilings for all its Perks.

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

- **Decision**: ~0.1% chance per qualifying Perk component use (Actions and Triggers only — not passive Stat Adjustments) to discover a random unowned Perk from the same Trait's tree. Acquired at minimum star level, free of XP cost. Discovery roll is rarity-weighted (lower-minimum-star Perks more likely). Discovered Perks override the Perk level cap.
- **Rationale**: Rewards continued engagement with existing Perks. Creates exciting surprise moments. The low rate prevents it from undermining the trainer economy. Rarity weighting makes discovery of powerful Perks appropriately rare.
- **Implications**: Combat spec must trigger Perk discovery checks on Action use and Trigger fire events. Economy impact is minimal at 0.1% but should be monitored.

---

## Open Questions

All structural and mechanical questions are resolved. Two non-blocking items are deferred:

1. **MVP content examples** (deferred to implementation): What are the specific ~3 Core, ~4 Role, ~2 Bond Traits with their 3–5 Perks each? This is content authoring, not design.
2. **Combat-triggered Trait discovery** (deferred to later phase): Can characters acquire entirely new Traits (not just Perks) through combat events?

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [characters](characters.md) | Trait Slot count defined by Character Star Rating. Core Traits may modify anatomical slots. Species = Core Traits pattern (no hardcoded species list). |
| [combat](combat.md) | Perks provide Actions (combat abilities), Stat Adjustments (combat stats), and Triggers (combat reactions). Resource Family pool formulas must be implemented (Mana, Faith, Spirit, Focus pools). Resource pools start at full capacity + slow per-tick regen (formula owned by combat). Per-stat scaling multipliers for resource pools. Trait level amplification multipliers (tuning values). Perk Discovery trigger specifics: rolls on Action use and Trigger fire events (not passive Stat Adjustments), rarity-weighted. |
| [equipment](equipment.md) | Equipment can require specific Traits/Perks. Affix bonuses can enhance specific Perks. Tags create Perk–equipment synergies (shared tag vocabulary). Variable anatomy from ancestry Core Traits affects available slots. |
| [consumables](consumables.md) | Specialist Perks (Bomber, Apothecary) amplify consumable effectiveness. Scribe Perk enables Scroll creation from known Perks. Tags create Perk–consumable synergies. |
| [combat-ai](combat-ai.md) | Personality Core Traits may modify AI behavior. Action priority lists reference Perk-granted Actions. |
| [economy](economy.md) | Trainer service fees (gold) for Trait/Perk leveling. Respec gold cost formula. XP cost schedule (100/300/600/1000/1500) interacts with economy balance. Group-specific trainer fee model (specialist vs. generalist pricing differences). |
| [groups](groups.md) | Trait generation loot table definitions per Group/archetype. Respec is a Group vendor service. Trainer services for Trait/Perk acquisition and leveling. Bond Traits map to specific Groups. Trainer availability model: Group-specific theme alignment, generalist Groups for baseline access. Bond Trait hybrid scaling: smooth benefits (price, breadth) + discrete tier unlocks (exclusive items at 3★, unique services at 5★). |
| [roster-management](roster-management.md) | Trait composition drives character hiring value. Respec is a vendor service in the economy. Starter Perks affect character readiness at recruitment. |
| [data-model](../architecture/data-model.md) | Trait/Perk data shape definition needed: Trait → Perk tree, resource family membership, tag lists, loot table weights. |

---

_Last updated: 2026-02-14 — Interrogation round 2: requirement checking, perk scaling/caps/prerequisites, pool mechanics, perk discovery triggers, trainer availability, bond trait hybrid scaling, trait slot constraints, cross-trait tags-only interaction_
