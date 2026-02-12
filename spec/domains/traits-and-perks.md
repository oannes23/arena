# Traits and Perks — Domain Specification

**Status**: 🟡 In progress
**Last interrogated**: —
**Last verified**: —
**Depends on**: [characters](characters.md)
**Depended on by**: [combat](combat.md), [equipment](equipment.md), [consumables](consumables.md), [roster-management](roster-management.md)

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

- **Minimum Star Level**: Each Trait has a minimum purchase level (1–5★).
- **Leveling**: All Traits can be leveled up to 5★ regardless of minimum.
- **Trait Level Effects**: Modifies ALL Perks within that Trait (higher Trait level = stronger Perks).
- **Requirements**: Can require specific Attributes, other Traits/Perks, derived stats, or achievements.
- **Cross-Category Sharing**: Identical Perks can appear in different Traits across different categories.
- **Respeccing**: Expensive vendor service to remove Traits — encourages roster turnover over endlessly iterating one character.

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

- **Requirements**: Can require Attributes, Traits, other Perks, derived stats, achievements.
- **Minimum Star Level**: 1–5★ minimum purchase level.
- **Leveling**: All Perks can level to 5★.
- **Perk Level Effects**: Modifies only that specific Perk (independent of Trait level, but stacks with it).
- **Acquisition**: Purchase from Trainers using XP (must own the parent Trait first).
- **Auto-Granted**: Some Perks automatically granted when acquiring their parent Trait.

### Role Trait Dual Purpose

Characters with support Role Traits can fight AND provide household support:
- A Blacksmith Trait includes combat Perks (Strength boost, Smith's Strike) AND support Perks (craft weapons/armor).
- Same character fights in tournaments and provides crafting bonuses between fights.

### Bond Trait Vendor Access

Bond Traits provide vendor access as progression:
- Higher Bond Trait levels = better vendor access, pricing, and exclusive items.
- Bonds unlock themed equipment, consumables, training, and services.

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

- **Decision**: Leveling a Trait (1–5★) amplifies ALL Perks within that Trait. Leveling a specific Perk (1–5★) amplifies only that Perk. Both stack multiplicatively.
- **Rationale**: Creates a "wide vs. deep" investment choice — level the Trait for broad improvement, or level specific Perks for focused power.
- **Implications**: Balance must account for double-stacking (high Trait level × high Perk level). XP cost scaling for Trait leveling vs. Perk leveling is a key tuning lever.

### Cross-Category Perk Sharing

- **Decision**: Identical Perks can appear in different Traits across categories.
- **Rationale**: Allows thematic overlap (a Warrior's "Power Attack" and a Berserker's "Power Attack" can be the same Perk). Prevents siloing.
- **Implications**: If a character has the same Perk from two sources, stacking rules must be defined. (Most likely: doesn't stack, just provides redundant access.)

### Expensive Respec

- **Decision**: Removing Traits is expensive (vendor service). This is intentional.
- **Rationale**: Encourages roster turnover — it's often better to hire a new character than endlessly respec an existing one. Creates attachment to characters and investment in hiring decisions.
- **Implications**: Players need enough income to hire replacements. The meta shifts toward roster breadth over single-character perfection.

---

## Open Questions

1. Minimum number of Traits per category needed for MVP? (Content volume question — how many Core/Role/Bond Traits for meaningful choice?)
2. Minimum number of Perks per Trait? (Determines Trait tree depth and XP investment per Trait.)
3. Cross-category Trait synergies: Emergent only (from shared tags/Perks), or explicit bonus interactions ("if you have Trait X and Trait Y, gain Z")?
4. Stacking rules when the same Perk is available from multiple Traits: No stack (redundant access only)? Additive? Something else?
5. XP cost scaling for Trait leveling vs. Perk leveling — linear or exponential?
6. Can a character have multiple Traits from the same "family" (e.g., two different ancestry Core Traits)? Or is ancestry exclusive?
7. What is the exact formula for how Trait level amplifies Perk power? (Linear multiplier? Percentage bonus?)
8. How are auto-granted Perks determined? (Always the first Perk in a tree? A specific tagged subset?)

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [characters](characters.md) | Trait Slot count defined by Character Star Rating. Core Traits may modify anatomical slots. |
| [combat](combat.md) | Perks provide Actions (combat abilities), Stat Adjustments (combat stats), and Triggers (combat reactions). Resource types (Mana, Faith) unlocked by specific Traits. |
| [equipment](equipment.md) | Equipment can require specific Traits/Perks. Affix bonuses can enhance specific Perks. Tags create Perk synergies. |
| [consumables](consumables.md) | Specialist Perks (Bomber, Apothecary) amplify consumable effectiveness. Scribe Perk enables Scroll creation from known Perks. |
| [combat-ai](combat-ai.md) | Personality Core Traits may modify AI behavior. Action priority lists reference Perk-granted Actions. |
| [roster-management](roster-management.md) | Trait composition drives character hiring value. Respec is a vendor service in the economy. |

---

_Last updated: 2026-02-11_
