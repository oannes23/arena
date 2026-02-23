# Roster Management — Domain Specification

**Status**: 🟡 In progress
**Last interrogated**: —
**Last verified**: —
**Depends on**: [characters](characters.md), [economy](economy.md), [traits-and-perks](traits-and-perks/index.md)
**Depended on by**: [tournaments](tournaments.md)

---

## Overview

Roster Management covers everything that happens to characters outside of combat — hiring, firing, injury, death, resurrection, metacurrency progression, and base building. This is the "between fights" management layer that gives the game its management sim identity. It includes all the systems that determine which characters you have, what condition they're in, and what account-level progression unlocks are available.

---

## Core Concepts

### Character Acquisition — Vendor Types

| Vendor | Character Quality | Cost | Notes |
|--------|------------------|------|-------|
| **Arena Recruitment Center** | Low-tier, blank slate (few/no starting Traits) | Cheap | Weighted toward lower stars; no starting equipment |
| **Guild Vendors** | Themed builds (Pyromancer Guild, Warrior's Academy) | Moderate–High | ≥1 guaranteed themed Trait; may have additional random themed Traits; often paired with themed equipment vendor |
| **Tavern** | Wide variety, pre-existing Traits/Perks/Equipment | Variable | Diverse random generation; cost based on total character value |
| **Specialist Vendors** | Themed to faction (temple, criminal, military) | Variable | Divine-themed, shadow/trickster-themed, warrior/defender-themed |

**Hiring Cost**: Vendor baseline + total character value (Attributes, Traits, Perks, equipment).

### Injury & Death System

**Defeat → Injury Check Flow**:
1. Character falls in combat (Health reaches 0)
2. End-of-fight injury resistance check
3. **Pass**: No injury
4. **Fail**: Injury severity based on degree of failure
5. Second check (difficulty based on injury severity)
6. **Pass**: Character survives with injury
7. **Fail**: Character dies

**Injury Types**:
- Persistent negative statuses (until healed)
- Lost limbs (permanent anatomical slot loss unless restored)
- Stat penalties (temporary or permanent)
- Time-limited injuries (heal naturally over several ticks)

**Recovery Options**:

| Method | Cost | Outcome |
|--------|------|---------|
| Natural Healing | Free (over ticks) | Time-limited injuries only |
| Medical Vendor | Gold | Consistent recovery for common injuries |
| Healing Perks | Character has Perk | Heal specific injury types |
| Severe Injury Specialists | High gold or Bond access | Requires Bond Trait connections or extreme markup |
| Raise Dead (Cheapest) | Great gold cost | Halve all Current Attributes, −1 to all Potentials |
| Raise Dead (Mid-tier) | Very high gold cost | Moderate attribute penalties |
| Raise Dead (Premium) | Extreme gold cost | No permanent penalties |

### Roster Economics

- **Upkeep**: Every character has ongoing upkeep cost deducted each tick
- **Over-hiring → Bankruptcy**: If gold runs out, most expensive contracts leave first
- **Released Characters**: Become free agents (available to other players in multiplayer)
- **Roster Size**: Base limit, expandable via base building

### Metacurrency & Backgrounds

**Metacurrency** — persistent account-level progression:

**Earned Through**:
- Achievements and milestones (primary source)
- Tournament victories
- Persistent play milestones
- Graceful retirement of veteran characters (one source among many, not primary)

**Spent On**:
- **Backgrounds**: Starter templates with pre-configured Trait packages
  - "Orc Warrior" → Orc ancestry Trait + combat Traits
  - "Noble Lineage" → Starting gold, reputation bonuses
  - "Battle-Scarred Veteran" → Combat Traits, injury resistances
- Premium character generation options
- Controlled power creep for recruit quality over time

### Base Building (Future System)

- **Facilities**: Barracks, training grounds, infirmary, forge, etc.
- **Upgrades**: Bonuses to roster size, training efficiency, healing speed, etc.
- **Lifestyle Levels**: Adjustable per-character (affects morale, recovery, upkeep cost)

---

## Decisions

### Vendor Variety for Character Acquisition

- **Decision**: Multiple vendor types with different character generation profiles (blank slates, themed builds, random variety, faction-specific).
- **Rationale**: Creates meaningful hiring decisions. Do you want a cheap blank slate to mold, or an expensive pre-built specialist? Different vendors reward different strategies.
- **Implications**: Each vendor type needs its own character generation tables. Guild vendors need content-rich themed Trait pools.

### Two-Step Injury/Death Check

- **Decision**: Defeat → injury check (severity) → death check (based on severity). Two separate rolls.
- **Rationale**: Graduation of consequences. Most defeats result in no injury or minor injury. Death is rare but impactful. Two rolls give more tuning knobs than a single combined check.
- **Implications**: Injury resistance and severity tables are key balance parameters. Exhibition matches can bypass or soften this system.

### Tiered Resurrection Costs

- **Decision**: Three tiers of Raise Dead with decreasing penalties at increasing cost.
- **Rationale**: Death is meaningful but not permanent unless the player chooses. Budget players accept severe penalties; wealthy players pay for clean resurrection. Creates economic decisions around character investment.
- **Implications**: Premium resurrection must be expensive enough that it's a real choice, not automatic. The cheapest tier (halve all Current, −1 Potentials) must be harsh enough that sometimes letting a character die is the rational choice.

### Metacurrency from Multiple Sources

- **Decision**: Metacurrency earned from achievements, milestones, tournaments, AND retirement — but retirement is one source among many, not the primary one.
- **Rationale**: Prevents "metacurrency farming" where players treat characters as disposable. Players should form attachment to characters while still having a retirement option.
- **Implications**: Achievement/milestone system needs enough depth to be the primary metacurrency source. Retirement bonus should be meaningful but not dominant.

### Bankruptcy: Most Expensive Leave First

- **Decision**: When gold runs out, characters with the highest upkeep contracts leave first.
- **Rationale**: Protects cheap/new characters. Expensive veterans leaving first is both thematically appropriate (they have better options) and strategically interesting (you lose your best first).
- **Implications**: Players must monitor upkeep-to-income ratio. Warning systems needed before bankruptcy hits.

---

## Open Questions

1. Base roster size limit? (How many characters before needing base building upgrades?)
2. Power ceiling: is there a maximum character power level, or infinite scaling? (Affects whether retirement is ever "necessary.")
3. Low-star character viability: any reason to field 1–2★ characters beyond cost savings? (If not, low-tier vendors become irrelevant mid-game.)
4. Metacurrency earn rates: how many achievements/milestones exist? How much metacurrency per source?
5. Horizontal vs. vertical late-game progression: do players scale existing characters higher, or diversify their roster wider?
6. Endgame content: what keeps players engaged after reaching high star ratings and quality equipment?
7. If metacurrency retirement is "one source among many," how much should it reward relative to achievements? (10%? 25%? 50% of total metacurrency income?)
8. Base building scope: how many facilities? How impactful are upgrades? (Risk of scope creep.)
9. Can released characters (from bankruptcy or voluntary release) be re-hired? At what cost?
10. Lifestyle levels: how do they interact with morale and performance? (Ties to the Phase 4 morale system.)

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [characters](characters.md) | Character generation rules define what vendors produce. Injury modifies Attributes and anatomical slots. |
| [economy](economy.md) | Hiring, upkeep, healing, resurrection, and retirement are major gold sinks. Metacurrency is a separate economy. |
| [traits-and-perks](traits-and-perks/index.md) | Vendor-generated characters come with pre-built Traits. Respec is an expensive vendor service. |
| [tournaments](tournaments.md) | Injured/dead characters may be unavailable for tournaments. Roster depth determines tournament participation capacity. |
| [combat](combat/index.md) | Injury outcomes depend on combat defeat conditions. Character availability for future fights depends on injury/death status. |

---

_Last updated: 2026-02-11_
