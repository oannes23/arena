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
| Data Model | [data-model.md](architecture/data-model.md) | 🔴 | Entity relationships, storage approach — deferred until domains stabilize. **From traits-and-perks**: Trait/Perk data shape definition (Trait → Perk tree, resource family membership, tag lists, loot table weights) |
| MVP Scope | [mvp-scope.md](architecture/mvp-scope.md) | 🟡 | MVP 0 / MVP 1 boundaries, parking lot |

---

## Domain Specs

<!-- Order by conceptual dependency: primitives first, composites later -->

| # | Area | Doc | Status | Phase | Key Open Questions |
|---|------|-----|--------|-------|--------------------|
| 1 | Characters | [characters.md](domains/characters.md) | 🟢 | 1 | None — tuning values deferred to combat, tournaments, economy specs |
| 2 | Traits & Perks | [traits-and-perks.md](domains/traits-and-perks.md) | 🟢 | 1 | None — MVP content examples deferred to implementation, combat-triggered Trait discovery deferred to later phase |
| 3 | Combat | [combat.md](domains/combat.md) | 🔄 | 1 | 26 open questions — resolution details, stealth, morale, initiative formula. **Needs revision**: attributes expanded 4→9, multi-attribute blending, Luck crit/resistance mechanic. **From characters**: Stamina exhaustion (0→Health drain), Stamina regen + Defend boost, 15 derived stat formulas with locked weight ratios, Magic Defense stat, per-stat scaling multipliers (Health ×10 baseline), Fallen sub-state mechanics (HP < 1 → out of fight; mid-combat revival?), injury/death roll mechanics for non-exhibition Fallen characters, ephemeral combatant lifecycle (creation/teardown), post-battle recruitment check (Charisma/Luck/Awareness → Named NPC generation). **From traits-and-perks**: Resource Family pool formulas (Mana, Faith, Spirit, Focus), resource pool regen (full start + slow per-tick regen, formula owned by combat), Perk Discovery trigger specifics (Actions + Triggers, not passive Stat Adjustments, rarity-weighted roll), Trait level amplification multipliers (tuning values) |
| 4 | Combat AI | [combat-ai.md](domains/combat-ai.md) | 🟡 | 2 | Trigger granularity, personality override model, NPC AI |
| 5 | Equipment | [equipment.md](domains/equipment.md) | 🟡 | 2 | Quality scaling (linear vs diminishing), degradation rate, loot curves |
| 6 | Consumables | [consumables.md](domains/consumables.md) | 🟡 | 2 | Quality levels, crafting recipe acquisition, scroll failure formula |
| 7 | Economy | [economy.md](domains/economy.md) | 🟡 | 3 | Primary gold sink, XP scaling, tick frequency, PvE availability. **From characters**: Promotion costs metacurrency only (very expensive), training cost = Current value per +1. Pricing/transaction mechanics for Group services. XP earn rates per fight (per-character). Hiring cost should factor starting Trait count. **From traits-and-perks**: Trainer service fees (gold) for Trait/Perk leveling, respec gold cost formula, XP cost schedule (100/300/600/1000/1500) interactions, Group-specific trainer fee model (specialist vs. generalist pricing differences) |
| 8 | Groups | [groups.md](domains/groups.md) | 🟡 | 3 | Group count for MVP, reputation/standing system, Group relationships, service scaling by Bond star level, Group discovery/unlock. **From characters**: NPC membership effects (vendor inventories, loot tables, trainer availability), Free Agent pool (persistent Named NPCs recruitable by players), each recruiting Group must have a Bond Trait (guaranteed at generation). **From traits-and-perks**: Trait generation loot table definitions per Group/archetype (nestable weighted subtables, openness parameter), trainer availability model (Group-specific theme alignment, generalist Groups for baseline access), Bond Trait hybrid scaling (smooth benefits + discrete tier unlocks at 3★/5★) |
| 9 | Roster Management | [roster-management.md](domains/roster-management.md) | 🟡 | 3 | Roster size limits, metacurrency rates, endgame, base building scope. **From characters**: activity restrictions during Recovering state, recovery tick durations, character dismissal/firing mechanics, Free Agent recruitment, post-battle recruitment flow |
| 10 | Tournaments | [tournaments.md](domains/tournaments.md) | 🟡 | 3 | Elimination format, injury timing, multi-tournament rules, forfeit. **Needs**: crowd/momentum section (Charisma-driven). **From characters**: configurable Health/Stamina reset per event type, define which event types are "lethal" (can cause Dead state) |
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

### 2026-02-14: Traits & Perks Spec Interrogation (Round 2)

- Resolved all 18 remaining structural/mechanical questions. Status updated from 🟡 to 🟢 Complete.
- Added acquisition-only requirement gates: Traits, Perks, and prerequisites are never deactivated once owned.
- Added Perk level cap rule: Perk level ≤ parent Trait level (exception: Perk Discovery overrides cap).
- Added Perk scaling: output values scale with level, resource costs/cooldowns stay flat.
- Added Perk prerequisites: flexible graph (not linear chain), cross-Trait prerequisites allowed (rare).
- Clarified no individual Perk removal — only full Trait respec.
- Updated Pool Mechanics: pools start full at combat, slow per-tick regen (formula owned by combat), pool growth is emergent from Perk content, pool disappears on last-Trait respec.
- Updated Perk Discovery: triggers on Actions + Triggers (not passive Stat Adjustments), rarity-weighted roll, discovered Perks override level cap.
- Added Trainer Availability: Group-specific theme alignment, generalist Groups for baseline access.
- Added Bond Trait hybrid scaling: smooth scaling (price, breadth) + discrete tier unlocks (3★, 5★).
- Added Trait Slot Constraints: empty slots only, no overflow/queue/replacement shortcut.
- Added cross-Trait interaction rule: tags only, no Perk can name/reference another specific Trait or Perk.
- Updated glossary: Perk (level cap), Perk Discovery (triggers, rarity, cap override), Respec (Trait-level only).
- Flagged downstream specs:
  - `combat` — resource pool regen (full start + slow tick), Perk Discovery trigger specifics
  - `groups` — trainer availability model, Bond Trait hybrid scaling
  - `economy` — Group-specific trainer fee model (specialist vs. generalist)

### 2026-02-14: Traits & Perks Spec Interrogation (Round 1)

- Resolved all 8 original open questions. Status updated from 🔄 to 🟡 (remaining: MVP content examples, combat Trait discovery deferred).
- Added 5 Resource Families: Arcane (Mana), Divine (Faith), Primal (Spirit), Psychic (Focus), Martial (Stamina). Locked pool formulas with attribute blend weights.
- Defined Species/Ancestry Core Traits: stackable ancestry Traits, per-slot max conflict resolution, default humanoid anatomy.
- Added Tag System: tags on Perks/Actions, Traits derive tags from union of Perks, no explicit set bonuses — all synergies emergent.
- Added Trait Generation Loot Table structure: nestable weighted subtables, per-star-level roll overrides, openness parameter for veteran vs. open-potential archetypes.
- Added Perk Discovery: ~0.1% chance per Perk use to discover a random unowned Perk from the same Trait tree.
- Locked XP cost schedule: 100/300/600/1000/1500 for both Traits and Perks. Leveling costs XP + trainer service fee (gold).
- Defined Starter Perk: every Trait has exactly one auto-granted Perk. Perk stacking: no stack (redundant access only). Perks per Trait: 3–5.
- Respec expanded: all XP invested is lost, gold cost deferred to economy spec.
- Trait level amplification: accelerating multiplier curve (×1.0/×1.2/×1.4/×1.7/×2.0), exact values deferred to combat.
- Updated glossary: added Resource Family, Mana, Faith, Spirit, Focus, Starter Perk, Perk Discovery, Openness; updated Tag, Respec, Resource.
- Flagged downstream specs:
  - `combat` — resource pool formulas, Perk discovery XP trigger, Trait level amplification multipliers
  - `economy` — trainer service fees, respec gold cost, XP cost schedule interactions
  - `groups` — loot table definitions per Group/archetype
  - `data-model` — Trait/Perk data shape definition

### 2026-02-14: Characters Spec Final Gap Sweep (Round 6)

- Added Inventory Slots to character chassis: 5 Equipment Slots + 5 Consumable Slots, fixed (do not scale).
- Added Trait Generation Loot Table: rolls per category = Star Rating. First roll per category guaranteed (reroll on "nothing"). Duplicate rolls unlock random Perk from that Trait's tree. Recruiting Group's Bond Trait always guaranteed.
- Clarified XP is per-character (earned and stored individually). Gold is household-level (shared pool).
- Updated implications for traits-and-perks (loot table weights per archetype), economy (XP earn rates, hiring cost factors starting Traits), groups (each recruiting Group must have a Bond Trait).

### 2026-02-14: Characters Spec Gap Sweep (Round 5)

- Added Stat Visibility decision: full transparency for all Current and Potential values at all times. No hidden stat information — evaluation skill is synergy recognition, not stat discovery.
- Added Character Model Applicability decision: two-tier entity model. Persistent characters (player-owned and Named NPCs) are full Character entities; ephemeral combatants (unnamed enemies) use the same stat model but aren't persistent.
- Added trait overflow on Star Rating decrease: player chooses which Trait(s) to remove from over-capacity categories. Removed Traits are lost.
- Dismissal/firing mechanics deferred to roster-management spec (not part of the character state machine).
- Added glossary terms: Ephemeral Combatant, Named NPC, Free Agent.
- Updated implications for combat (ephemeral combatant lifecycle, post-battle recruitment check), groups (NPC membership effects, Free Agent pool), roster-management (dismissal, Free Agent recruitment, post-battle recruitment flow).

### 2026-02-14: Characters Spec Deep Interrogation (Round 4)

- Added derived stat scaling multipliers: weight ratios are canonical blend weights, per-stat scaling multipliers (e.g., Health ×10) convert to game values. Multipliers are tuning values owned by combat spec.
- Added Fallen in-combat sub-state: HP < 1 → out of fight for remainder. Post-combat fate depends on event type (exhibition = safe recovery, non-exhibition = injury/death roll).
- Defined post-combat outcome rules by event type: exhibition has no injury risk; non-exhibition triggers injury/death roll after combat resolves.
- Star Rating can decrease in extreme events (very rare, design note — triggers deferred to downstream specs).
- Added glossary term: Fallen.
- Deferred to combat spec: per-stat scaling multipliers, Fallen mid-combat revival, injury/death roll mechanics.
- Updated implications for combat (scaling multipliers, Fallen mechanics, injury/death rolls) and tournaments (exhibition safety).

### 2026-02-14: Characters Spec Deep Interrogation (Round 3)

- Clarified Bonus Modifier → Derived Stat flow: Effective Attribute = base Current + all Bonus Modifiers, feeds derived stat formulas. Potential only gates training, not effective values. Hard ceiling is scale max (200).
- Added explicit Character State Machine with 5 states: Available, In-Combat, Recovering, Dead, Retired. Defined all valid state transitions.
- Recovering characters CAN fight (player's risk with penalties). Dead characters require Raise Dead service. Retired is terminal (hall of fame + metacurrency).
- Defined Phase 1 identity fields: name (first + optional epithet/surname), physical descriptors, age, gender/presentation, one-line origin blurb.
- Deferred Training Speed formula to economy spec, recovery tick durations and Recovering activity restrictions to roster-management spec.
- Added glossary terms: Effective Attribute, Recovering, Retired.
- Updated implications for roster-management (activity restrictions, recovery ticks), tournaments (lethal event types), economy (Raise Dead pricing, Training Speed).

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
_2026-02-14 — Traits & Perks spec interrogation round 2 (requirement gates, perk caps/scaling/prerequisites, pool mechanics, discovery triggers, trainer availability, bond hybrid scaling, trait slot constraints, tags-only interaction)._
