# Groups — Domain Specification

**Status**: 🟡 In progress
**Last interrogated**: —
**Last verified**: —
**Depends on**: [characters](characters.md), [traits-and-perks](traits-and-perks/index.md), [economy](economy.md)
**Depended on by**: [equipment](equipment/index.md), [consumables](consumables.md), [roster-management](roster-management.md), [tournaments](tournaments.md)

---

## Overview

A Group is any named organization in the city — guilds, temples, taverns, factions, mercenary companies, the Arena office, monasteries, etc. Groups are the unified entity behind what might otherwise be called "vendors," "recruiters," "trainers," or "factions." Each Group offers a menu of services, some public (available to anyone) and some member-only (requiring a Bond Trait connection).

Bond Traits are the access mechanism: a character with a Bond Trait for a Group gains membership and unlocks that Group's private services. Groups are where players spend most of their gold and interact with the city outside of combat.

---

## Core Concepts

### Groups as Service Providers

Every Group is defined by its **service menu** — a list of services it offers. Services are drawn from a set of service types:

| Service Type | Description | Examples |
|---|---|---|
| **Vendor** | Buy/sell goods. Each Group defines its own loot tables and pricing. | Weapon shop, potion ingredients, holy relics |
| **Training** | Improve character attributes, traits, or perks. | Attribute trainers, Perk unlock mentors |
| **Recruitment** | Hire new characters. Each Group defines its own themed archetypes. | Guild initiates, tavern drifters, arena fighters |
| **Spellcasting** | Cast spells or rituals as a service. | Healing, resurrection, enchanting, identification |
| **Crafting** | Produce items using recipes/materials. | Smithing, alchemy, scribing |
| **Quests** | Offer tasks for rewards (future system). | Bounties, errands, storyline missions |

A single Group can offer multiple service types. A temple might offer spellcasting (healing, resurrection), vendor (holy items), training (faith-related perks), and recruitment (acolytes, paladins).

### Public vs. Member Services

- **Public services**: Available to any character. Typically basic offerings at standard prices.
- **Member services**: Require a Bond Trait connecting the character to this Group. Unlock exclusive inventory, better prices, specialized training, unique recruitment archetypes, and powerful services.

Membership depth could scale with Bond Trait star level — a 1★ Bond gets basic member access while a 5★ Bond gets the full catalogue.

### Groups and Recruitment Archetypes

Groups that offer recruitment define their own themed archetypes (see [characters.md](characters.md) — Character Generation Model). Each archetype specifies attribute bumps/dumps, likely trait pools, and equipment themes. This creates meaningful differentiation between recruitment sources — players learn which Groups produce which kinds of fighters.

Examples:
- *Necromancer's Guild*: Death Knight, Necromancer, Grave Warden, Soul Reaper
- *Arena Recruitment Center*: Pit Fighter, Arena Brawler, Crowd Pleaser
- *Tavern*: Wanderer, Sellsword, Drifter
- *Temple of Tharzul*: Crusader, War Priest

Major Groups offer 4–6 archetypes; minor Groups offer 1–2.

### Groups and Bond Traits

Each Group has a corresponding Bond Trait (or set of Bond Traits). The Bond Trait:
- Grants membership and unlocks member services
- May provide passive bonuses related to the Group's theme
- Has a Perk tree that deepens the character's connection

A character's Bond Trait slots (count = Star Rating) limit how many Groups they can be affiliated with.

---

## Decisions

### Unified Entity Model

- **Decision**: Guilds, temples, taverns, factions, and all other city organizations are a single entity type called a "Group." There is no separate "vendor" or "recruiter" concept — these are service types a Group can offer.
- **Rationale**: Simplifies the data model. One entity type with a composable service menu replaces multiple overlapping concepts. Players interact with Groups, not with abstract system categories.
- **Implications**: All specs that reference "vendors" should be understood as referring to a Group's vendor service. Economy spec defines pricing and transaction mechanics; Groups spec defines who offers what.

### Service Access via Bond Traits

- **Decision**: Bond Traits are the membership mechanism. Public services exist without membership; member services require the corresponding Bond Trait.
- **Rationale**: Ties the Group system directly to character builds. Choosing which Groups to affiliate with is a meaningful roster decision — Bond Trait slots are limited by Star Rating.
- **Implications**: Traits-and-perks spec must define Bond Traits per Group. Each Bond Trait's Perk tree should include service-related benefits (better prices, exclusive items, etc.).

---

## Open Questions

1. How many Groups exist in the starting city? What's the right density for MVP?
2. Can Group reputation change over time (e.g., faction standing, favor)?
3. Do Groups have relationships with each other (rivalries, alliances) that affect gameplay?
4. How does Bond Trait star level scale access to member services?
5. Can Groups be discovered/unlocked, or are all available from the start?
6. How do Group loot tables / vendor inventories rotate or refresh?
7. What defines a Group's "tier" — does it affect service quality and recruit star ratings?

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [characters](characters.md) | Group-specific recruitment archetypes are defined per Group, not globally. |
| [traits-and-perks](traits-and-perks/index.md) | Bond Traits map to Groups. Each Group needs at least one Bond Trait defined. Perk trees should include service access benefits. |
| [combat](combat/index.md) | Some Groups may offer pre-combat buff services or post-combat healing. |
| [equipment](equipment/index.md) | Equipment is bought/sold through Group vendor services. Group identity determines available loot tables. |
| [consumables](consumables.md) | Consumables purchased/crafted through Group services. |
| [economy](economy.md) | Groups are the primary gold sinks (services cost gold). Pricing mechanics, vendor refresh cycles, and transaction rules live in economy spec. |
| [roster-management](roster-management.md) | Recruitment happens through Groups. Group access affects roster-building strategy. |
| [tournaments](tournaments.md) | The Arena itself may be a Group with its own services and Bond Trait. |

---

_Last updated: 2026-02-11_
