# Combat — Domain Specification

**Status**: 🟡 In progress
**Last interrogated**: —
**Last verified**: —
**Depends on**: [characters](characters.md), [traits-and-perks](traits-and-perks.md)
**Depended on by**: [combat-ai](combat-ai.md), [tournaments](tournaments.md)

---

## Overview

Combat is the core gameplay loop — automated tactical fights between teams of characters on zone-based maps. Players configure builds and AI before combat; combat executes without player input. This domain covers the spatial system, initiative, action economy, combat resolution, damage types, status effects, stealth, targeting, and resources. Combat AI configuration is a separate domain.

---

## Core Concepts

### Battle Format

- **Scale**: 1v1 to 20v20.
- **Automation**: Pre-configured AI orders; combat executes automatically (idle/auto-battler).
- **Player interaction**: Configure builds and AI before combat; spectate during.

### Spatial System — Zones and Ranges

**Ranges**:
- **Short**: Same zone (melee range)
- **Medium**: Adjacent zones
- **Long**: Non-adjacent zones

**Zone Maps**:
- **Default**: 5 zones (North, South, East, West, Center)
- **Variants**: Custom layouts scaled to combatant count (Phase 4)
- **Gimmicks**: Traps, timed hazards, movement modifiers, obstacles (Phase 4)
- **Zone Capacity**: No strict limits by default; can be specified per-zone per-battlefield

**Movement**:
- Basic Move action costs the character's full turn
- Forced movement effects: push to random adjacent zone, push away, pull toward, etc.

**Design Intent**: Making Move cost a full turn creates meaningful positioning. Ranged characters have a genuine advantage — melee fighters pay a real cost to close distance. Perks that grant "attack + move" or "free movement on kill" become high-value abilities.

### Initiative System

**Speed-based meter with diminishing returns**:
- Each tick, characters add `sqrt(Speed)` to their Initiative meter
- When Initiative ≥ 100, character takes a turn
- After action, Initiative reduced (base −100, modified by Action Speed)
- Speed derived from: Agility + Equipment + Perks + Statuses

**Diminishing returns**: Square root scaling means doubling Speed from 100→200 only increases Initiative gain from 10.0→~14.1 per tick (41% increase for 100% more Speed).

**Tie-Breaking Order**:
1. Highest total Initiative (current + this tick's addition)
2. Highest Initiative added this tick
3. Highest Agility attribute
4. Highest Awareness attribute
5. Highest sum of all Attributes
6. Random coin flip

### Action Economy

A standard turn: **one Action**. Options:
- **Attack**: Standard damage using equipped weapon
- **Move**: Change zones (costs full turn)
- **Defend**: Increase defensive stats until next turn
- **Active Action**: Use a Perk-granted ability (variable Action Speed, cooldowns, resource costs)

Perks may grant bonus actions, free movement, or combined movement+attack. These are exceptions, not baseline.

### Combat Resolution — Roll-vs-Static Defense

**Step 1: Attack Roll vs. Defense**
- Attacker rolls 1 to [Attack Value]
- Defender's Defense = flat subtraction
- Result < 0: Miss
- Result ≥ 0: Hit — excess becomes damage potential bonus

*Example*: 120 Attack vs. 50 Defense → roll 1–120, subtract 50. Results: −49 (miss) to +70 (strong hit). Hit chance ≈ 58%.

**Step 2: Damage Roll vs. Soak**
- Base weapon damage + excess from Step 1, rolled against Soak
- Same model: roll 1 to [Damage Value], subtract Soak
- Result < 0: Fully absorbed
- Result ≥ 0: Damage dealt to Health

**Step 3: Rider Effects**
- Applied after successful hit, each with own resolution
- Examples: Stunning Blow (roll vs. Stun Resistance), Flaming Sword proc (roll vs. Fire Resistance), Armor Penetration (reduces Soak before roll)
- Rider effects defined per-Perk and per-equipment-affix; system supports arbitrary rider hooks

**Balance Implications**:
- Defense stacking is very powerful (flat subtraction) — Defense should scale slower than Attack
- Low Attack is completely ineffective vs. high Defense (intentional hard tier gating)
- All variance on attacker's side; defenders have predictable durability

### Damage Types

**Physical**: Blunt (hammers, maces, unarmed), Piercing (spears, arrows, daggers), Slashing (swords, axes, claws)

**Elemental**: Fire (burning, explosions), Cold (freezing, chill), Lightning (shock, thunder)

**Magical**: Poison (toxins, disease), Shadow (darkness, necrotic), Light (holy, radiant), Psychic (mental, illusion)

**Philosophy**: No inherent rock-paper-scissors. Differences come from affiliated status effects and Perk/equipment interactions. Fire *tends* to inflict Burning, Cold *tends* to inflict Slow — thematic patterns, not hard rules.

### Status Effect System

**Stack-Based Mechanics**:
- Max stacks: 9999 (effectively unlimited)
- Decay: Variable per status type
- Application: From rider effects, triggers, environmental hazards, Perks

**Status Examples** (numbers are tuning targets, not final):

| Status | Effect | Decay Model |
|--------|--------|-------------|
| Stun | Reduces or prevents Initiative gain | Decays by Willpower per tick |
| Burning | Fire DoT per tick | Fixed per-tick reduction |
| Bleeding | Physical DoT | Exotic: reduces by 1 per Health restored |
| Slow | Reduces Speed (and thus Initiative gain) | Fixed duration or per-tick |
| Sneaking | Cannot be targeted (see Stealth) | Broken by attacking or failed check |
| Buff/Debuff | Stat modifications | Various decay rates |

### Stealth & Detection

- While Sneaking, character cannot be targeted by enemies
- Enemies must pass a **contested Awareness check** to detect
- Failed detection: cannot target with single-target or selected-target abilities
- Sneaking is a status within the standard stack-based system

### Targeting

**Target Style Flags** (defined per-Action):
- Single selected
- Single random
- Multiple selected
- Multiple random
- Hybrid (e.g., single high damage + multiple random low damage)
- **Friendly fire flag**: Defined per-Action (some AoEs hit allies, others don't)

### Resources

**Default** (all characters): Health, Stamina
**Trait-defined**: Mana (elemental magic), Faith (divine), others per Trait family

- **Health**: Fully recovers between battles. No natural in-combat regeneration.
- **Stamina**: General-purpose action resource for physical abilities.
- **Resource scale**: Hundreds to thousands for granular tuning.
- **Healing design**: No passive in-combat healing. All healing from Perks, consumables, or equipment procs.

### Morale (Phase 4)

Fighters have a Morale value affected by combat events. Low Morale may cause debuffs, fleeing, or refusal to act. High Morale may grant bonuses. Full design TBD.

---

## Decisions

### Roll-vs-Static Defense Model

- **Decision**: Attacker rolls, defender provides flat threshold. Two-step: Attack vs. Defense, then Damage vs. Soak.
- **Rationale**: All variance on attacker's side makes defenders predictably durable. Flat subtraction means defense investment has clear, legible value.
- **Implications**: Defense stacking is powerful — scaling must be tuned carefully. Low-power characters literally cannot hit high-defense targets (intentional tier gating).

### Zone-Based Spatial System

- **Decision**: Discrete zones with three range bands (Short/Medium/Long), not grid-based or hex-based.
- **Rationale**: Simple enough for auto-battler AI, deep enough for positioning to matter. Zones are easy to represent in TUI.
- **Implications**: All spatial abilities must work in terms of zones and ranges, not absolute positions or distances.

### Movement Costs Full Turn

- **Decision**: Move is a full action — no free movement by default.
- **Rationale**: Creates positioning tension. Ranged characters have a genuine advantage. "Move + attack" Perks become high-value.
- **Implications**: Melee-only characters need gap-closing Perks or forced movement to compete with ranged. AI must factor movement cost into decisions.

### Initiative: Sqrt Speed Scaling

- **Decision**: Initiative gain per tick = `sqrt(Speed)`, acting at threshold ≥ 100.
- **Rationale**: Diminishing returns prevent degenerate speed stacking while keeping speed valuable.
- **Implications**: Speed investment has clear but limited returns. May need to switch to logarithmic or bracket system if sqrt is hard to tune — the principle (strong diminishing returns) is locked; the formula is not.

### 10 Damage Types, No Inherent Advantages

- **Decision**: 10 damage types across 3 families (Physical, Elemental, Magical) with no inherent rock-paper-scissors.
- **Rationale**: Advantages come from Perk/equipment interactions and status affiliations, not from hardcoded type matchups. Enables emergent meta via content, not systemic rules.
- **Implications**: All 10 types need at least one Trait and representative Perks to be viable. Resistance must be per-type or per-family.

### Unified Stack-Based Status System

- **Decision**: All temporary effects use stacks with per-status decay rules. No separate subsystems for buffs vs. debuffs vs. DoTs vs. CC.
- **Rationale**: Consistent, extensible. One implementation, infinite content variety.
- **Implications**: Every new status effect is just a data definition (per-stack effect + decay model), not new code.

### No Passive In-Combat Healing

- **Decision**: No natural Health regeneration during combat.
- **Rationale**: Makes healing a build investment. Teams must commit resources (Perk slots, consumable slots, equipment affixes) to sustain.
- **Implications**: Matches trend toward attrition without healing investment. Match length tuning is critical.

---

## Open Questions

### Combat Resolution
1. Does the damage roll (Step 2) use the same roll-vs-static model, or a different one? Double flat subtraction may make armor builds nearly immune to chip damage.
2. Initiative recovery after action: always −100, or modified by Action Speed? (Partially defined — Action Speed exists but exact formula TBD.)
3. Status application timing: before or after damage?
4. Resistance stacking: additive or multiplicative?
5. Do critical hits exist? How do they work?
6. Overkill/underkill mechanics: does excess damage matter?
7. Zone-based damage falloff: range penalties for non-optimal range?
8. Defense scaling: how to prevent Defense from becoming too dominant given flat-subtraction?
9. Expected match length in turns/ticks? (Shapes every balance decision: DoTs, cooldowns, resources, healing.)
10. Mirror match handling in multiplayer: how do identical builds resolve?

### Stealth & Detection
11. Does Sneaking break on attack? (Classic stealth: strike then reveal?) If "attack + re-stealth" loops exist, needs heavy Perk gating.
12. How does AoE interact with Sneaking? Can AoE hit a Sneaking character in the same zone?
13. Detection range: zone-based or map-wide?

### Morale
14. Is Morale combat-only or persistent between fights?
15. What triggers affect Morale? (Ally death, heavy damage, kills, rallying Perks)
16. Consequences of low Morale: debuffs, fleeing, refusal to act?
17. Is it stack-based (fits status system) or a separate system?

### Initiative
18. Exact formula: `sqrt(Speed)` per tick, or alternative scaling?
19. Speed floor/ceiling for Initiative calculations?
20. Haste/Slow effects: modify Speed directly, or modify Initiative gain separately?

### Perk Actions
21. Detailed resource costs per action type?
22. Resource regeneration rates (Stamina, Mana, Faith)?
23. Action interrupt mechanics?
24. Channeled abilities (multi-turn actions)?

### Content Volume
25. All 10 damage types in Phase 1, or subset?
26. Zone/battlefield variants in Phase 1?

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [combat-ai](combat-ai.md) | AI must understand zones, ranges, action options, and target selection. AI configuration references Action priority lists. |
| [characters](characters.md) | Attributes feed all combat calculations. Anatomical slots determine weapon/armor options. |
| [traits-and-perks](traits-and-perks.md) | Perks provide Actions, Stat Adjustments, Triggers. Traits unlock resource types. |
| [equipment](equipment.md) | Weapons provide Attack/Damage values and tags. Armor provides Defense/Soak. Affixes add rider effects. |
| [consumables](consumables.md) | Consumables are usable during combat turns (potions, bombs, scrolls). AI must decide when to use them. |
| [tournaments](tournaments.md) | Tournament matches are combat instances. Match format and injury rules feed from combat outcomes. |

---

_Last updated: 2026-02-11_
