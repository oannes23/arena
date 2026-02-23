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
| Data Model | [data-model.md](architecture/data-model.md) | 🔴 | Entity relationships, storage approach — deferred until domains stabilize. **From traits-and-perks**: Trait/Perk data shape definition (Trait → Perk tree, tag lists, loot table weights). Resource type definitions are content data (name, pool formula, auto-tag) — not hardcoded schema. Perk components need structured representation: Actions (target tags, effect component list, speed, cooldown, resource costs, `primary_damage_type`), Stat Adjustments (two subtypes: flat `{stat_key, value}` and tag-scoped `{tag, stat_key, value, value_type: "flat"|"percentage"}`), Triggers (event type, effect component list). Effect components use generic typed format (type tag + key-value parameters). Prerequisite graph must be acyclic (content validation). **From combat**: Hidden system Trait type (Combatant), per-family Soak values (3 families: Physical/Elemental/Magical), Physical + Magic Defense as derived stats, universal Penetration, per-component damage resolution, tiered injury roll data (two-tier check with overkill factor), overkill tracking per Fallen character, target tag vocabulary, Trigger event vocabulary (25+ event types including OnDyingBlow), effect component type catalog (11+ types with parameters), Combat Scoreboard per-character stat accumulation (including Dying Blows), Shield instances (HP pool, duration, type filter, FIFO), Context Flags schema, event template schema (starting zones per team, Initiative offsets), seeded PRNG per combat, structured event stream/log storage, per-team elimination order tracking. **From combat rounds 10-13**: Formula modifier tag representation (attribute substitution: `{source_attr, target_attr, formula_scope}`), per-weapon-type Attack/Damage formula definitions (content data, not schema), summon templates (stat blocks, AI configs, duration), `team_sizes: [int]` list in Context Flags schema (replaces `team_size`/`opponent_team_size`). Stat Adjustment now three subtypes (flat, tag-scoped, formula modifier). **From equipment**: Weapon type definitions (base damage, formula weights, tags, range, Action Speed), armor type definitions (base Soak, scaling weights, slot coverage, category), affix schema (type, tier pool, source-themed weight tables, positive/negative), item instances (type, quality, max durability, affix list), multi-slot anatomical mapping, block chance/value on shield items, equipment-granted Action data (same as Perk Action model), item-local Formula Modifier Tag affixes, loot table schema (per-event, per-enemy, with affix source overrides), default humanoid anatomy (11 slots). |
| Meta-Balance | [meta-balance.md](architecture/meta-balance.md) | 🔴 | **NEW** — Automatic underdog balancing system: win/loss ratio tracking per Trait, automatic bonuses for underperforming Traits/builds. Cross-cutting system affecting combat, tournaments, and economy. **From traits-and-perks**: build diversity philosophy relies on this system as a fourth pillar alongside economic scarcity, content variety, and roster pressure. |
| MVP Scope | [mvp-scope.md](architecture/mvp-scope.md) | 🟡 | MVP 0 / MVP 1 boundaries, parking lot |

---

## Domain Specs

<!-- Order by conceptual dependency: primitives first, composites later -->

| # | Area | Doc | Status | Phase | Key Open Questions |
|---|------|-----|--------|-------|--------------------|
| 1 | Characters | [characters.md](domains/characters.md) | 🟢 | 1 | None — tuning values deferred to combat, tournaments, economy specs |
| 2 | Traits & Perks | [traits-and-perks](domains/traits-and-perks/index.md) | 🟢 | 1 | Updated 2026-02-18: Perk Discovery model changed to per-Trait end-of-combat roll (active Traits only). Post-combat references updated to post-combat.md. MVP content examples deferred to implementation. All structural/mechanical questions resolved. |
| 3 | Combat | [combat](domains/combat/index.md) | 🟢 | 1 | Updated 2026-02-22: rounds 26-31 — 23 additional decisions (implementation readiness pass). Hit bonus primary-only, deferred OnKill queue, Step 5 kill triggers, Effects Phase ordering, default Action Speeds, Revive Step 0, detection per-turn, trigger vocabulary expansion (7 new, 1 removed). 31 rounds total, 143 decisions. |
| 3b | Post-Combat | [post-combat.md](domains/post-combat.md) | 🟢 | 1 | Updated 2026-02-18: 8 interrogation rounds, 18 decisions. Injury mechanics fully specified (diminishing returns formula, Injury Resistance derived stat, tiered severity, named injury catalog, game-tick recovery, death immunity Perks). XP: equal share + participation bonus. Recruitment: NPC generation from event archetype pool, declined → Free Agent. Per-event post-combat configuration. Tournament: full post-combat per round, immediate effects. |
| 4 | Combat AI | [combat-ai.md](domains/combat-ai.md) | 🟢 | 2 | Updated 2026-02-18: Context Flags reference combat.md for canonical schema. resource_efficiency uses fight phase signal. Previous: Round 2 (14 decisions). Standard library: 4 Gates + 14 Tactical Scorers + 5 Personality Scorers. Remaining items are tuning values. |
| 5 | Equipment | [equipment](domains/equipment/index.md) | 🟢 | 2 | Updated 2026-02-21: 31 interrogation rounds, 103 decisions. **Round 31 (cleanup)**: Deadly/Sunder per-implement scope, tag-derived Action Speed, enchanting source-themed, DW penalties (−15%/−35%), armor Speed penalties (Med −2/Hvy −5), armor 3-attribute scaling locked, degradation ~2/fight, percentage-based penalties (~20%). All design decisions resolved. Remaining: implementation-phase tuning values and deferred economic numbers. |
| 6 | Consumables | [consumables.md](domains/consumables.md) | 🟡 | 2 | Quality levels, crafting recipe acquisition, scroll failure formula |
| 7 | Economy | [economy.md](domains/economy.md) | 🟡 | 3 | Primary gold sink, XP scaling, tick frequency, PvE availability. **From characters**: Promotion costs metacurrency only (very expensive), training cost = Current value per +1. Pricing/transaction mechanics for Group services. XP earn rates per fight (per-character). Hiring cost should factor starting Trait count. **From traits-and-perks**: Trainer service fees (gold) for Trait/Perk leveling, respec gold cost formula, XP cost schedule (100/300/600/1000/1500) interactions, Group-specific trainer fee model (specialist vs. generalist pricing differences), XP cost as natural gate for high-star Traits on low-star characters. **From equipment**: Three-tier repair costs (cheap/standard/premium), quality forging exponential cost, affix rerolling costs (gold + materials), enemy drop quality penalty, loot table drop rates per event type. |
| 8 | Groups | [groups.md](domains/groups.md) | 🟡 | 3 | Group count for MVP, reputation/standing system, Group relationships, service scaling by Bond star level, Group discovery/unlock. **From characters**: NPC membership effects (vendor inventories, loot tables, trainer availability), Free Agent pool (persistent Named NPCs recruitable by players), each recruiting Group must have a Bond Trait (guaranteed at generation). **From traits-and-perks**: Trait generation loot table definitions per Group/archetype (nestable weighted subtables, openness parameter), trainer availability model (Group-specific theme alignment, generalist Groups for baseline access), Bond Trait hybrid scaling (smooth benefits + discrete tier unlocks at 3★/5★), one Bond Trait per Group per character, each Group defines exactly one Bond Trait. Note: Bond level no longer has a special Perk Discovery bonus — discovery is driven by Trait Level Multiplier (universal across all categories). **From equipment**: Quality forging requires specific Group service (e.g., Blacksmith's Guild). Different Groups offer different affix reroll services (single-affix vs. full-affix). Source-themed affix tables per Group vendor. Different repair tier access per Group. |
| 9 | Roster Management | [roster-management.md](domains/roster-management.md) | 🟡 | 3 | Roster size limits, metacurrency rates, endgame, base building scope. **From characters**: activity restrictions during Recovering state, recovery tick durations, character dismissal/firing mechanics, Free Agent recruitment, post-battle recruitment flow |
| 10 | Tournaments | [tournaments.md](domains/tournaments.md) | 🟡 | 3 | Elimination format, injury timing, multi-tournament rules, forfeit. **Needs**: crowd/momentum section (Charisma-driven). **From characters**: configurable Health/Stamina reset per event type, define which event types are "lethal" (can cause Dead state). **From traits-and-perks**: Star-gated tournament entry to manage power gaps. Per-combat cooldown reset means no cross-fight cooldown attrition. **From combat**: Attrition ramp onset/rate per event type. Per-combat resource/cooldown resets between rounds. **From post-combat**: Injury/death, Perk discovery, recruitment, loot handled per round by [post-combat](domains/post-combat.md). **From equipment**: Equipment locked at tournament registration — no swaps between rounds. **⚠️ Fight length revised to 25–150 ticks** — tournament pacing assumptions need updating. |
| 11 | Quests | — | 🔴 | 4+ | Future system. Tasks offered by Groups for rewards. Not yet scoped. |

---

## Implementation Specs

### MVP 0

| Epic | Doc | Status | Blocked By |
|------|-----|--------|------------|
| Overview | [overview.md](implementation/mvp-0/overview.md) | 🔴 | Characters 🟢, Traits & Perks 🟢, Combat 🟢 — all domain specs on critical path are complete. Ready for implementation planning. |

### MVP 1

| Epic | Doc | Status | Blocked By |
|------|-----|--------|------------|
| Overview | [overview.md](implementation/mvp-1/overview.md) | 🔴 | MVP 0 complete + Equipment 🟢, Consumables 🟡, Combat AI 🟢 — Equipment complete, Consumables needs interrogation. |

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
    │       │       ├──→ post-combat (depends on: combat, characters, traits-and-perks)
    │       │       │       │
    │       │       │       └──→ tournaments (depends on: combat, post-combat, economy, groups)
    │       │       │
    │       │       └──→ tournaments (depends on: combat, post-combat, economy, groups)
    │       │
    │       ├──→ equipment (depends on: characters, traits-and-perks, groups)
    │       │
    │       ├──→ consumables (depends on: traits-and-perks, groups)
    │       │
    │       ├──→ groups (depends on: characters, traits-and-perks, economy)
    │       │       │
    │       │       └──→ quests (depends on: groups) [future]
    │       │
    │       └──→ roster-management (depends on: characters, economy, traits-and-perks, groups, post-combat)
    │
    ├──→ economy (depends on: characters)
    │       │
    │       ├──→ groups (depends on: characters, traits-and-perks, economy)
    │       │
    │       ├──→ roster-management (depends on: characters, economy, traits-and-perks, groups, post-combat)
    │       │
    │       └──→ tournaments (depends on: combat, post-combat, economy, groups)
    │
    └──→ meta-balance (cross-cutting — depends on: combat, traits-and-perks, tournaments)
```

**MVP 0 critical path**: characters → traits-and-perks → combat → post-combat

**Recommended interrogation order**: characters → traits-and-perks → combat → combat-ai → equipment → consumables → economy → groups → roster-management → tournaments

---

## Recent Changes

See [CHANGELOG.md](CHANGELOG.md) for detailed change history.

---

## Glossary

See [glossary.md](glossary.md) for canonical definitions of all terms.

---

## Last Updated
_2026-02-22 — Combat rounds 26-31: 143 decisions total (23 new). Implementation readiness pass — hit bonus primary-only, deferred OnKill queue, Step 5 kill triggers, Effects Phase ordering, default Action Speeds, Revive Step 0, detection per-turn, trigger vocabulary expansion (7 new, 1 removed). Seven specs 🟢 Complete._
