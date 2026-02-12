# System Overview

**Status**: 🟡 In progress
**Last verified**: —

---

## What This System Does

Arena is a procedurally generated gladiatorial house management game — a roguelike idle/auto-battler designed for short-session play (particularly CI/CD pipeline waits). Players recruit fighters, optimize builds through deep Trait/Perk systems, manage equipment and economy, and watch automated tactical combat unfold in a classic roguelike TUI interface.

**Tech Stack**: Python + Claude Code, MUD-style architecture (headless server + TUI client), future expansion to web/mobile clients.

---

## Core Architectural Principle

> **Deep build optimization with economic pressure, played in short sessions. Complexity lives in roster and build management, not moment-to-moment reflexes.**

This principle guides all design decisions. The game loop is: check in → review results → make roster/build decisions → queue actions → leave.

---

## System Context

### Users/Actors

- **Player**: Manages a gladiatorial household. Recruits fighters, optimizes builds, configures AI orders, manages economy. Interacts between fights; spectates during combat.
- **NPC Teams**: AI-controlled opponent households that fill tournament brackets and provide PvE encounters. Generated procedurally.

### Game Modes

- **Single-player**: Player controls time advancement (tick on command). Idle when you want, grind when you want.
- **Multiplayer** (Phase 5): Persistent world state, real-time ticks, scheduled tournaments, PvP matchmaking, cross-player economy.

### External Systems

- **TUI Client**: MUD-style terminal interface for player interaction and combat spectating.
- **Headless Server**: Game logic runs server-side; clients connect for display. Enables future web/mobile clients.

---

## Constraints

### Technical Constraints

- **TUI-first**: All systems must be representable in a terminal interface. No graphics dependencies.
- **Headless server**: Game state and logic are server-side; clients are display-only.
- **High number scaling**: Stats use integers in the tens/hundreds, resources in hundreds/thousands. No floating point for balance math.
- **Deterministic combat**: Given the same inputs and random seed, combat must produce the same result (for replay and debugging).

### Design Constraints

- **Idle-friendly**: Sessions should be productive in 2–5 minutes. No time pressure during decision-making.
- **Auto-battler**: Players configure builds and AI before combat; combat executes without player input.
- **Modularity**: Systems connect via tags, requirements, and mechanical interfaces — not hardcoded interactions.
- **Tunable balance**: Category-based weighting with data-driven tracking, not fixed formulas.

### Non-Goals

- **Real-time combat interaction**: Players do not control fighters during combat.
- **Twitch reflexes**: No gameplay requires fast reactions.
- **Pixel art / graphical rendering**: TUI only (at least through Phase 4).
- **Pay-to-win monetization**: Not a design consideration.
- **Story campaign**: No linear narrative. Emergent stories through roster management.
- **Modding API** (for now): Moddability is a future consideration, not a Phase 1–4 requirement.

---

## Key Architectural Decisions

### Star Rating System (0–5★)

Universal quality/power rating applied across all game systems. This is a shared lookup table, not a domain-specific system.

- **Decision**: A single 0–5★ scale with color coding applies to Characters, Traits, Perks, and Equipment.
- **Rationale**: Provides a consistent power language across all systems. Players instantly understand relative quality regardless of context.
- **Implications**: Every system that uses star ratings must map to this table. Balance tuning can use star tier as a coarse power bracket.

| Stars | Color  | Tier              |
|-------|--------|-------------------|
| 0★    | Gray   | Broken / Degraded |
| 1★    | White  | Standard / Common |
| 2★    | Green  | Superior / Uncommon |
| 3★    | Blue   | Rare              |
| 4★    | Purple | Epic              |
| 5★    | Orange | Legendary         |

**Applied to**:
- **Characters**: Star rating determines Trait slot capacity (1–5 slots per category).
- **Traits**: Minimum purchase star level (1–5★); all level to 5★.
- **Perks**: Minimum purchase star level (1–5★); all level to 5★.
- **Equipment**: Quality ranges map to star tiers (0–99 = 0★, 100 = 1★, ..., 500+ = 5★).

### Modularity via Tags and Requirements

- **Decision**: Systems connect through shared tags (weapon tags, armor tags, consumable tags) and a universal requirements system (attribute/trait/perk prerequisites).
- **Rationale**: Enables emergent interactions without hardcoded cross-system logic. Adding a new Perk that interacts with "Reach" weapons requires no changes to the equipment system.
- **Implications**: All content definition must use the tag vocabulary. New tags must be globally unique.

### Stack-Based Status System

- **Decision**: All temporary effects (buffs, debuffs, DoTs, crowd control) use a unified stack-based system with per-status decay rules.
- **Rationale**: One system to learn, one system to implement. Extensible — adding a new status means defining its per-stack effect and decay model.
- **Implications**: No special-case status systems. Morale, if implemented as a status, uses stacks too.

### No Passive In-Combat Healing

- **Decision**: All in-combat healing comes from Perks, consumables, or equipment procs. No natural regeneration.
- **Rationale**: Makes healing a build investment. Creates meaningful team composition decisions (dedicated healer vs. more damage).
- **Implications**: Sustain builds require explicit investment. Match length tuning must account for low-healing scenarios.

---

## Design Principles

1. **Modularity**: Systems hook together via tags, requirements, and mechanical interfaces
2. **Tunable Balance**: Category-based weighting with data-driven tracking
3. **Interdependency**: Perks and equipment designed together (Phase 2 priority)
4. **Build Diversity**: Three Trait categories create identity; tags and requirements create constraints
5. **Economic Pressure**: Upkeep, degradation, injury, and consumable costs create resource tension
6. **Progression Through Turnover**: Expensive respec encourages roster rotation
7. **Vendor Access as Progression**: Bond Traits unlock better services and markets
8. **Dual-Purpose Characters**: Role Traits enable combat AND support capabilities
9. **Quality as Universal Modifier**: 0–5★ applies everywhere
10. **High Number Scaling**: Granular tuning without floating point
11. **Tag-Driven Mechanics**: Shared tag vocabulary across all equipment and consumables
12. **Idle-Friendly**: Complexity between fights, not during
13. **No Natural Healing**: In-combat healing is a build choice

---

_Last updated: 2026-02-11_
