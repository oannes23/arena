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
| 1 | Characters | [characters.md](domains/characters.md) | 🟡 | 1 | Star rating generation, Current/Potential distribution, derived stat formulas |
| 2 | Traits & Perks | [traits-and-perks.md](domains/traits-and-perks.md) | 🟡 | 1 | Content volume (min traits/perks), cross-category synergies, stacking rules |
| 3 | Combat | [combat.md](domains/combat.md) | 🟡 | 1 | 26 open questions — resolution details, stealth, morale, initiative formula |
| 4 | Combat AI | [combat-ai.md](domains/combat-ai.md) | 🟡 | 2 | Trigger granularity, personality override model, NPC AI |
| 5 | Equipment | [equipment.md](domains/equipment.md) | 🟡 | 2 | Quality scaling (linear vs diminishing), degradation rate, loot curves |
| 6 | Consumables | [consumables.md](domains/consumables.md) | 🟡 | 2 | Quality levels, crafting recipe acquisition, scroll failure formula |
| 7 | Economy | [economy.md](domains/economy.md) | 🟡 | 3 | Primary gold sink, XP scaling, tick frequency, PvE availability |
| 8 | Roster Management | [roster-management.md](domains/roster-management.md) | 🟡 | 3 | Roster size limits, metacurrency rates, endgame, base building scope |
| 9 | Tournaments | [tournaments.md](domains/tournaments.md) | 🟡 | 3 | Elimination format, injury timing, multi-tournament rules, forfeit |

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
    │       │       └──→ tournaments (depends on: combat, economy)
    │       │
    │       ├──→ equipment (depends on: characters, traits-and-perks)
    │       │
    │       ├──→ consumables (depends on: traits-and-perks)
    │       │
    │       └──→ roster-management (depends on: characters, economy, traits-and-perks)
    │
    └──→ economy (depends on: characters)
            │
            ├──→ roster-management (depends on: characters, economy, traits-and-perks)
            │
            └──→ tournaments (depends on: combat, economy)
```

**MVP 0 critical path**: characters → traits-and-perks → combat

**Recommended interrogation order**: characters → traits-and-perks → combat → combat-ai → equipment → consumables → economy → roster-management → tournaments

---

## Recent Changes

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
_2026-02-11 — Spec structure populated from game design document._
