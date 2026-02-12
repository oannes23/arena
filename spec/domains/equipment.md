# Equipment — Domain Specification

**Status**: 🟡 In progress
**Last interrogated**: —
**Last verified**: —
**Depends on**: [characters](characters.md), [traits-and-perks](traits-and-perks.md)
**Depended on by**: [economy](economy.md)

---

## Overview

Equipment is the gear system — weapons, armor, and accessories that characters equip to enhance their combat capabilities. Each character has 5 Equipment Slots and a set of Anatomical Slots determined by their ancestry. Equipment provides base stats, quality-scaled effectiveness, and Diablo-style affixes for build customization. Equipment degrades over time, creating a permanent economic sink.

---

## Core Concepts

### 5-Item Limit with Anatomical Constraints

**Core Rule**: A character can equip up to **5 items**. Each item occupies one or more **Anatomical Slots**. The 5-item limit constrains loadout breadth; Anatomical Slots prevent illogical stacking (two helmets, three boots).

**Multi-Slot Items**: A single item can occupy multiple Anatomical Slots but counts as only 1 of the 5 Equipment Slots.
- Example: Plate Armor occupies Head + Torso + Arms + Legs = 1 equipment slot
- **Advantage**: Powerful consolidated bonuses, fewer pieces to manage
- **Disadvantage**: Fewer total affix sources (5 affixes from 5 items vs. more from separate pieces)
- **Balance**: Multi-slot items have higher base stats but same affix budget as single-slot items of equivalent quality

### Quality System (0–500+)

Quality is a numeric scale that acts as a percentage modifier to item effectiveness:

| Quality Range | Star | Color | Modifier |
|---------------|------|-------|----------|
| 0–99 | 0★ | Gray | Below baseline |
| 100 | 1★ | White | 1.0× (baseline) |
| 200 | 2★ | Green | 2.0× |
| 300 | 3★ | Blue | 3.0× |
| 400 | 4★ | Purple | 4.0× |
| 500+ | 5★ | Orange | 5.0×+ |

**Quality Effects**:
- Directly multiplies weapon damage and armor protection
- Example: 60 base damage weapon at 150 quality = 90 effective damage (60 × 1.5)
- Enhances positive item traits, reduces negative ones
- Example: High-quality warhammer has less Speed penalty than low-quality one

### Durability & Degradation

- **Current Quality**: Drops during combat use (wear and tear)
- **Max Durability**: Equals initial quality rating
- **Repair**: Restores current quality to (max − 1). Each repair permanently reduces max by 1.
- **Breaking**: Small chance based on wear ratio (current/max quality). Lower ratio = higher break chance.
- **Quality Forging**: Expensive vendor service to increase max quality (counteracts degradation but net-negative long-term)

**Design Intent**: Gear is a consumable resource on a long timeline. Even the best equipment eventually degrades and must be replaced. Permanent gold sink keeps the economy flowing.

### Affix System (Diablo-Style)

**Budget by Quality Tier**:

| Tier | Max Affixes | Per-Affix Power |
|------|------------|----------------|
| 1★ White | 1 | 1.4× |
| 2★ Green | 2 | 1.0× each |
| 3★ Blue | 3 | 0.7× each |
| 4★ Purple | 4 | 0.6× each |
| 5★ Orange | 5 | 0.5× each |

**Design Intent**: Lower-tier items with fewer affixes have individually stronger affixes. A White item with one affix at 1.4× can beat an Orange item with five at 0.5× *for a specific build*. Genuine decisions at every tier.

**Affix Types**:
- Attribute bonuses (+Strength, +Agility, etc.)
- Damage type bonuses (+Fire damage, +Cold resistance)
- Special effects (life steal, chance to stun, proc effects)
- Perk bonuses (improves specific abilities — requires corresponding Perk)
- Affix potency also modified by item quality

**Rare Affixes** (5★ items only): "Mythic", "Legendary", "Divine" tier affixes with uniquely powerful effects.

### Tag System

Tags are shared mechanical keywords connecting weapons, armor, consumables, and Perks.

**Weapon Tags**:

| Tag | Examples | Effect |
|-----|----------|--------|
| Reach | Glaives, spears, staves | Attack Medium range from Short; Polearm Master synergy |
| Two-Handed | Greatswords, warhammers, longbows | Both hand slots; higher base damage; no shield |
| Light | Daggers, shortswords, hand axes | Faster attack speed; reduced Strength req; dodge bonus |
| Heavy | Greataxes, mauls, warhammers | High damage + armor pen; Speed penalty; Strength req |
| Finesse | Rapiers, daggers, whips | Agility for damage; crit bonus; Trickster/Duelist synergy |
| Ranged | Bows, crossbows, slings | Attack Long range; Agility accuracy; Short range penalty |
| Thrown | Javelins, throwing axes | Limited ammo; counts as melee and ranged |
| Versatile | Longswords, spears, axes | One-hand or two-hand; better stats two-handed |
| Arcane Focus | Wands, orbs, staves | Required for certain spells; spell power bonus |
| Channeling | Staves, rods, tomes | Reduces casting time; enhances sustained spells |

**Armor Tags**:

| Tag | Examples | Effect |
|-----|----------|--------|
| Light Armor | Leather, cloth, hide | No Agility penalty; low protection; rogues/casters |
| Medium Armor | Chainmail, scale, brigandine | Moderate protection; small Agility penalty |
| Heavy Armor | Plate, full mail | High protection; Agility/dodge penalty; Strength req |
| Shield | Bucklers, tower shields | Off-hand; block/reduction; defensive Perk synergy |

### Equipment Requirements

Items can require:
- **Attributes**: Minimum Strength, Agility, etc.
- **Traits**: Must possess a specific Trait (e.g., "Requires Arcane Affinity")
- **Perks**: Must know a specific Perk

Prevents equipping mismatched gear and creates meaningful gating for powerful items.

---

## Decisions

### 5-Item Limit (Not Slot-Per-Body-Part)

- **Decision**: Maximum 5 equipped items total, with Anatomical Slots preventing illogical stacking, rather than one-item-per-slot.
- **Rationale**: Creates interesting multi-slot vs. single-slot trade-offs. Plate Armor (1 item, 4 anatomical slots) vs. separate helmet + chest + gauntlets + greaves (4 items, same slots). Budget of affixes vs. base stat consolidation.
- **Implications**: Equipment UI must clearly show both Equipment Slot usage (X/5) and Anatomical Slot coverage.

### Quality as Linear Multiplier

- **Decision**: Quality 100 = 1.0× baseline; quality 500 = 5.0× (linear scaling).
- **Rationale**: Simple, legible. Players instantly understand the power difference between tiers.
- **Implications**: Creates enormous gear gap between tiers (5× at Legendary vs. Common). May need revision to diminishing returns if gear overshadows character builds — flagged as open question.

### Inverse Affix Budget

- **Decision**: Fewer affixes = higher per-affix power (1★ at 1.4× vs. 5★ at 0.5×).
- **Rationale**: Prevents strict "higher tier = always better" hierarchy. Builds that only need one specific affix may prefer lower-tier items. Creates itemization decisions at every tier.
- **Implications**: Build diversity increases. "Best in slot" becomes build-dependent rather than tier-dependent.

### Gear Degradation as Long-Term Sink

- **Decision**: Equipment degrades permanently (repair reduces max by 1). Even the best gear eventually needs replacement.
- **Rationale**: Prevents economic stagnation. Players always need gold for gear maintenance/replacement. Creates attachment/loss dynamics.
- **Implications**: Degradation rate is a critical tuning parameter. Too fast = punishing; too slow = no pressure.

---

## Open Questions

1. Quality scaling: linear (current 5× at 500) or diminishing returns (e.g., sqrt scaling: 400 = 2× instead of 4×)? Critical for gear-vs-build balance.
2. Multi-slot item power balance: proportionally stronger base stats, or just simpler? If a Plate Armor has 4× the base stats of a helmet, is that balanced against losing 3 Equipment Slots worth of affixes?
3. Loot drop rate curves: how often do different quality tiers drop?
4. Quality forging cost scaling: linear, exponential, or bracket-based?
5. Degradation rate: per-fight, per-hit, or per-action? How many fights before a Common item breaks?
6. For an idle game, does relentless degradation feel punishing? Should quality floors exist (gear can't degrade below star tier baseline)?
7. Minimum weapon/armor types needed for MVP?
8. Does affix re-rolling preserve item quality, or is there risk of quality loss?

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [characters](characters.md) | Anatomical Slot availability determines equippable items. Attribute values gate equipment requirements. |
| [traits-and-perks](traits-and-perks.md) | Traits/Perks can be equipment requirements. Affixes can enhance specific Perks. Tags create Perk synergies. |
| [combat](combat.md) | Weapons provide Attack/Damage values. Armor provides Defense/Soak. Tags determine range capabilities and combat interactions. |
| [consumables](consumables.md) | Equipment and consumables share the Tag vocabulary. Affixes may interact with consumable effectiveness. |
| [economy](economy.md) | Equipment acquisition (loot, crafting, market), repair, quality forging, and affix manipulation are major gold sinks. |

---

_Last updated: 2026-02-11_
