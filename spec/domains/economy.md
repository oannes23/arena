# Economy — Domain Specification

**Status**: 🟡 In progress
**Last interrogated**: —
**Last verified**: —
**Depends on**: [characters](characters.md)
**Depended on by**: [roster-management](roster-management.md), [tournaments](tournaments.md)

---

## Overview

The economy governs resource flows — gold, XP, and time. This domain includes the tick-based time model (which drives upkeep, vendor turnover, passive income, and healing), vendor services, and the gold/XP systems. Time and economy are inseparable because ticks drive upkeep deductions, vendor inventory rotation, and passive income generation.

---

## Core Concepts

### Time Model — Ticks

The game advances in discrete **ticks** — periodic update cycles where passive systems resolve.

**Single-Player**: Ticks advance on player command (button press). Play at your own pace.

**Multiplayer**: Ticks are real-time on a server schedule (configurable — hourly, every 15 minutes, etc.).

**Each Tick Resolves**:
- Passive resource generation (gold income from Support Staff, etc.)
- Vendor inventory turnover (new items, characters available for hire)
- Natural healing (injuries recover over time)
- Upkeep deduction (character salaries, facility costs)
- Tournament scheduling

**Between Ticks**, players can:
- Spend gold for immediate healing at temples
- Train characters (spend XP on Attributes and Perks)
- Buy/sell equipment and consumables
- Manage roster (hire, fire, reassign)
- Run PvE encounters (on-demand combat for loot, XP, gold)
- Register characters for upcoming tournaments

**Design Intent**: Natural idle loop. Check in → review results → make decisions → queue actions → leave. Complexity in decisions between ticks, not during combat.

### Gold Economy

**Income Sources**:
- Victory rewards (fights, tournaments, PvE encounters)
- Passive income from Support Staff (Accountant/Merchant)
- Selling equipment and consumables

**Expense Sinks**:
- Character upkeep (per-tick salary per character)
- Hiring new characters
- Equipment repair, quality forging, affix manipulation
- Consumable purchases and crafting materials
- Healing services (temples, medics)
- Training costs (Perk acquisition, Attribute training)
- Respec services (expensive Trait removal)
- Raise Dead services (scaling cost tiers)

**Bankruptcy**: If gold runs out, most expensive contracts leave first. Over-hiring is risky.

### XP Economy

- Earned from: combat victories, PvE encounters, tournaments
- Spent on: Training Attributes toward Potential, purchasing Perks from Trainers, acquiring new Traits
- Per-character resource (not pooled across roster)

### Vendor Services

| Service | Provider | Function |
|---------|----------|----------|
| Perk Training | Trainers | Spend XP to purchase Perks from owned Traits |
| Attribute Training | Trainers | Spend XP to increase Attributes toward Potential |
| Healing | Temples, Medics | Recover Health, cure injuries |
| Raise Dead | Temples | Resurrect dead characters (scaling cost/penalties) |
| Repair | Blacksmiths | Restore equipment quality (max − 1) |
| Quality Forging | Blacksmiths | Increase max quality (expensive) |
| Affix Manipulation | Enchanters | Re-roll, add, or remove affixes |
| Consumable Crafting | Alchemists | Create potions, bombs from recipes |
| Scroll Copying | Scribes | Create scrolls from known Perks |
| Respeccing | Specialist Vendors | Remove Traits (expensive) |

### Support Staff

Characters with support Role Traits provide passive bonuses to the household:

| Role | Benefit |
|------|---------|
| Blacksmith | Reduced repair/forging costs, equipment maintenance |
| Enchanter | Affix manipulation services, quality bonuses on crafted items |
| Trainer | Reduced XP costs for specific Trait families, advanced Perk access |
| Accountant/Merchant | Gold generation bonuses, market discounts |
| Quartermaster | Equipment upkeep reduction, inventory management |
| Medic/Priest | Faster injury recovery, reduced healing costs |

**Dual-purpose**: Same characters can fight AND provide support if their Perks allow both.

### PvE Encounters

On-demand combat available between ticks for grinding:
- Set difficulty tiers with scaling rewards
- Training/exhibition style — no injury risk (or reduced)
- Loot drops, XP, and gold rewards
- Predefined stages with themed opponents and loot tables
- Used to test builds before committing to tournaments

---

## Decisions

### Tick-Based Time Model

- **Decision**: All passive systems resolve on discrete ticks. Single-player ticks on command; multiplayer on server schedule.
- **Rationale**: Creates natural session boundaries. Supports idle play (things happen when you're away in multiplayer). Single-player mode lets players control pacing.
- **Implications**: All passive systems (upkeep, healing, income, vendor rotation) must be tick-based. Between-tick actions are unlimited.

### Upkeep as Core Pressure

- **Decision**: Every character has ongoing gold upkeep deducted each tick.
- **Rationale**: Prevents unlimited roster hoarding. Forces hiring/firing decisions. Creates tension between roster breadth and economic sustainability.
- **Implications**: Upkeep scaling must be tuned so players can maintain a competitive roster without constant anxiety. Bankruptcy mechanics need to feel fair (most expensive contracts leave first, not random).

### Dual-Purpose Support Staff

- **Decision**: Support staff can also be fighters. A Blacksmith who fights in tournaments still provides crafting bonuses between fights.
- **Rationale**: Avoids the "dead slot" problem where hiring a Blacksmith feels wasteful because they can't fight. Encourages diverse roster builds.
- **Implications**: Role Traits must include both combat and support Perks. Support bonuses should be meaningful even from low-star support Traits.

### Equipment Degradation as Primary Gold Sink

- **Decision**: Equipment permanently degrades (repair costs + max quality loss), creating the primary long-term gold sink.
- **Rationale**: Prevents economic stagnation after initial build completion. Players always need gold, maintaining economic pressure even at endgame.
- **Implications**: Degradation rate is the master tuning knob for economic pressure. Other sinks (training, healing, hiring) are secondary.

---

## Open Questions

1. Primary gold sink priority: is equipment degradation truly primary, or should upkeep/training/hiring be the main tension? All sinks should be tuned around the primary one.
2. XP cost scaling: linear or exponential? (Linear = predictable grind; exponential = soft ceiling on character power.)
3. Gold income relative to costs: what's the target ratio? (Should a player break even on upkeep, or need active play to stay solvent?)
4. Loot drop rate tuning: how often do different quality tiers drop from combat?
5. Leveling speed: how many battles per meaningful upgrade? (Too fast = no anticipation; too slow = frustration.)
6. Degradation feel: does it feel punishing for idle play, or manageable? Should there be a "minimum quality floor" per star tier?
7. Tick frequency in multiplayer: hourly, every 15 minutes, configurable per server?
8. PvE encounter availability: unlimited between ticks, or rationed per tick?
9. Vendor turnover rate: how often do vendor inventories refresh? Every tick? Randomly?
10. Power ceiling: is there a max character power, or infinite scaling? Affects long-term economy design.

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [characters](characters.md) | Character value (star rating, attributes, traits) determines hiring cost and upkeep. |
| [traits-and-perks](traits-and-perks.md) | Training costs (XP for Perks/Attributes) are a major XP sink. Support Role Traits provide economic bonuses. |
| [equipment](equipment.md) | Repair, forging, affix manipulation, and market transactions are primary gold sinks. |
| [consumables](consumables.md) | Crafting costs, vendor purchases, and recipe acquisition are ongoing gold sinks. |
| [roster-management](roster-management.md) | Hiring costs, upkeep, healing, and resurrection are major gold sinks. Bankruptcy triggers roster changes. |
| [tournaments](tournaments.md) | Tournament rewards are a primary income source. Reward scaling must balance against costs. |

---

_Last updated: 2026-02-11_
