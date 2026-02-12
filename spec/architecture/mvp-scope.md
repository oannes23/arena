# MVP Scope

**Status**: 🟡 In progress
**Last verified**: —

---

## MVP Philosophy

Ship the smallest thing that validates the core hypothesis. For Arena, the hypothesis is: **automated tactical combat with deep build optimization is fun to watch and tinker with.**

---

## MVP 0: Core Combat Engine

### Goal

Playable 3v3 fights that validate the combat math and feel. Prove the core loop works before adding build complexity.

### In Scope

- 3v3 fights on a 5-zone map (North, South, East, West, Center)
- Basic character generation (Attributes + starting Traits)
- Simple hiring market (gold-based, Arena Recruitment Center only)
- Combat actions: Attack, Move, Defend + basic Action Perks from starting Traits
- Speed-based initiative system with diminishing returns (sqrt scaling)
- Roll-vs-static-defense combat resolution (two-step: attack vs defense, damage vs soak)
- Rider effects from Perks (status application after hits)
- Stack-based status effect system with per-status decay rules
- 10 damage types (3 physical, 3 elemental, 4 magical) with thematic status affiliations
- Basic AI: attack nearest, move toward nearest, defend when low HP
- Victory rewards: Gold + XP
- Minimum Trait set for damage type coverage:
  - Physical: Swordmaster (slashing), Lancer (piercing), Brawler (blunt)
  - Elemental: Pyromancer (fire), Cryomancer (cold), Storm Caller (lightning)
  - Magical: Shadowmancer (shadow), Priest (light), Mentalist (psychic), Poisoner (poison)
- Combat spectating: turn-by-turn log output in TUI

### Explicitly Out of Scope

- Equipment system (pure baseline combat testing)
- Consumables
- XP spending / Perk purchasing (characters start with Traits + auto-granted Perks)
- AI configuration (basic AI only)
- Stealth / detection
- Tournaments / PvE encounter system
- Economy beyond simple gold rewards and hiring
- Injury / death system
- Morale
- Multiplayer

### Success Criteria

- A 3v3 fight runs to completion without errors
- Different Trait builds produce meaningfully different combat outcomes
- Combat log is readable and conveys tactical flow
- Fights resolve in a reasonable number of ticks (target: TBD during prototyping)
- Players can make a hiring decision and see it affect their next fight

---

## MVP 1: Build Depth

### Goal

Full build optimization loop — characters feel meaningfully different based on player choices. Prove that the Trait/Perk/Equipment triad creates interesting build decisions.

### Depends On

- MVP 0 complete

### In Scope

- **XP spending**: Train Attributes toward Potential; purchase Perks from Trainers; acquire new Traits
- **Equipment system**: Quality tiers, weapon/armor types, tag system, prefix/suffix affixes, equipment requirements, multi-slot items, anatomical slot constraints, 5-item limit
- **Durability / degradation**: Current quality drops with use; repair reduces max by 1
- **Consumables**: Potions (fast, guaranteed, capped), Bombs (fast, offensive), Scrolls (slow, failure chance, cheap, no ceiling). 5 consumable slots.
- **Loot drops** from victories
- **Basic equipment market** (buy/sell)
- **AI configuration (basic)**: Target priority and action priority lists — enough for players to influence fights
- **Stealth / detection**: Sneaking status, contested Awareness checks
- **Scale**: 1v1 to 20v20 match sizes

### Explicitly Out of Scope

- Support Staff / household bonuses
- Forging / enhancement services
- Injury / death / resurrection
- Metacurrency / Backgrounds
- Advanced vendor variety (Guild vendors, Tavern, etc.)
- Time model (ticks, passive generation)
- Tournaments / PvE encounters
- Morale system
- Full AI customization (range preferences, consumable triggers, personality)
- Multiplayer

### Success Criteria

- Two characters with the same Attributes but different Trait/Perk/Equipment builds play measurably differently
- Equipment affixes interact with Perks (e.g., "+Fire damage" affix enhances Pyromancer Perks)
- Players feel meaningful tension in build choices (trade-offs, not obvious best answers)
- Consumables provide tactical options without dominating fights
- Stealth creates counterplay dynamics (Awareness investment matters)

---

## Feature Parking Lot

Features explicitly deferred beyond MVP 1:

| Feature | Why Deferred | Earliest Phase |
|---------|--------------|----------------|
| Support Staff system | Requires economic depth to matter | Phase 3 |
| Forging / Enhancement | Requires equipment system to stabilize first | Phase 3 |
| Injury / Death / Resurrection | Economic system needed for cost tension | Phase 3 |
| Metacurrency / Backgrounds | Needs progression system to feel rewarding | Phase 3 |
| Vendor variety (Guild, Tavern, etc.) | Economy must exist before diversifying | Phase 3 |
| Tick-based time model | Passive systems need economy context | Phase 3 |
| Tournament structure | Needs economy for meaningful rewards | Phase 3 |
| PvE encounter system | Needs economy + rewards tuning | Phase 3 |
| Full AI customization | Requires stable combat to configure against | Phase 4 |
| Personality system (AI modifiers) | Requires full AI system | Phase 4 |
| Morale system | Requires stable combat + UI for feedback | Phase 4 |
| Advanced battlefield gimmicks | Requires stable combat on basic maps | Phase 4 |
| Multiplayer (all aspects) | All single-player systems must stabilize | Phase 5 |
| PvP matchmaking | Requires multiplayer infrastructure | Phase 5 |
| Cross-player economy | Requires multiplayer + stable economy | Phase 5 |
| Seasons / resets | Requires multiplayer + endgame content | Phase 5 |
| Base building | Future system, design TBD | Phase 3+ |

---

_Last updated: 2026-02-11_
