# Consumables — Domain Specification

**Status**: 🟡 In progress
**Last interrogated**: —
**Last verified**: —
**Depends on**: [traits-and-perks](traits-and-perks/index.md)
**Depended on by**: [economy](economy.md)

---

## Overview

Consumables are single-use items brought into battle — potions, bombs, and scrolls. Each character has 5 Consumable Slots separate from their Equipment Slots. Consumables are designed to be low-impact at baseline but powerful when a build invests in specialist Perks (Bomber, Apothecary, etc.). They share the Tag vocabulary with equipment for Perk synergies.

---

## Core Concepts

### 5 Consumable Slots

Each character brings up to 5 consumables into battle. Consumed on use (destroyed). Separate resource from Equipment Slots.

### Three Consumable Types

| Type | Speed | Reliability | Power | Cost |
|------|-------|-------------|-------|------|
| **Potion** | Fast (minimal Initiative penalty) | Guaranteed success | Moderate cap | High |
| **Bomb** | Fast (minimal Initiative penalty) | Guaranteed success | Offensive (damage, status, AoE) | Moderate |
| **Scroll** | Slow (high Initiative penalty) | Failure chance (check required) | No ceiling | Low |

**Potions**: Healing, buffs, restoration. Recipes define effects (e.g., Minor Healing Potion, Major Healing Potion, Vigor Potion). Reliable but capped in power.

**Bombs**: Damage, status application, AoE effects. Tags define damage types and targeting. Fast offensive option.

**Scrolls**: Can replicate any Perk effect. Slow and unreliable but cheap and uncapped. High-level effects MUST use scrolls since potion costs scale prohibitively. Created by characters with Scribe Perk from any Perk they possess.

### Vendor Services (Pre-Combat Only)

Not carried into battle. Purchased before a fight for one-time effects:
- Even cheaper than scrolls
- No Perk requirements
- Examples: pre-fight healing, temporary buffs

### Specialist Perk Synergies

Consumables have small baseline impact UNLESS a build specializes:
- **Bomber** Perk: Drastically increases bomb effectiveness
- **Apothecary** Perk: Improves potion potency
- **Scribe** Perk: Creates scrolls from known Perks
- Tags on consumables hook into Trait/Perk synergies (e.g., Fire Bomb benefits from Pyromancer Perks)

### Acquisition

- **Crafting**: Requires specific Traits/Perks (Alchemist creates potions/bombs, Scribe creates scrolls)
- **Vendor purchases**: Buy from shops
- **Loot drops**: From combat victories
- **No expiration**: Consumables last indefinitely once acquired

---

## Decisions

### Small Baseline, Strong with Investment

- **Decision**: Consumables are low-impact by default. Specialist Perks amplify them significantly.
- **Rationale**: Prevents consumable spam from dominating combat. Makes consumable builds a genuine investment (Trait/Perk slots for specialist abilities) rather than a universal power boost.
- **Implications**: Consumable balance has two axes: baseline power (for non-specialists) and amplified power (for specialists). Both must be tuned independently.

### Scrolls as Cheap, Unreliable, Uncapped

- **Decision**: Scrolls are the only consumable type with no power ceiling but have failure chance and slow speed.
- **Rationale**: Creates a unique niche — powerful effects at low gold cost but high risk. The Scribe Perk (creating scrolls from known Perks) becomes very valuable for high-level strategies.
- **Implications**: Scroll failure chance is a key balance lever. Too reliable = always use scrolls; too unreliable = never use them.

### Separate from Equipment

- **Decision**: Consumable Slots are a distinct resource from Equipment Slots.
- **Rationale**: Different lifecycle (consumed vs. degrades), different acquisition paths, different build considerations. Merging them would force equipment/consumable trade-offs that don't make thematic sense.
- **Implications**: 5 Equipment + 5 Consumable = 10 total "loadout" slots to configure per character.

---

## Open Questions

1. Consumable quality levels: scaffolded in design but not specified. Does quality affect potency, success chance, or both?
2. Crafting recipe acquisition: how do you learn new recipes? (Loot drops, vendor purchase, research, or auto-unlock from Traits?)
3. Specific AI consumable trigger configuration: how granular are the trigger conditions? (HP%, turn count, enemy count, status stacks?)
4. Scroll failure chance formula: based on what? (Character Willpower? Scroll complexity? Flat per-tier?)
5. Can consumables be used outside of combat? (Healing potions between fights, buff scrolls before entering arena?)
6. Consumable stacking: can you bring 5 of the same potion, or must each slot be a different item?
7. Bomb AoE targeting: does the bomb hit an entire zone, or does it use the per-action targeting flags?

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [traits-and-perks](traits-and-perks/index.md) | Specialist Perks (Bomber, Apothecary, Scribe) amplify consumable power. Scribe creates scrolls from known Perks. |
| [combat](combat/index.md) | Consumables are combat actions with Initiative costs. Status effects from bombs/scrolls use the stack-based system. |
| [combat-ai](combat-ai.md) | AI needs trigger conditions for consumable usage (HP thresholds, turn count, etc.). |
| [equipment](equipment/index.md) | Shared Tag vocabulary enables cross-system Perk synergies. |
| [economy](economy.md) | Consumable crafting, purchasing, and usage are gold sinks. Alchemist/Scribe support staff affect production costs. |

---

_Last updated: 2026-02-11_
