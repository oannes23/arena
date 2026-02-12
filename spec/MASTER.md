# Arena — Spec Master Index

## How This Document Works

This is the central index for all design specifications. Each area links to its detailed spec doc.

**Status indicators:**
- 🔴 Not started — needs initial interrogation
- 🟡 In progress — has content, needs deepening
- 🟢 Complete — no open questions remain
- 🔄 Needs revision — downstream decisions may have invalidated something

**Workflow:**
1. Run `/interrogate spec/domains/<area>` to deepen any spec
2. Answer questions until the agent has no more to ask
3. Agent updates the spec doc, glossary, and this index
4. Repeat for next area

---

## Core Principle

> **Deep build optimization with economic pressure, played in short sessions. Complexity lives in roster and build management, not moment-to-moment reflexes.**

---

## Scope

### MVP 0 — Core Combat Engine
- Playable 3v3 fights that validate the combat math and feel. No equipment, no economy beyond basic hiring.

### MVP 1 — Build Depth
- Full Trait/Perk/Equipment build optimization loop. Characters feel meaningfully different based on player choices.

See [mvp-scope.md](architecture/mvp-scope.md) for full details.

---

## Architecture Specs

| Area | Doc | Status | Notes |
|------|-----|--------|-------|
| System Overview | [overview.md](architecture/overview.md) | 🟡 | Philosophy, constraints, non-goals, star rating system |
| Data Model | [data-model.md](architecture/data-model.md) | 🔴 | Entity relationships, storage approach — deferred until domains stabilize |
| MVP Scope | [mvp-scope.md](architecture/mvp-scope.md) | 🟡 | MVP 0 / MVP 1 boundaries, parking lot |

---

## Domain Specs

<!-- Order by conceptual dependency: primitives first, composites later -->

| # | Area | Doc | Status | Phase | Key Open Questions |
|---|------|-----|--------|-------|--------------------|
| 1 | Characters | [characters.md](domains/characters.md) | 🟢 | 1 | None — tuning values deferred to combat, tournaments, economy specs |
| 2 | Traits & Perks | [traits-and-perks.md](domains/traits-and-perks.md) | 🔄 | 1 | Content volume (min traits/perks), cross-category synergies, stacking rules. **Needs revision**: attributes expanded from 4 to 9, species-as-Core-Traits pattern |
| 3 | Combat | [combat.md](domains/combat.md) | 🔄 | 1 | 26 open questions — resolution details, stealth, morale, initiative formula. **Needs revision**: attributes expanded 4→9, multi-attribute blending, Luck crit/resistance mechanic. **From characters**: Stamina exhaustion (0→Health drain), Stamina regen + Defend boost, 15 derived stat formulas with locked weight ratios, Magic Defense stat |
| 4 | Combat AI | [combat-ai.md](domains/combat-ai.md) | 🟡 | 2 | Trigger granularity, personality override model, NPC AI |
| 5 | Equipment | [equipment.md](domains/equipment.md) | 🟡 | 2 | Quality scaling (linear vs diminishing), degradation rate, loot curves |
| 6 | Consumables | [consumables.md](domains/consumables.md) | 🟡 | 2 | Quality levels, crafting recipe acquisition, scroll failure formula |
| 7 | Economy | [economy.md](domains/economy.md) | 🟡 | 3 | Primary gold sink, XP scaling, tick frequency, PvE availability. **From characters**: Promotion costs metacurrency only (very expensive), training cost = Current value per +1. Pricing/transaction mechanics for Group services |
| 8 | Groups | [groups.md](domains/groups.md) | 🟡 | 3 | Group count for MVP, reputation/standing system, Group relationships, service scaling by Bond star level, Group discovery/unlock |
| 9 | Roster Management | [roster-management.md](domains/roster-management.md) | 🟡 | 3 | Roster size limits, metacurrency rates, endgame, base building scope |
| 10 | Tournaments | [tournaments.md](domains/tournaments.md) | 🟡 | 3 | Elimination format, injury timing, multi-tournament rules, forfeit. **Needs**: crowd/momentum section (Charisma-driven). **From characters**: configurable Health/Stamina reset per event type |
| 11 | Quests | — | 🔴 | 4+ | Future system. Tasks offered by Groups for rewards. Not yet scoped. |

---

## Implementation Specs

### MVP 0

| Epic | Doc | Status | Blocked By |
|------|-----|--------|------------|
| Overview | [overview.md](implementation/mvp-0/overview.md) | 🔴 | Characters, Traits & Perks, Combat specs need interrogation |

### MVP 1

| Epic | Doc | Status | Blocked By |
|------|-----|--------|------------|
| Overview | [overview.md](implementation/mvp-1/overview.md) | 🔴 | MVP 0 complete + Equipment, Consumables, Combat AI specs |

---

## Dependency Graph

```
characters (primitive — depends on nothing)
    │
    ├──→ traits-and-perks (depends on: characters)
    │       │
    │       ├──→ combat (depends on: characters, traits-and-perks)
    │       │       │
    │       │       ├──→ combat-ai (depends on: combat)
    │       │       │
    │       │       └──→ tournaments (depends on: combat, economy, groups)
    │       │
    │       ├──→ equipment (depends on: characters, traits-and-perks, groups)
    │       │
    │       ├──→ consumables (depends on: traits-and-perks, groups)
    │       │
    │       ├──→ groups (depends on: characters, traits-and-perks, economy)
    │       │       │
    │       │       └──→ quests (depends on: groups) [future]
    │       │
    │       └──→ roster-management (depends on: characters, economy, traits-and-perks, groups)
    │
    └──→ economy (depends on: characters)
            │
            ├──→ groups (depends on: characters, traits-and-perks, economy)
            │
            ├──→ roster-management (depends on: characters, economy, traits-and-perks, groups)
            │
            └──→ tournaments (depends on: combat, economy, groups)
```

**MVP 0 critical path**: characters → traits-and-perks → combat

**Recommended interrogation order**: characters → traits-and-perks → combat → combat-ai → equipment → consumables → economy → groups → roster-management → tournaments

---

## Recent Changes

### 2026-02-11: Groups Domain Created

- Unified guilds, temples, taverns, factions, and all city organizations under a single entity type: "Group"
- Created `spec/domains/groups.md` with service menu model (vendor, training, recruitment, spellcasting, crafting, quests)
- Public vs. member service access via Bond Traits
- Added Quests as future domain placeholder (🔴)
- Updated glossary: Group, Service (Group), Bond Trait (references Groups), Archetype (Group-specific)
- Updated dependency graph to include Groups and Quests

### 2026-02-11: Characters Spec Deep Interrogation (Round 2)

- Resolved all 4 remaining open questions — characters spec now 🟢 Complete
- Locked 15 derived stat formulas with exact weight ratios (no more TBD)
- Replaced universal archetypes with vendor/guild-specific archetype system
- Set attribute range to 0–200 for both Current and Potential
- Defined star rating attribute scaling (1★ ~35/~75 → 5★ ~65/~115)
- Added Bonus Modifier system (tracked layer, additive stacking)
- Defined training cost formula (cost per +1 = current value)
- Specified promotion: metacurrency-only, +10% all Potentials
- Added Stamina exhaustion mechanic (0 Stamina → Health drain)
- Added Potential mutability rules (Promotion up, injuries down, rare events)
- Deferred tuning values to combat, tournaments, and economy specs
- Added glossary terms: Bonus Modifier, Exhaustion, Magic Defense
- Updated cross-spec implications for Combat, Economy, Tournaments, Roster Management, Traits & Perks

### 2026-02-11: Characters Spec Interrogated (Round 1)

- Expanded primary attributes from 4 (Strength, Agility, Willpower, Awareness) to 9 (Might, Speed, Accuracy, Endurance, Charisma, Awareness, Intellect, Willpower, Luck)
- Established multi-attribute blending principle for derived stats (no dump stats)
- Added crowd/momentum mechanics via Charisma → Crowd Appeal
- Added star rating generation model (vendor tiers + gold + metacurrency)
- Added rare promotion mechanic (+1★ via expensive process)
- Added character generation model (invisible archetypes + noise)
- Added career milestones and optional retirement for metacurrency
- Added phased character identity system (cosmetic → personality → narrative)
- Resolved all 6 original open questions; 4 new implementation-level questions remain
- Flagged Traits & Perks and Combat specs for revision (attribute changes)
- Flagged Tournaments for crowd/momentum section

### 2026-02-11: Spec Structure Built from Game Design Doc

- Populated all architecture specs (overview, mvp-scope) from design doc
- Created 9 domain specs with decided content and open questions
- Built glossary with ~45 canonical term definitions
- Established dependency graph and interrogation order
- All domains at 🟡 status — need interrogation to resolve open questions

### 2026-02-10: Project Initialized

- Created initial spec structure
- Key decisions pending: all

---

## Glossary

See [glossary.md](glossary.md) for canonical definitions of all terms.

---

## Last Updated
_2026-02-11 — Groups domain created; Characters spec complete (🟢)._
