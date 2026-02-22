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
| 2 | Traits & Perks | [traits-and-perks.md](domains/traits-and-perks.md) | 🟢 | 1 | Updated 2026-02-18: Perk Discovery model changed to per-Trait end-of-combat roll (active Traits only). Post-combat references updated to post-combat.md. MVP content examples deferred to implementation. All structural/mechanical questions resolved. |
| 3 | Combat | [combat.md](domains/combat.md) | 🟢 | 1 | Updated 2026-02-18: rounds 14-19 — 18 additional decisions. Revival mechanics (clean slate, healing saves, self-save, summoner revival), summon control capacity budget, mutual elimination draw, non-Stamina resource hard-gating, crit/resistance/stealth formulas resolved, min 1 stack, frozen decay while Fallen, Fallen Resolution in Initiative order, default Attack costs Stamina, formula representation, formula modifier per-category scope, Revive on any ally + `[Fallen]` target tag. 19 rounds total, 100 decisions. |
| 3b | Post-Combat | [post-combat.md](domains/post-combat.md) | 🟢 | 1 | Updated 2026-02-18: 8 interrogation rounds, 18 decisions. Injury mechanics fully specified (diminishing returns formula, Injury Resistance derived stat, tiered severity, named injury catalog, game-tick recovery, death immunity Perks). XP: equal share + participation bonus. Recruitment: NPC generation from event archetype pool, declined → Free Agent. Per-event post-combat configuration. Tournament: full post-combat per round, immediate effects. |
| 4 | Combat AI | [combat-ai.md](domains/combat-ai.md) | 🟢 | 2 | Updated 2026-02-18: Context Flags reference combat.md for canonical schema. resource_efficiency uses fight phase signal. Previous: Round 2 (14 decisions). Standard library: 4 Gates + 14 Tactical Scorers + 5 Personality Scorers. Remaining items are tuning values. |
| 5 | Equipment | [equipment.md](domains/equipment.md) | 🟢 | 2 | Updated 2026-02-21: 31 interrogation rounds, 103 decisions. **Round 31 (cleanup)**: Deadly/Sunder per-implement scope, tag-derived Action Speed, enchanting source-themed, DW penalties (−15%/−35%), armor Speed penalties (Med −2/Hvy −5), armor 3-attribute scaling locked, degradation ~2/fight, percentage-based penalties (~20%). All design decisions resolved. Remaining: implementation-phase tuning values and deferred economic numbers. |
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

### 2026-02-21: Equipment Cleanup Round 31 — 15 Decisions (Complete → 🟢)

- **Completed equipment spec** with 103 decisions across 31 interrogation rounds. Status updated from 🟡 to 🟢 Complete.
- **Deadly/Sunder per-implement**: Offensive implement tags (Deadly, Sunder) apply only to attacks made with that specific implement. Defensive tags (Defend, Deflect) remain passive/character-wide. Clean split: defensive = always on, offensive = per-weapon.
- **Action Speed tag-derived**: Light (+bonus), Heavy (−penalty), Two-Handed (−small). No per-type modifiers. Simpler, fewer knobs.
- **Enchanting source-themed**: Group vendor theme biases available affixes for enchanting, same as loot generation.
- **No Versatile focuses**: Focuses are strictly Light or Two-Handed. Versatile is weapons-only.
- **DW penalties locked**: Main-hand −15%, off-hand −35% (Attack and Damage). Raw DW ≈ 75% output. Perk-gated investment.
- **Armor Speed penalties locked**: Medium −2, Heavy −5 per piece (Light = 0). Full Heavy = −30 Speed. Forces mixed loadouts.
- **Armor 3-attribute scaling locked**: Heavy (55% End + 30% Mig + 15% Spd), Medium (40% End + 30% Spd + 30% Awr), Light (60% Spd + 30% Awr + 10% End).
- **Degradation base rate**: ~2 quality per fight at 1.0× usage. A 100-quality item lasts ~50 fights.
- **Percentage-based penalties**: Reach/Ranged (~20% Attack Value), Multihit additional targets (~15–20% Attack Value). Scales with character power.
- **Accessory degradation**: Proportional to combat length (ticks survived).
- Updated specs:
  - `equipment.md` — 15 new decisions (#89-103). Status → 🟢. Implement tags updated with scope (passive vs per-implement). All penalty ranges locked. Armor scaling locked. Open Questions reduced to implementation-phase tuning only.
  - `glossary.md` — 6 updated (Weapon Type, Focus Type, Tag, Dual-Wield, Durability, Enchanting).
  - Downstream flagged: characters (remove "5 Equipment Slots" reference), combat (percentage penalties, Deadly/Sunder per-implement resolution), data-model (implement tag scope field, armor 3-attribute scaling, tag-derived Action Speed)

### 2026-02-21: Equipment Interrogation — 66 Decisions (Rounds 24-25: Focus Types + Implement Tags)

- **FOCUS TYPE REDESIGN (Round 24)**: 7 focus types with distinct combat roles, replacing previous 5 types. Relic split into Holy Symbol + Artifact. Rod added.
- **Wand**: Long range, single target, lower base damage, Deadly tag (crit bonus). Sniper. Intellect + Accuracy.
- **Tome**: Long range, Multihit tag (AoE). Grenade lobber. Two-Handed. Intellect + Awareness.
- **Staff**: Medium range, high base damage, Defend + Deflect tags (Phys + Magic Defense). Tanky two-hander. Intellect + Willpower.
- **Orb**: Medium range, Multihit tag (AoE). One-hand Light. Intellect-weighted.
- **Holy Symbol** (was Relic): Short range, Deflect + Multihit tags (AoE + Magic Defense). Willpower-weighted.
- **Artifact** (split from Relic): Long range, single target, wild card. Template overrides per item.
- **Rod** (NEW): Short range, single target, Defend tag (Physical Defense). Caster tank. Willpower + Endurance.
- **Focus = magic weapon for damage**: Focus provides default damage basis for magic Actions. Perk Actions override range/tags but use focus base power.
- **One adaptive Attack**: Single Attack Action adapts to main-hand. Multihit tag changes targeting to AoE.
- **IMPLEMENT TAG SYSTEM (Round 25)**: 4 new tags shared across weapons and focuses:
  - **Defend**: +~10% of implement's Attack Value → Physical Defense bonus. On: Staff, Rod. May apply to some weapons.
  - **Deflect**: +~10% of implement's Attack Value → Magic Defense bonus. On: Staff, Holy Symbol.
  - **Deadly**: Crit chance bonus. On: Wand, Axe (weapon). Implement's base damage set lower to compensate.
  - **Multihit**: Default Attack hits N different enemies in zone (AoE). One attack roll. N is type-defined. On: Tome, Orb, Holy Symbol.
- **Artifact template overrides**: Individual Artifact item templates can override base combat role fields (range, tags, targeting, bonuses). More a category than a fixed type.
- **Multihit unified**: Multi-target and multi-hit are the same mechanic — all three (Tome, Orb, Holy Symbol) hit N enemies via AoE.
- Updated specs:
  - `equipment.md` — Focus Types table with tags, Implement Tags section, Default Attack Adaptation updated, Axe gets Deadly, decisions 60-66.
  - `glossary.md` — Focus Type updated, Tag updated (implement tags: Defend/Deflect/Deadly/Multihit).
  - Downstream flagged: combat (Defend/Deflect/Deadly/Multihit tag resolution, Artifact overrides), data-model (implement tags, Artifact template overrides, Multihit N per type)

### 2026-02-19: Equipment Interrogation — 59 Decisions (Rounds 16-23: Major Restructure)

- **MAJOR RESTRUCTURE**: Weapons and Focuses now parallel Hand-slot implement categories. Status reverted 🟢 → 🟡 (Focus ranges/formulas still pending).
- **Weapons = physical implements**: Sword, Dagger, Mace, Axe, Spear, Greatsword, Warhammer, Bow. Define base damage for physical Actions.
- **Focuses = magical implements**: Staff, Wand, Tome, Orb, Relic. Define base magic power for magical Actions. Same mechanical role as weapons. Staff/Wand/Tome moved OUT of weapons table.
- **Off-hand bonus attack**: Every Action triggers a generic Attack from the off-hand implement. Main-hand Action resolves first (full effects), then off-hand fires a basic Attack of its type. Physical Action + magic off-hand = physical strike + magic bonus. Magic Action + weapon off-hand = spell + physical bonus.
- **Implement resolution**: Actions specify physical or magical implement type. Fallthrough: main-hand → off-hand → default (Unarmed/Unfocused).
- **Unfocused magic default**: Magic without a focus = weak default (like Unarmed for physical). Perks can enhance.
- **Mixed weapon+focus builds**: Sword+Orb is hybrid. Any combination valid.
- **Light-only off-hand (both categories)**: Weapons and focuses restricted to Light tag for off-hand. Perk override (Ambidextrous).
- **DW = one Action + bonus attack**: Not two separate Actions. One Initiative cost. Off-hand auto-fires generic Attack.
- **Per-implement Trigger firing**: Each implement's procs fire only on its own hits.
- **Equipment combat lock**: All fights (not just tournaments) snapshot equipment at combat start.
- **Random 1-to-max affix count**: Loot variance. Fewer affixes = stronger each.
- **Enchanting service (NEW)**: Group vendor fills empty affix slots. Two-step upgrade: forge to open slots, enchant to fill them.
- **Cursed items**: Free unequip (no sticky curses). Negative affixes rerollable via standard vendor services.
- **Always visible affixes**: No identification mechanic.
- **Focus tag replaces Arcane Focus**: Generic tag on all focuses.
- **Orb**: One-hand Light focus. Intellect-weighted (arcane/destructive).
- **Relic**: One-hand Light focus. Willpower-weighted (divine/protective). Replaces Holy Symbol.
- **Focus ranges and formulas**: Deferred to next interrogation.

#### Previous rounds (1-15):
- **Completed equipment spec** with 25 decisions across 9 interrogation rounds. Status updated from 🟡 to 🟢 Complete.
- **Quality scaling: Linear confirmed**. Quality multiplies base stats, but base stats feed into attribute-blend formulas. "Squire with Excalibur ≈ Musashi with an oar" — build depth is the dominant power axis, not gear quality.
- **Weapon type defines all**: Base damage, Attack/Damage formula weights, tags, range, speed. Individual items vary by quality + affixes. Rare affixes can override formula weights (item-local Formula Modifier Tags, e.g., Charisma-scaling sword).
- **Degradation: Per-fight with usage weight**. `base_degrade × usage_weight` (weapons: Actions taken, armor: damage absorbed). No quality floor — full decay to 0 = item destroyed. Maximum economic pressure.
- **Three-tier repair**: Cheap (max−2), Standard (max−1), Premium (max, very expensive). Economic choices about gear lifespan.
- **Quality forging**: Group-gated (e.g., Blacksmith's Guild), exponential cost. MVP 1. Creates "maintain vs. replace" tension.
- **Tiered affix pools**: Progressive unlock per quality tier. 1-2★ basic stats, 3★ damage types + procs, 4★ formula modifiers + simple Actions, 5★ rare unique mechanics.
- **Source-themed affix generation**: Each loot source (event type, enemy type, Group vendor) biases toward thematic affixes.
- **Affix rerolling**: Vendor-dependent — different Groups offer single-affix vs. full-affix rerolling.
- **Block Chance from shield items**: Post-hit flat reduction. Shield items have block chance % + block value (quality-scaled). Separate from Perk/spell Shield HP-pool mechanic.
- **Accessories in MVP 1**: Rings (2x Finger), Amulet (Neck), Cloak (Back). Pure affix carriers.
- **Default humanoid anatomy: 11 slots**: Head, Torso, Arms, Legs, Hands, Feet, 2x Hand, 2x Finger, Neck, Back.
- **Hand slots**: 2x flexible. Hold Weapons (physical) OR Focuses (magical) OR Shields. Any combination. Dual-wield = one Action + off-hand generic bonus Attack with penalties (Perks mitigate). Versatile = auto-mode by off-hand occupancy, two stat profiles.
- **Equipment can grant Actions**: Same data model as Perk Actions. Available at 4-5★ affix pools.
- **Armor: Hybrid flat + scaling**: Flat base Soak (quality-scaled) + attribute-scaling per armor type (Light: Speed+Awareness, Medium: Endurance+Speed, Heavy: Endurance+Might).
- **Multi-slot items: Less than proportional** (~1.0/1.7/2.2/2.6/3.0× for 1-5 slots).
- **Damage-type affixes add effect components**: +Fire affix adds Fire Damage component to Attack action, resolves through Elemental Soak.
- **Cursed items**: Negative affixes alongside positive ones. Always identified.
- **Loot generation: Hybrid** — event loot tables + enemy equipment drops (quality penalty).
- **MVP types defined**: 12 weapon types, 18 per-slot armor pieces + 3 multi-slot + 3 shields + 3 accessories.
- **Thrown weapons deferred**: Will be Consumables when added.
- **Rounds 10-15 (deepening)**: 16 additional decisions across 6 rounds.
- **Armor Speed penalties**: Flat per piece (Bonus Modifier). Medium/Heavy pieces impose stacking Speed reduction.
- **Armor Soak families**: All armor contributes to all 3 Soak families at different ratios (Heavy: 60/25/15 Physical/Elemental/Magical, Light: 15/25/60, Medium: 40/35/25).
- **Armor mixing**: Free, no interaction. Each piece independent.
- **Household stash**: Shared storage, no per-character inventory. Items don't degrade in stash.
- **Death & gear**: Preserved on corpse. Returns to stash if permanently lost.
- **Proc effects = Triggers**: Equipment proc affixes use the combat Trigger system (OnHit, etc.). No new mechanic.
- **Ranged Short range penalty**: Accuracy penalty at Short range, mirroring Reach penalty at Medium.
- **Unified affix/Perk data model**: Affixes ARE mini-Perks — same component schema (Stat Adjustments, Triggers, Actions, Damage). Maximum code reuse.
- **Unique items**: Templates with fixed name/lore/some affixes + 1-2 random slots. Chase items from specific sources.
- **Sell or salvage**: Two disposal paths — gold or materials. Material system (tiered + typed) deferred.
- **Tournament equipment lock**: Gear locked at registration. No swaps between rounds.
- **Unarmed as valid choice**: Empty Hand slots = Unarmed build path.
- **Continuous affix quality scaling**: `affix_power = base × tier_multiplier × (quality/100)`. Every quality point matters for affix power, not just tier thresholds.
- **Dual-wield revised**: Two independent attack rolls (not one). Main-hand has small Attack/Damage penalty, off-hand has moderate penalty. Perks (Ambidextrous, Two-Weapon Fighting) mitigate.
- **Versatile two stat profiles**: Each Versatile weapon type defines both one-hand and two-hand stat profiles. Weapon type template carries both.
- **Block proportional split**: Block value splits proportionally across multi-component damage based on relative magnitudes.
- Updated specs:
  - `equipment.md` — 59 decisions across 23 rounds. Major restructure: parallel weapon/focus categories, off-hand bonus attack, implement resolution, enchanting service.
  - `glossary.md` — 12 new terms total (Focus Type, Unfocused, Implement Resolution, Enchanting + earlier terms). 10+ updated (Weapon Type, Dual-Wield, Tag + earlier terms).
  - Downstream flagged: combat (implement type on Actions, off-hand bonus attack pipeline, equipment combat lock, block proportional split, dual-wield as single Action), characters (Combatant Trait provides Unarmed + Unfocused), traits-and-perks (Ambidextrous Perk, cross-implement Perks, DW enhancement Perks), combat-ai (DW = one Action for AI, hybrid builds), economy (enchanting costs, three-tier repair, forging, reroll), groups (NEW: Enchanting service, forging/reroll/repair), data-model (two implement categories, implement type on Actions, variable affix count, Focus type definitions)

### 2026-02-18: Post-Combat Interrogation — 18 Decisions (Complete)

- **Completed post-combat spec** with 18 decisions across 8 interrogation rounds. Status updated from 🟡 to 🟢 Complete.
- **Injury mechanics**: Diminishing returns formula `overkill / (overkill + IR × K)`. New derived stat: **Injury Resistance** (50% Endurance + 30% Luck + 20% Willpower). Tiered weighted random severity. Named injury catalog as content data within structural tiers (minor/major/critical/death). Recovery on game ticks (3–5 / 10–15 / 20–30). Perk-based death immunity.
- **Revived = alive**: Only characters Fallen at combat end get injury checks. Revival fully negates fall risk. Final fall's overkill only.
- **XP distribution**: Equal share + ticks-survived participation bonus. `character_xp = base_share + (participation_bonus × ticks_active / total_ticks)`.
- **Recruitment**: NPC generation from event archetype pool (not capturing defeated NPCs). Flat chance per event type. Declined recruits become Free Agents. Not available in tournaments. Event-defined Star Rating range.
- **Per-event post-combat configuration**: Inherited defaults + overrides model. Each event category defines a default profile; individual event templates override fields. Supports injury/discovery/recruitment/loot/XP toggles plus gold reward/cost and placement scaling.
- **Reward scaling**: Placement-proportional (1st=100%, 2nd=60%, 3rd=40%, 4th+=25%). Applies to gold, XP, loot, recruitment. Injury and discovery are not scaled.
- **Tournament post-combat**: Full post-combat per round. Injuries immediate (persist across rounds, no recovery). Discoveries usable next round. No recruitment.
- **Crowd Appeal**: Tournament-specific only — does not affect standard post-combat rewards.
- Updated specs:
  - `post-combat.md` — Full rewrite: 18 decisions, all phases specified
  - `glossary.md` — 2 new terms (Injury Resistance, Post-Combat Configuration). 2 updated (Injury Check, Overkill).
  - Downstream flagged: characters (Injury Resistance derived stat #17), economy (XP pools, hiring costs, healing costs), tournaments (per-round post-combat, compounding injuries), roster-management (Free Agent generation from declined recruits), data-model (injury catalog, post-combat profile schema)

### 2026-02-18: Combat Rounds 14-19 — 18 Additional Decisions

- **Deepened combat spec** with 18 decisions across 6 interrogation rounds. Total decisions: 100 across 19 rounds.
- **Revival mechanics**: Clean slate on revival (all statuses cleared). Self-save via Dying Blow (self-heal prevents Fallen). Revived summoner's summons stay dead (must re-summon).
- **Summon Control Capacity**: Budget-based Resource Pool (e.g., "Necro Control" = 0.8×Willpower + 0.2×Charisma). Each summon has a control cost. Summoners maintain summons up to capacity; dying summons free capacity.
- **Mutual elimination**: Draw — all teams share 1st place. Consistent with same-tick shared rank rule.
- **Resource depletion**: Only Stamina has Health substitution (Exhaustion). Other resources hard-gate Actions via `resource_gate`.
- **Formula shapes resolved**:
  - Crit: `output = base × (1 + crit_multiplier)` — percentage of base (multiplier is tuning value)
  - Resistance: `value × (1 - Resistance/(Resistance+K))` — percentage with diminishing returns, same formula/K for stacks and damage
  - Stealth break: `break_chance = damage_taken / (stealth_stacks × K)` — linear ratio
- **Status effect floor**: Minimum 1 stack always applies regardless of Resistance. Status decay frozen while Fallen.
- **Fallen Resolution order**: Reverse Initiative order (lowest first). `OnFallen` Triggers fire sequentially.
- **Default Action costs**: Attack costs small Stamina (tuning value). Move, Defend, Search are free.
- **Formula representation**: Structured `{base, weights}` for 95% of formulas; optional `formula_override` expression for complex Perks/spells.
- **Formula modifier scope**: Per-category (attack, damage, defense, healing, etc.) — prevents universal attribute replacement.
- **Revive targeting**: Revive component works on any ally (no-op on living targets; other components still resolve). New `[Fallen]` target tag for Fallen-only Actions. Enables hybrid heal+revive Actions.
- **DoT overkill**: Overkill = abs(final_HP) regardless of damage source.
- Updated specs:
  - `combat.md` — All 18 decisions integrated: new subsections (Summon Control Capacity, Formula Representation, Resource Depletion Rules), expanded existing sections (Fallen State, Critical Hit, Status Effects, Targeting, Stealth). Open Questions: 3 formula shapes resolved (crit, resistance, stealth break), 5 new tuning values added.
  - `glossary.md` — 2 new terms (Summon Control Capacity, Fallen Target Tag). 10 updated (Fallen, Resistance, Critical Hit, Sneaking, Status Effect, Ephemeral Combatant, Effect Component, Formula Modifier Tag, Exhaustion, Overkill).
  - `combat-ai.md` — Revival priority note: Revive Actions can target any ally (hybrid heal+revive AI evaluation).
  - Downstream flagged: data-model (summon control capacity, formula representation, formula modifier scope, Resistance K values)

### 2026-02-18: Combat Rounds 10-13 — 11 Additional Decisions

- **Deepened combat spec** with 11 decisions across 4 interrogation rounds. Total decisions: 82 across 13 rounds.
- **Weapon-type-defined formulas**: No fixed "Physical Attack" or "Magic Attack" derived stats. Each weapon type defines its own attribute blend for Attack Value and Damage Value. Spells define their own formulas independently. Daggers use Speed, warhammers use Might, swords use balanced blends.
- **Zone adjacency**: Center + ring topology. Center adjacent to all 4 cardinals. Ring neighbors: N↔E, E↔S, S↔W, W↔N. Opposite cardinals (N↔S, E↔W) = Long range. Creates meaningful Long range on the default map.
- **Unarmed combat**: Attribute-scaled baseline (Might-based, Blunt type). Functional but weaker than weapons. Perks can enhance (Monk builds).
- **Formula modifier tags**: New Stat Adjustment subtype for attribute substitution (e.g., "Intelligent Strikes" — Intellect replaces Accuracy). Highest value wins on conflicts.
- **DoTs bypass Soak**: DoT damage skips Soak (already gated by Resistance at application). Shields still absorb. Makes Resistance the primary DoT defense.
- **Passive detection timing**: Before each character's turn in Turns Phase (not once per tick). More granular detection benefits natural team coordination.
- **Summons**: Ephemeral Combatants with own Initiative, preset AI, team matching summoner. Die when killed or summoner falls. No post-combat phases.
- **Stun mechanics**: Prevents Initiative gain entirely (0 gain while stacks > 0). Stacks decay by Willpower per tick. Most powerful CC — must be priced accordingly.
- **Context Flags**: `team_sizes: [int]` list (own team first) replaces `team_size`/`opponent_team_size`. Supports 2+ teams naturally.
- Updated specs:
  - `combat.md` — All 11 decisions integrated: new sections (Attack & Damage Formulas, DoT Damage Rules, Passive Detection Timing, Summon Mechanics, Stun Mechanics), zone adjacency details, unarmed combat, formula modifier tags, Context Flags schema update. Implications table updated.
  - `glossary.md` — 1 new term (Formula Modifier Tag). 6 updated (Zone, Roll-vs-Static Defense, Status Effect, Stat Adjustment, Ephemeral Combatant, Combat Context Flags).
  - Downstream flagged: data-model (formula modifier representation, summon templates, weapon-type formula definitions)

### 2026-02-18: Combat Round 9 — 24 Additional Decisions

- **Deepened combat spec** with 24 decisions across 8 interrogation sub-rounds. Total decisions: 71 across 9 rounds.
- **Multi-team support**: 2+ teams of any sizes (free-for-all, asymmetric). Per-team elimination order with ranked placement. Tied elimination = shared rank.
- **Per-damage-type Soak**: Replaced single Soak with 3 family formulas — Physical Soak (60% Endurance + 25% Might + 15% Speed), Elemental Soak (45% Endurance + 35% Awareness + 20% Luck), Magical Soak (55% Willpower + 25% Intellect + 20% Luck). Per-type bonuses via tag-scoped Stat Adjustments.
- **Dual Defense**: Physical Defense (40% Speed + 35% Accuracy + 25% Awareness) for physical attacks, Magic Defense for elemental/magical attacks. Actions designate a primary damage type.
- **Per-component damage resolution**: Each Damage component in an Action resolves through its own Soak + Shield + Resistance calculation independently.
- **Dying Blows**: Deaths deferred to end of tick. Characters at ≤0 HP can still act — their Actions are "Dying Blows" (tracked, OnDyingBlow trigger). Enables "Last Stand" Perks.
- **One turn per tick**: Tick is atomic. Excess Initiative carries over. Fast characters act consecutively across ticks.
- **Starting positions**: Event-type defined via per-team zone lists.
- **Starting Initiative**: Default 0, with offset modifiers from Perks/affixes/events.
- **Deterministic simulation**: Seeded PRNG. Same seed + inputs = identical replay.
- **Combat event stream**: Structured typed events. Combat simulates in full, then presents via summary + logs.
- **Universal crit**: All effect component types can crit (heals, shields, etc.).
- **Universal Penetration**: Single stat applied against whichever family Soak is relevant.
- **Allies skip resistance**: Friendly effects always apply at full strength.
- **Batch-then-triggers AoE**: All AoE targets resolve simultaneously, then all triggers fire together.
- **Unified pipeline**: Component-driven branching (no explicit action type flag).
- **Action-defined damage formulas**: Each Damage component has its own formula field.
- Updated specs:
  - `combat.md` — All 24 decisions integrated, new sections (Per-Damage-Type Soak, Dual Defense, Dying Blows, AoE Resolution, Combat Event Stream, Starting Positions/Initiative). Combat pipeline restructured for unified model.
  - `glossary.md` — 5 new terms (Physical Defense, Physical/Elemental/Magical Soak, Dying Blow, Primary Damage Type, Combat Event Stream). 6 updated (Soak, Penetration, Critical Hit, Resistance, Combat Scoreboard, Tick).
  - Downstream flagged: characters (4 new derived stats), combat-ai (multi-team targeting, Dying Blow AI), equipment (per-type Soak bonuses), tournaments (multi-team formats)

### 2026-02-18: Combat Round 8 — 20 Additional Decisions + Post-Combat Split

- **Deepened combat spec** with 20 decisions across new mechanics, resolution clarifications, and spec ownership changes. Total decisions: 47 across 8 rounds.
- **NEW SPEC: post-combat.md** — Post-combat resolution split from combat.md into its own domain spec. Owns: phased presentation flow, injury/death mechanics, Perk Discovery resolution, recruitment phase (placeholder), loot phase (placeholder).
- **Attrition Ramp**: Anti-stalemate mechanic. After configurable onset tick (~100), stacking global damage bonus (+0.5–1%/tick). Healing unaffected. Per-event-type configuration via Context Flags. No hard tick limit.
- **Shield Mechanics**: Second defense layer after Soak. Shields have own HP pool, stack (oldest-first drain), optional damage type filter. Penetration doesn't affect shields.
- **Combat Scoreboard**: Per-character stat tracking (damage dealt/taken/healed, kills, Actions by type, Fallen/revival events, ticks survived). Feeds post-combat and meta-balance.
- **Fight Phase Signal**: `current_tick / attrition_ramp_onset` ratio for AI fight phase detection and Trigger conditions.
- **Win Condition**: Elimination only (last team standing). Attrition ramp ensures convergence.
- **Turn Order**: Explicitly noted as global Initiative interleaving (no team-alternating).
- **Stamina Regen**: Changed to percentage-based (X% of max pool per tick).
- **Defend Action**: Fixed % boost to Defense + Soak, plus Y% max Stamina burst.
- **Exhaustion**: Clarified 1:1 Health cost ratio (no penalty multiplier).
- **Forced Movement**: Unrestricted by default, fires OnForcedMove triggers, resistance Perks can reduce/prevent.
- **Zone Effects**: Explicitly deferred to Phase 4.
- **Context Flags Schema**: Canonical field list moved from combat-ai.md to combat.md. Added attrition ramp parameters.
- **Perk Discovery Model Change**: Per-Trait end-of-combat roll (not per-Action/Trigger-use). Active Traits only. Formula unchanged but applied per-Trait.
- Updated specs:
  - `combat.md` — All 20 decisions integrated, 4 new sections (Attrition Ramp, Shield, Scoreboard, Fight Phase), Post-Combat Flow replaced with handoff reference
  - `post-combat.md` — **NEW** — Injury mechanics, Perk Discovery, phased presentation
  - `traits-and-perks.md` — Discovery model updated to per-Trait end-of-combat roll, "active Traits only" rule
  - `combat-ai.md` — Context Flags references combat.md, resource_efficiency uses fight phase signal
  - `glossary.md` — 4 new terms (Attrition Ramp, Shield, Combat Scoreboard, Fight Phase Signal), 3 updated (Perk Discovery, Exhaustion, Defend)
  - `tournaments.md` — Post-combat references updated
  - `characters.md` — Post-combat references updated

### 2026-02-17: Combat AI Round 2 — 14 Additional Decisions

- **Deepened combat-ai spec** with 14 decisions across 4 interrogation rounds. Standard Scorer library expanded from 11 to 14 Tactical Scorers (3 new: `consumable_scarcity`, `stealth_awareness`, `future_state_value`).
- **Consumable AI**: Consumables use the standard Gate/Scorer pipeline. New `consumable_scarcity` Scorer penalizes usage unless impactful — scales with remaining uses, impact threshold, and tournament awareness (conserve across rounds).
- **Personality archetype mixing**: All Personality Scorers contribute equally. Contradictory archetypes (e.g., [Aggressive, Protective]) create internally conflicted characters whose behavior shifts based on situational context.
- **Judgment-Gated Lookahead**: AI prediction depth scales with Judgment stat. Low Judgment = 1 tick lookahead, high Judgment = up to 10 ticks. Projects deterministic events only: Effects Phase outcomes (DoTs, regen, buff/debuff expiry) and Initiative timing. NOT threat estimation. Implemented as `future_state_value` Scorer — existing Scorers stay simple (current state only).
- **Movement AI (position_need)**: Updated to evaluate range-gap AND zone density (avoid overcrowding, prefer zones with allies).
- **Team coordination**: Sequential evaluation with state updates. Characters act in Initiative order; board state updates after each. No explicit coordination logic — natural coordination emerges from reacting to current reality.
- **Stealth AI**: New `stealth_awareness` Tactical Scorer. Boosts AoE when stealth enemies exist, boosts Search for high-value targets, reduces single-target damage when best targets are stealthed.
- **Revival value model**: `revival_priority` updated to forward-looking "remaining potential" — evaluates fallen ally's available Actions, resource pools, cooldowns, and remaining fight length.
- **Combat Context Flags**: Explicit metadata passed from combat system to AI at fight start: `{is_tournament, round, total_rounds, consumables_replenish, is_exhibition, pve_tier, ...}`. Any Scorer can query these flags.
- **Fight length revised**: Target match length changed from 15–25 ticks to **25–150 ticks**. Range reflects fight variety: PvE fodder (~25), standard tournament (~50–80), championship finals (~100–150).
- **Performance guidelines**: Soft content authoring guidance — prefer standard Scorers, keep custom Considerations simple, optimize the engine not the content.
- Updated specs:
  - `combat-ai.md` — All 14 decisions integrated, 3 new Scorers added to library, new sections (Consumable AI, Context Flags, Lookahead, Team Coordination, Performance Guidelines), edge cases expanded
  - `combat.md` — Fight length 15–25 → 25–150 ticks, Context Flags section, sequential team evaluation, Judgment controls 3 parameters
  - `glossary.md` — Added 5 new terms (Combat Context Flags, Consumable Scarcity, Stealth Awareness, Future State Value, Lookahead Depth), updated Judgment and Personality Archetype
  - `tournaments.md` — Flagged for fight length pacing revision

### 2026-02-17: Combat AI Spec — Utility AI System (Complete)

- **Major rewrite**: Replaced priority-list model with **Utility AI** system. Status updated from 🟡 to 🟢 Complete. All 8 original open questions resolved.
- **Utility AI model**: Each turn, AI evaluates all available Actions via weighted Considerations (Gates + Scorers), blends Tactical and Personality scores, selects via weighted random with Judgment-controlled sharpness.
- **Judgment derived stat**: New derived combat stat (40% Awareness + 35% Willpower + 25% Intellect). Controls tactical-vs-personality blend (~0.3 to ~0.95) and selection sharpness (~1 to ~10). Added to combat.md derived stats.
- **Consideration system**: Two types — Gates (binary pass/fail prerequisites) and Scorers (continuous 0.01–1.0 preference signals). Initial standard library: 4 Gates + 11 Tactical Scorers + 5 Personality Scorers. Score composition via geometric mean normalization.
- **Two-track scoring**: Tactical Score (objective effectiveness) + Personality Score (character temperament). Blended by Judgment. Even at minimum Judgment, 30% tactical weight prevents most suicidal decisions.
- **Personality archetypes**: Core Traits can be tagged with personality archetypes (Aggressive, Cautious, Protective, Vindictive, Showoff). Feed the Personality Score track. Characters without personality tags get neutral personality.
- **Perk-embedded AI**: Each Perk Action definition includes an `ai` block (gates, tactical_scorers, personality_scorers). Content authors define AI behavior alongside ability mechanics.
- **Player config as weight overrides**: Target priority, range preference, Action preferences, consumable triggers — all map to weight multipliers on the scoring system. Phased rollout unchanged (MVP 0: no config, MVP 1: basic, Phase 4: full).
- **NPC AI parity**: NPCs use the same Utility AI system. Difficulty comes from Judgment stat values and available Perks, not separate code paths.
- **Personality as soft bias**: Personality modifiers are soft biases via scoring, not hard overrides. Judgment controls influence strength.
- **Edge cases resolved**: All Gates vetoed (default Actions always pass), AoE friendly fire (net value scoring), resource conservation (fight-phase-aware), action repetition (anti-repetition Scorer), personality vs survival (minimum tactical weight).
- Updated specs:
  - `combat.md` — Added Judgment derived stat, updated combat-ai implications
  - `traits-and-perks.md` — Added AI Considerations to Perk Action data model, personality archetype tags on Core Traits
  - `glossary.md` — Added 10 new terms: Judgment, Consideration, Gate, Scorer, Tactical Score, Personality Score, Utility Score, Selection Sharpness, Response Curve, Personality Archetype

### 2026-02-16: Combat Spec Interrogation (7 Rounds — Complete)

- Resolved 27 decisions across 7 rounds. Status updated from 🔄 to 🟢 Complete.
- **Match length**: 15–25 ticks target. Fast, decisive fights — attrition comes from multi-round tournament structure.
- **Percentage Soak**: Replaced flat Soak with `Reduction % = Soak / (Soak + K)` diminishing returns formula. Added Penetration as counter-stat.
- **Critical hits**: Extra damage roll after successful hit. Uses Physical Crit / Magic Crit derived stats (Awareness + Luck + Accuracy/Intellect).
- **Type-ordered effect resolution**: Status/debuff → damage/healing → movement within a single Action. **Overrides** traits-and-perks "simultaneous" model. Updated traits-and-perks spec to reference combat.
- **Global Initiative Multiplier**: `sqrt(Speed) × ~3.0` per tick. Match-pacing knob independent of character balance.
- **Action Speed as flat modifier**: Fast action +30 = -70 Initiative cost; slow action -40 = -140 cost. Default Actions have Action Speed 0.
- **Tick order**: Effects → Initiative → Turns. DoTs/decay happen before anyone acts.
- **Resistance: dual reduction**: Status always applies (no binary resist). Resistance reduces stacks AND damage of that type.
- **Haste/Slow dual channel**: Speed modification (permanent) and Initiative gain modification (temporary) are separate, stackable channels.
- **Stealth rework**: AoE hits Sneaking characters. Non-stealth Actions break stealth; stealth-tagged Actions don't. Damage has chance to break stealth. Passive detection (same-zone, automatic) + Active detection (Search Action, map-wide with range penalties).
- **No default regen for Trait-granted resources**: Mana, Faith, etc. have no passive regen — comes from Perks/Triggers. Stamina retains slow passive regen.
- **Hidden Combatant system Trait**: Level 1, invisible, immutable. Provides Attack, Defend, Move, Search as standard Perk Action components. No special-case code for default actions.
- **Fallen revival**: Yes, via Perks, consumables, rare equipment. Revive at fractional HP.
- **Overkill → injury severity**: Excess damage tracked, increases injury chance and severity.
- **Tiered injury checks**: Two rolls — injury yes/no, then severity (minor/major/critical/death). Modified by overkill, Endurance, Luck.
- **Phased post-combat presentation**: Combat resolution → injury/death → Perk discovery (accept/reject) → recruitment → loot.
- **Binary range**: In-range or not, no damage falloff.
- **Formula-based scaling multipliers**: Derive from desired stat ranges, not hand-tuned.
- **Canonical vocabularies defined**: Target tags (10 MVP tags), Trigger event types (24 events across 8 categories), effect component types (11 types with parameters).
- **MVP scope**: All 10 damage types + default 5-zone map.
- **Deferred**: Channeled abilities/interrupts (Phase 2+), Morale (Phase 4), zone variants (Phase 4).
- Updated traits-and-perks spec: "simultaneous" effect resolution → "type-ordered" per combat decisions. Resource regen note updated.
- Updated glossary: Soak, Initiative, Action Speed, Sneaking, Defend, Rider Effect revised. Added: Penetration, Critical Hit, Search, Combatant System Trait, Effect Resolution Order, Overkill, Injury Check, Resistance, Global Initiative Multiplier.
- Flagged downstream specs:
  - `combat-ai` — stealth/detection trade-offs, Action Speed evaluation, revival priority, resource conservation, 20-40+ Actions per character
  - `equipment` — Penetration as affix stat, crit bonuses, stealth-tagged effects
  - `tournaments` — exhibition vs non-exhibition injury handling, overkill severity, phased post-combat per round
  - `data-model` — hidden Trait type, injury roll data, overkill tracking, expanded vocabulary catalogs
  - `meta-balance` — combat outcomes feed underdog balancing

### 2026-02-16: Traits & Perks Spec Interrogation (Round 6)

- Resolved 12 game design, consequence, and new idea questions across 3 sub-rounds. Status remains 🟢 Complete.
- **Discovery rate: no cap**: No hard cap on discovery rate. Small tree size (2-4 discoverable Perks) is the natural limit. At max investment, ~30-40% chance per combat of discovering a Perk is intended — that's the reward for discovery-optimized builds.
- **Post-combat discovery resolution with rejection**: Discovery rolls happen during combat, but resolution is post-combat (alongside injury checks, NPC recruitment, loot). Player can accept or reject each discovered Perk. Rejected Perks remain in the discovery pool.
- **Tradeoff Perks via existing components**: Perks can have negative Stat Adjustments and harmful Triggers as built-in tradeoffs. No new Drawback component type — the existing system supports negative values natively. Both positive and negative components scale with level.
- **Multi-tag additive stacking**: Tag-scoped percentage bonuses from different tags on the same Action stack additively. +10% [Fire] + 15% [Arcane] = +25% total on a [Fire, Arcane] Action.
- **Equal category combat power**: No Trait category "owns" combat. Core, Role, and Bond Traits can all provide combat-relevant abilities. A character with strong Core + Bond and no Role Traits is a valid combat build.
- **No Action count limit**: No mechanical limit on Actions per character. AI evaluates all available Actions; situational filtering narrows choices. Flagged as key challenge for combat-ai spec.
- **Intentional power gap**: Large power difference between star ratings is intended. Gap is breadth (slot count), not depth (individual Trait/Perk power — 1★ characters can have 5★ Traits/Perks). Star-gated tournaments manage matchmaking.
- **Build diversity philosophy**: Three forces + one system: economic scarcity, content variety, roster pressure, and automatic underdog balancing (win/loss ratio tracking per Trait with automatic bonuses for underperforming Traits).
- **NEW SPEC: Meta-Balance**: Automatic underdog balancing is a cross-cutting system requiring a new `architecture/meta-balance.md` spec.
- **Prerequisite graph**: No cycles is the only hard constraint. No max depth. Cross-Trait prerequisites remain allowed. Content validation catches cycles at authoring time.
- Updated glossary: Perk Discovery (post-combat resolution, accept/reject, no rate cap).
- Flagged downstream specs:
  - `combat` — discovery collected during combat, resolved post-combat; tradeoff Perks; multi-tag additive stacking
  - `combat-ai` — no Action count limit (20-40+ Actions per character); equal combat power from all categories
  - `tournaments` — star-gated entry; per-combat cooldown reset
  - `characters` — post-combat phase includes discovery resolution
  - `meta-balance` (NEW) — automatic underdog balancing system

### 2026-02-16: Traits & Perks Spec Interrogation (Round 5)

- Resolved 5 implementation-precision questions. Status remains 🟢 Complete.
- **Tag-scoped bonuses: flat or percentage**: Tag-scoped Stat Adjustments can be flat (+5 Soak vs [Poison]) or percentage (+10% [Fire] damage). Multiple percentage bonuses stack additively. Application order: flat bonuses first, then combined percentage as a single multiplier on already-level-scaled values.
- **Pre-multiplier bonus pool capacity**: Bonus pool capacity from Stat Adjustments (e.g., "+50 Mana pool") adds to the weighted attribute blend before the per-stat scaling multiplier. The multiplier amplifies both base and bonus together, making bonus capacity a powerful investment.
- **CHANGE: Trait Level Multiplier replaces Bond Level Bonus in Perk Discovery**: The parent Trait's level now multiplies discovery chance using the standard amplification curve (×1.0–×2.0). This is universal across all Trait categories — no Bond-to-Trait association needed. Formula: `(Base Rate + Luck Bonus + Perk Bonuses) × Trait Level Multiplier`. Perk-granted discovery bonuses scale with level (consistent with all-numeric-outputs-scale rule).
- **Per-combat cooldown reset**: All cooldowns reset fully between fights. No cross-fight cooldown persistence, even in multi-round tournaments. Consistent with pools starting at full capacity.
- **Automatic redundant access for shared Perks**: When a character acquires a Trait containing an already-owned Perk, redundant access is granted automatically — no separate purchase needed. Perk Discovery rolls skip already-owned shared Perks.
- Updated glossary: Stat Adjustment (flat/% tag-scoped, application order), Action (per-combat cooldown), Resource Pool (pre-multiplier bonus), Perk Discovery (Trait Level Multiplier formula).
- Flagged downstream specs:
  - `combat` — pre-multiplier bonus pool capacity, tag-scoped % stacking and application order, per-combat cooldown reset, updated Perk Discovery formula (Trait Level Multiplier)
  - `groups` — Bond level no longer has a special Perk Discovery bonus (replaced by universal Trait Level Multiplier)
  - `data-model` — tag-scoped Stat Adjustment now has `{tag, stat_key, value, value_type: "flat"|"percentage"}` representation

### 2026-02-15: Traits & Perks Spec Interrogation (Round 4)

- Resolved 14 additional Perk component data model questions. Status remains 🟢 Complete.
- **Tag-based Action targeting**: Actions specify target tags (e.g., `[Enemy, Single]`, `[Ally, AoE-Zone]`). Combat spec resolves tags to battlefield targets. New targeting modes = new tags.
- **Effect component list model**: Actions and Triggers express outputs as typed effect component lists (Damage, Heal, ApplyStatus, RemoveStatus, Move, Summon, Shield + extensible). Components have type tag + key-value parameters.
- **Simultaneous intra-Action effect resolution**: All effect components within a single Action/Trigger resolve simultaneously — consistent with multi-Trigger resolution model.
- **Shared effect model**: Triggers use the exact same effect component list as Actions. Only difference is activation (automatic vs. chosen).
- **Stat Adjustments: flat + tag-scoped**: Two subtypes — flat bonuses (any numeric stat: `+5 Might`, `+50 Mana pool`) and tag-scoped bonuses (conditional: `+10% [Fire] damage`, `-15% [Martial] resource cost`). Can modify any numeric value in the system.
- **Perk Discovery additive modifiers**: Base ~0.1% + flat bonuses from Luck, Bond level, and specific Perks. Exact values are tuning.
- **Bond Trait hybrid Perks**: Bond Perks grant both utility (discounts, access) and combat capabilities (themed Actions, Stat Adjustments). Not pure utility tax.
- **Equipment stays equipped after respec**: Consistent with acquisition-only gates. Requirements checked at equip time only.
- **All numeric outputs scale**: Universal rule — output values scale with level multiplier, costs/cooldowns stay flat. No per-component exceptions.
- **Illustrative Trigger event types**: OnHit, OnHitBy, OnKill, OnAllyFallen, OnCombatStart, OnTurnStart, OnTurnEnd, OnStatusApplied, OnResourceDepleted. Full vocabulary owned by combat.
- Updated glossary: Effect Component (new), Target Tag (new), Action (targeting + effects), Stat Adjustment (flat + tag-scoped), Trigger (event types + shared effects), Perk Discovery (additive modifiers).
- Flagged downstream specs:
  - `combat` — must define: canonical target tag vocabulary, Trigger event vocabulary, effect component type catalog; simultaneous intra-Action resolution; discovery modifier handling
  - `combat-ai` — must parse target tags, evaluate tag-scoped bonuses when choosing Actions
  - `equipment` — acquisition-only requirement checks (gear stays on respec)
  - `data-model` — Action/Stat Adjustment/Trigger structured representations, effect component generic format
  - `groups` — Bond combat Perks increase Group combat relevance; Bond level affects discovery chance
  - `economy` — discovery modifier stacking monitoring

### 2026-02-14: Traits & Perks Spec Interrogation (Round 3)

- Resolved 12 additional implementation-edge-case questions. Status remains 🟢 Complete.
- **Shared Perk amplification**: When the same Perk is owned from two Traits, the higher Trait's amplification multiplier applies.
- **Multi-resource Actions**: Actions can cost any combination of resource types (e.g., Mana + Stamina).
- **Perk Discovery cap override**: Applies to acquisition moment only. Further manual leveling still capped by Trait level.
- **No star gate for Traits**: Character Star Rating only determines slot count — doesn't restrict which Traits can be acquired.
- **Perk scaling curve**: Same as Trait amplification (×1.0/×1.2/×1.4/×1.7/×2.0). Max stack at 5★/5★ = 4.0×.
- **MAJOR: Resource pools are now emergent from Perk content** (not Trait-level assignment). Open/extensible resource type system — 5 default types (Mana, Faith, Spirit, Focus, Stamina), content authors can define more. Pool activates when first Perk references it (attribute formula = base, Perks add bonus capacity). Perks auto-tagged by resource type.
- **Soft component guideline**: No hard limit on Actions/Stat Adjustments/Triggers per Perk. Content guidelines suggest 1–3.
- **Empty categories allowed**: Characters can respec down to zero Traits in any category.
- **One Bond per Group**: Characters hold one Bond Trait per Group; it levels to deepen affiliation.
- **Simultaneous Trigger resolution**: All qualifying Triggers fire together, effects collected and applied with no ordering.
- **Full tree discovery**: Silently wasted roll.
- Updated glossary: Resource Family → Resource Pool (emergent, open/extensible), Perk (scaling curve, shared amplification), Perk Discovery (cap override clarification), Bond Trait (one per Group), Tag (auto-tags from resources).
- Flagged downstream specs:
  - `combat` — open resource pool system, multi-resource Actions, simultaneous Trigger resolution, updated amplification curve
  - `combat-ai` — multi-resource Action cost evaluation
  - `groups` — one Bond Trait per Group per character, each Group defines exactly one Bond Trait
  - `data-model` — resource types are content data (not schema), Perk components need structured typed representation

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
_2026-02-21 — Equipment cleanup round 31: 103 decisions total. Deadly/Sunder per-implement, DW penalties (−15%/−35%), armor Speed/scaling locked, percentage-based penalties. Status → 🟢 Complete. Seven specs 🟢 Complete._
