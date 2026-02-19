# Post-Combat — Domain Specification

**Status**: 🟢 Complete (8 interrogation rounds, 18 decisions)
**Last interrogated**: 2026-02-18
**Last verified**: —
**Depends on**: [combat](combat.md), [characters](characters.md), [traits-and-perks](traits-and-perks.md)
**Depended on by**: [tournaments](tournaments.md), [roster-management](roster-management.md), [economy](economy.md)

---

## Overview

Post-combat is the resolution phase after a fight concludes. When combat declares victory or defeat (last team standing, or mutual elimination draw), the post-combat system takes over to resolve consequences: injuries, Perk discovery, recruitment opportunities, and loot distribution. This phase is presented as a sequential dramatic narrative, not a bulk summary.

**All teams** go through post-combat, not just the winner. The difference is in what each team receives (reward scaling by placement). Post-combat receives data from the combat system: the [Combat Scoreboard](combat.md#combat-scoreboard), Fallen characters with overkill values, active Traits list, and [Combat Context Flags](combat.md#combat-context-flags).

---

## Core Concepts

### Phased Presentation Flow

Post-combat resolves as a **phased dramatic presentation**:

1. **Combat Resolution**: Final blow, victory/defeat declared. Summary statistics from the Combat Scoreboard.
2. **Injury & Death Phase**: Fallen characters' fates revealed one by one (injury rolls, severity outcomes).
3. **Perk Discovery Phase**: Discovery results presented per qualifying Trait (accept/reject per discovery).
4. **Recruitment Phase**: If applicable, a chance to recruit a newly generated Named NPC from the event's archetype pool.
5. **Loot Phase**: Equipment, gold, XP, and other rewards distributed.

Each phase is presented sequentially for dramatic effect — the player experiences the aftermath as a narrative sequence, not a bulk summary. Phases with no content (e.g., no Fallen characters → skip injury phase) are skipped silently.

### Per-Event Post-Combat Configuration

Post-combat phases are **per-event-type configurable**. Each event category (tournament, PvE, exhibition, vendor practice) has a **default post-combat profile** that individual event templates can override:

```
post_combat_profile: {
  injury_enabled: bool,
  discovery_enabled: bool,
  recruitment_enabled: bool,
  recruitment_chance: float,
  recruit_star_range: [int, int],
  loot_enabled: bool,
  xp_enabled: bool,
  gold_reward: int | null,    // reward amount (null = no gold reward)
  gold_cost: int | null,      // cost to participate (null = free)
  reward_scale: { 1: 1.0, 2: 0.6, 3: 0.4, default: 0.25 }
}
```

**Examples**:
- **Tournament match**: injury on, discovery on, recruitment off, loot on, XP on, gold reward on
- **Exhibition**: injury off, discovery on, recruitment off, loot reduced, XP on, gold reward reduced
- **Vendor training practice**: injury off, discovery on, recruitment off, loot off, XP on, gold cost (paid to vendor)
- **Bandit clearance job**: injury on, discovery on, recruitment on, loot on, XP on, gold reward on

This is an **inherited defaults + overrides** model: each event category defines a default profile, and individual event templates override specific fields as needed.

### Reward Scaling by Placement

All teams go through post-combat, but **rewards scale by placement** in the combat's elimination order:

| Placement | Reward Scale |
|-----------|-------------|
| 1st place | 100% |
| 2nd place | 60% |
| 3rd place | 40% |
| 4th+ | 25% |

Reward scale applies to gold, XP bonus, loot drop rates, and recruitment chance. Injury checks and Perk Discovery are **not** scaled — they apply at full rate regardless of placement. These percentages are tuning values.

In a **mutual elimination (draw)**, all surviving teams share 1st place and receive 100% rewards (consistent with combat's shared-rank rule).

### Injury Mechanics

After combat resolves, Fallen characters undergo a tiered injury check (if injury is enabled for this event type).

**Who gets checked**: Only characters in the **Fallen state at combat end**. Characters who fell during combat but were **revived** before the fight ended are treated as alive — revival fully negates fall risk. If a character fell multiple times (fell, revived, fell again), only the **final fall's overkill** is used.

**Injury Resistance** — a new derived stat:
- **Formula**: 50% Endurance + 30% Luck + 20% Willpower (× scaling multiplier)
- **Perk bonuses**: Specific Perks can add flat bonuses to Injury Resistance
- Endurance represents physical toughness, Luck provides fortune, Willpower represents grit and the will to survive

**Tier 1: Injury Roll**
- Determines if an injury occurs at all
- Formula: `injury_chance = overkill / (overkill + IR × K)` where IR = Injury Resistance and K is a tuning constant
- 0 overkill = 0% injury chance. Equal overkill and IR×K = 50% chance. Massive overkill approaches but never reaches 100%.
- Consistent with combat's diminishing returns formulas (Soak, Resistance)
- **No injury**: Character returns to Available state
- **Injury**: Proceed to Tier 2

**Tier 2: Severity Roll**
- **Weighted random** draw from severity tiers, influenced by overkill magnitude and Injury Resistance
- Higher overkill shifts weights toward worse outcomes; higher IR shifts toward better outcomes
- The exact weight formula is a tuning value — the spec establishes "weighted random influenced by overkill and IR" without defining the formula shape
- Even small overkill has a tiny death chance; massive overkill heavily favors critical/death

**Severity Tiers** (structural — each contains content-defined named injuries):

| Tier | Effect | Recovery |
|------|--------|----------|
| **Minor** | Temporary stat penalty. Named types: Bruised, Strained, Winded | 3–5 game ticks |
| **Major** | Significant stat penalty, longer duration. Named types: Broken Bone, Deep Wound, Concussion | 10–15 game ticks |
| **Critical** | Permanent Potential reduction OR anatomical slot loss. Named types: Scarring (Charisma), Limb Loss (slot), Trauma (Willpower), Brain Damage (Judgment) | 20–30 game ticks (if recoverable) |
| **Death** | Character enters Dead state | Permanent (Raise Dead service required) |

**Named injury types** are **content data** within structural severity tiers — drawn from a weighted list per tier. New injuries can be added without code changes. Each named injury specifies: affected stats, penalty magnitude, recovery time (in game ticks), and any special rules.

**Recovery**: Injuries recover passively over **game ticks** (the global downtime cycle, not combat ticks). Healing services from Groups can accelerate recovery or provide instant cures. Within a multi-round tournament, **no game ticks pass between rounds**, so injuries persist across the entire event.

**Death prevention**: Specific rare Perks can grant **death immunity** — their worst possible outcome is critical injury instead of death. This is a meaningful defensive build investment. Without such Perks, any Fallen character in a non-exhibition event can die.

### Perk Discovery

Perk Discovery uses a **single end-of-combat check per qualifying Trait**, not per-Action/Trigger-use during combat. Fires only if discovery is enabled for this event type.

**Eligibility**:
- Only **active Traits** qualify — Traits that had at least one Action used or Trigger fire during the combat (determined from the Combat Scoreboard's "Actions Used by type" data)
- Traits where only passive Stat Adjustments were active do NOT qualify
- The hidden Combatant system Trait does NOT qualify for discovery (it has no discoverable Perks)

**Per-Trait Discovery Roll**:
- One roll per qualifying Trait
- Formula: `(Base Rate + Luck Bonus + Perk Bonuses) × Trait Level Multiplier`
  - **Base Rate**: Tuning value (higher than the old per-use 0.1% to produce similar per-fight discovery rates)
  - **Luck Bonus**: Small flat bonus from Luck Attribute
  - **Perk Bonuses**: Flat bonuses from specific Perks (e.g., "Scholar's Insight")
  - **Trait Level Multiplier**: Standard amplification curve (×1.0 / ×1.2 / ×1.4 / ×1.7 / ×2.0)
- If the Trait's tree has no unowned Perks, the roll is silently skipped (full tree = no discovery possible)
- **Rarity weighting**: When a discovery succeeds, the specific Perk discovered is rarity-weighted — lower-minimum-star Perks are more likely than higher-minimum-star ones

**Resolution**:
- Each discovered Perk is presented to the player individually
- **Accept**: Perk is acquired at its minimum star level, free of XP cost. Overrides the Perk level cap at acquisition (further leveling capped by parent Trait level)
- **Reject**: Perk is not acquired. It remains in the discovery pool for future rolls. No penalty for rejection.
- Multiple discoveries can occur from a single combat (from different Traits). Each is presented individually.

See [traits-and-perks](traits-and-perks.md) for the Perk Discovery system design and modifier sources.

### XP Distribution

XP is calculated per-character using an **equal share + participation bonus** model:

```
character_xp = base_share + (participation_bonus × ticks_active / total_ticks)
```

- **Base share**: Total XP pool for the event divided equally among all team members. Every character gets this regardless of performance.
- **Participation bonus**: A smaller additional XP pool distributed by **ticks survived ratio**. Characters who survived the full fight get the full bonus; characters who fell early get proportionally less. Revived characters count all ticks they were active (across multiple active periods).
- **Fallen characters**: Still receive the base share (they participated) but get reduced participation bonus proportional to how early they fell.
- **Placement scaling**: Both base share and participation bonus are scaled by placement reward percentage (1st = 100%, 2nd = 60%, etc.).

The total XP pool for an event is determined by event type and difficulty — defined in [economy](economy.md).

### Recruitment Phase

Recruitment generates **new Named NPCs** from the event's archetype pool — you do not recruit defeated Named NPCs or other players' characters.

**How it works**:
- After eligible fights (if recruitment is enabled for this event type), a single roll determines if a recruitable Named NPC is generated
- **Chance**: Flat base percentage per event type (e.g., exhibition 15%, PvE 10%, vendor practice 20%), modified by team Charisma or Luck. One roll per fight, not per enemy.
- **Template**: The event configuration specifies which Group/archetype pool the enemies came from. The recruitable NPC is rolled from that same archetype pool — fight bandits, potentially recruit a bandit-archetype character.
- **Star Rating**: Event-defined range (e.g., PvE tier 1 = 1★, PvE tier 3 = 1–2★). Higher difficulty events can produce better recruits.
- **Not in tournaments**: Tournament matches do not generate recruitment candidates.

**Resolution**:
- If a recruit is generated, they are presented to the player with full stat visibility
- **Accept**: Pay the hiring cost (gold, defined in [economy](economy.md)) to add the character to your roster
- **Decline**: The NPC becomes a **Free Agent** — added to the persistent Free Agent pool, recruitable later via other means. This is a primary source of Free Agent generation.

### Loot Phase

The final phase distributes material rewards from combat.

**Scope**: Loot distribution formulas (gold amounts, equipment drop rates, loot tables) are fully defined in [economy](economy.md) and [equipment](equipment.md). This spec defines only that:
- Loot phase fires after recruitment
- Rewards are scaled by placement (1st = 100%, 2nd = 60%, etc.)
- Reward types include: gold, equipment drops, consumables, materials, metacurrency (tournaments)
- Crowd Appeal does **not** affect standard post-combat rewards — Crowd Appeal mechanics are tournament-specific

### Tournament Post-Combat

In multi-round tournaments, **full post-combat fires after each round**:

- **Injury**: All effects apply **immediately**. A critical injury (Potential reduction, slot loss) in round 1 means round 2 is fought with reduced stats. No game ticks pass between rounds, so injuries do not recover during the tournament.
- **Perk Discovery**: Fires between rounds. Accepted Perks are **usable immediately** in subsequent rounds. Mid-tournament power spikes are possible (but Perks start at minimum star level, limiting impact).
- **Recruitment**: Not available in tournament matches.
- **Loot/XP**: Distributed per round.

This creates dramatic tournament arcs — a team that takes heavy casualties in early rounds faces compounding disadvantage, while discovery between rounds can provide clutch power boosts.

---

## Decisions

### Phased Post-Combat Presentation

- **Decision**: Post-combat flow is a sequential dramatic presentation: combat resolution → injury/death → Perk discovery → recruitment → loot.
- **Rationale**: Sequential presentation creates narrative moments and gives each phase appropriate attention. Bulk summaries feel rushed and don't generate player attachment.
- **Implications**: UI must support phased presentation. Each phase can be expanded with detail as the game matures.

### Tiered Injury Checks

- **Decision**: Post-combat injury uses two rolls: first determines if injury occurs, second determines severity. Both are modified by overkill, Injury Resistance, and Perks.
- **Rationale**: Two-tier system creates more nuanced outcomes than a single roll. Most Falls result in no injury (minor events). When injuries do occur, the severity spectrum (minor → major → critical → death) provides dramatic range.
- **Implications**: Characters built for durability (high Endurance, Luck, Willpower, injury-resistance Perks) have a meta-advantage beyond just surviving combat.

### Overkill Affects Injury Severity

- **Decision**: Excess damage beyond 0 HP (overkill) is tracked by combat and increases injury severity in the post-combat injury roll. Only the **final fall's overkill** is used — previous falls that were healed by revival don't accumulate.
- **Rationale**: Creates meaningful stakes for getting hit hard. A character barely knocked out vs. one obliterated have different injury expectations. Final-fall-only keeps the rule simple and avoids punishing characters who were revived mid-fight.
- **Implications**: Equipment and Perks that reduce spike damage (shields, damage caps) have indirect injury-prevention value.

### Perk Discovery — Per-Trait End-of-Combat Roll

- **Decision**: Perk Discovery uses a single end-of-combat check per qualifying Trait, not per-Action/Trigger-use during combat. One roll per Trait. Only Traits that had at least one Action used or Trigger fire qualify ("active Traits").
- **Rationale**: Per-use rolls during combat were implementation-complex and created a bias toward high-Action-count builds. A single per-Trait roll is simpler, fairer, and creates a cleaner handoff to the post-combat phase.
- **Implications**: Discovery probability per Trait per fight is higher than the old per-use base rate (retuned to produce similar per-fight discovery rates).
- **Alternatives considered**: Per-Action/Trigger-use during combat (original model — rejected for complexity and Action-count bias), per-character single roll (rejected — would not reward diverse Trait investment).

### Post-Combat as Separate Spec

- **Decision**: Post-combat resolution lives in its own spec, separate from combat.md.
- **Rationale**: Post-combat is a distinct gameplay phase with its own mechanics, UI flow, and dependencies (economy, roster-management, groups). Separating it reduces combat.md's scope and allows each phase to evolve independently.
- **Implications**: Combat.md ends at victory/defeat and hands off structured data. This spec owns injury checks, discovery resolution, recruitment, and loot.

### Injury Formula: Diminishing Returns

- **Decision**: Tier 1 injury chance uses `overkill / (overkill + IR × K)` where IR = Injury Resistance and K is a tuning constant. Consistent with combat's Soak and Resistance formula shapes.
- **Rationale**: Diminishing returns provides a single consistent formula shape across the entire system. Players learn one curve and understand both combat defense and injury mechanics. The K constant allows independent tuning without changing stat scaling.
- **Implications**: Injury Resistance is a new derived stat that must be added to the character model. The K constant is a tuning value.

### Injury Resistance: New Derived Stat

- **Decision**: Injury Resistance = 50% Endurance + 30% Luck + 20% Willpower (× scaling multiplier). Perk bonuses add on top.
- **Rationale**: Endurance is physical toughness (primary), Luck provides fortune, Willpower represents mental grit. Three attributes means diverse builds can invest in injury resistance without it being a single-attribute gate.
- **Implications**: Characters spec needs Injury Resistance in the derived stats table. This is derived stat #17 (combat) or a post-combat derived stat.

### Revived Characters: No Injury Check

- **Decision**: Only characters in the Fallen state at combat end undergo injury checks. Characters who fell during combat but were revived before the fight ended are treated as alive. Revival fully negates fall risk.
- **Rationale**: Revival is an expensive combat investment (resource cost, turn cost, build investment). It should fully protect against post-combat consequences. If revived characters still got injury checks, revival would feel incomplete.
- **Implications**: The combat system must track which characters are Fallen at combat end (already tracked). Revived characters are simply not in the Fallen list.

### Severity: Weighted Random from Overkill

- **Decision**: Tier 2 severity uses a weighted random draw. Higher overkill shifts weights toward worse outcomes; higher Injury Resistance shifts toward better outcomes. The exact weight formula is a tuning value.
- **Rationale**: Weighted random provides more variance and drama than fixed thresholds. Even small overkill has a tiny death chance (keeps stakes real). Massive overkill heavily favors critical/death outcomes. The formula shape is left as a tuning value for maximum implementation flexibility.
- **Implications**: Implementation needs a weighted random system. Balance must tune weight curves to produce appropriate severity distributions.

### Injury Catalog: Hybrid (Structural Tiers + Content Injuries)

- **Decision**: The 4 severity tiers (minor/major/critical/death) are structural. Within each tier, specific named injuries are content data drawn from a weighted list. Named injuries specify affected stats, penalty magnitude, recovery time, and special rules.
- **Rationale**: Structural tiers provide a clean severity spectrum for the formula system. Content injuries within tiers allow variety and thematic depth without code changes. Consistent with the Perk/Status effect philosophy of "everything is content data."
- **Implications**: Data model needs an injury type catalog (name, tier, effects, recovery time). Content authors can add new injury types per tier. Balance must ensure each tier has enough variety to feel non-repetitive.

### Death Prevention: Perk-Based Immunity

- **Decision**: Specific rare Perks can grant death immunity — their worst possible outcome from an injury roll is critical injury instead of death. Must be earned and invested in.
- **Rationale**: Death immunity is a meaningful defensive build choice. Players who invest in death-prevention Perks sacrifice other Perk slots for security. Without such Perks, any Fallen character can die — keeps stakes real for most characters.
- **Implications**: Death immunity is a boolean flag on the character, set by Perks. The Tier 2 severity roll checks this flag and caps the outcome at critical if set.

### Recovery: Fixed Game Ticks Per Tier

- **Decision**: Injury recovery times are fixed per severity tier: Minor = 3–5 game ticks, Major = 10–15 game ticks, Critical = 20–30 game ticks (all tuning values). Recovery happens on the game tick time scale, not combat ticks. Healing services from Groups can accelerate or instant-cure.
- **Rationale**: Fixed ticks per tier is simple and predictable. Players can plan around known recovery times. Game ticks are the natural time scale for out-of-combat events. Healing services provide an economic pressure valve (pay gold to recover faster).
- **Implications**: Within tournaments, no game ticks pass between rounds — injuries persist across the entire event. Recovery time is a tuning value per tier. Healing services are defined in Groups/economy specs.

### All Teams Get Post-Combat

- **Decision**: All teams go through the full post-combat flow regardless of outcome (win, loss, or draw). The difference is in what each team receives — rewards are scaled by placement.
- **Rationale**: Both winning and losing teams' Fallen characters need injury checks. Both teams can learn from combat (Perk Discovery). Losing shouldn't be purely punishing — you still progress, just slower. This is consistent across all event types.
- **Implications**: Post-combat must handle placement-based reward scaling. UI must support post-combat for losing teams (who might have fewer phases to show).

### Reward Scaling: Placement-Proportional

- **Decision**: Rewards scale by placement: 1st = 100%, 2nd = 60%, 3rd = 40%, 4th+ = 25% (tuning values). Applies to gold, XP bonus, loot drop rates, and recruitment chance. Injury checks and Perk Discovery are not scaled.
- **Rationale**: Placement-proportional scaling rewards survival and outlasting opponents, even if you don't win. Generalizes cleanly to multi-team fights. Not scaling injury/discovery ensures everyone experiences consequences and learning.
- **Implications**: Tournament formats with ranked elimination order naturally feed into this system. The percentages are tuning values.

### XP Distribution: Equal Share + Participation Bonus

- **Decision**: `character_xp = base_share + (participation_bonus × ticks_active / total_ticks)`. Base share is equal for all team members. Participation bonus scales with ticks survived. Fallen characters get base XP but reduced participation bonus.
- **Rationale**: Equal base share ensures all characters progress. Ticks-survived ratio rewards survival without creating perverse incentives (no kill-stealing, no damage-racing). Simple, trackable from Combat Scoreboard data.
- **Implications**: Total XP pool per event is defined in economy spec. Combat Scoreboard tracks ticks survived per character. Revived characters count all active ticks.

### Crowd Appeal: Tournaments Only

- **Decision**: Crowd Appeal does not affect standard post-combat rewards. Crowd Appeal mechanics are specific to tournament events.
- **Rationale**: Keeping Charisma's economic value in tournaments (where crowd appreciation matters narratively) is more thematic than a generic loot bonus. Standard post-combat reward formulas stay simpler.
- **Implications**: Tournaments spec defines how Crowd Appeal affects tournament rewards. Post-combat reward formulas don't reference Crowd Appeal.

### Recruitment: NPC Generation, Not Capture

- **Decision**: Recruitment generates new Named NPCs from the event's archetype pool. You do not recruit defeated Named NPCs or other players' characters. Flat base chance per event type, modified by team Charisma/Luck. Declined recruits become Free Agents. Not available in tournaments.
- **Rationale**: Generating from the archetype pool avoids the "steal enemy characters" problem (especially in tournaments). The archetype match is thematic — fight bandits, potentially recruit a bandit-type character. Declined recruits feeding the Free Agent pool creates a natural population churn.
- **Implications**: Event templates need archetype pool references. Free Agent pool gains a generation source. Roster-management spec handles Free Agent recruitment mechanics.

### Per-Event Post-Combat Configuration

- **Decision**: Post-combat phases are per-event-type configurable via an inherited defaults + overrides model. Each event category has a default post-combat profile; individual event templates override specific fields.
- **Rationale**: Different event types need different post-combat behavior (training = no injury + XP only; lethal PvE = everything; exhibition = no injury). Inherited defaults reduce boilerplate while allowing customization.
- **Implications**: Data model needs post-combat profile schema. Event category defaults must be defined. Individual event templates can override any profile field.

### Tournament Post-Combat: Full Per-Round, Immediate Effects

- **Decision**: Full post-combat fires after each tournament round. All effects (injury, discovery) apply immediately. Injuries persist across rounds (no game ticks between rounds). Accepted Perks are usable in subsequent rounds.
- **Rationale**: Immediate effects create dramatic tournament arcs — compounding injuries, clutch discoveries. Treating each round as a complete fight-to-aftermath cycle is consistent and simple. No game ticks between rounds means injuries are real consequences within the event.
- **Implications**: Tournament pacing must account for full post-combat between rounds. UI must handle per-round post-combat flow. Characters with death-prevention Perks gain tournament-specific value.

---

## Open Questions

Most structural questions are resolved. Remaining items are tuning values:

### Tuning Values
1. **Injury K constant**: What K in `overkill / (overkill + IR × K)` produces desired injury rates?
2. **Tier 2 severity weights**: What weight curves produce appropriate severity distributions at various overkill levels?
3. **Perk Discovery base rate**: What per-Trait base rate produces similar per-fight discovery rates to the old ~0.1% per-use model? Likely in the 1–5% range per qualifying Trait.
4. **XP pool per event type**: Total XP pool for each event type/difficulty — defined in [economy](economy.md).
5. **Recovery tick durations**: Exact values within the tier ranges (Minor 3–5, Major 10–15, Critical 20–30).
6. **Recruitment base chances**: Per-event-type recruitment chance (e.g., exhibition 15%, PvE 10%).
7. **Recruit Star Rating ranges**: Per-event-type/difficulty Star Rating ranges for generated recruits.
8. **Reward scaling percentages**: Exact placement-proportional percentages (1st=100%, 2nd=60%, 3rd=40%, 4th+=25% are starting points).
9. **XP participation bonus ratio**: What proportion of total XP is base share vs. participation bonus?
10. **Injury Resistance scaling multiplier**: What multiplier for the Injury Resistance derived stat produces desired injury rates at target attribute levels?

### Deferred to Other Specs
- **Gold amounts, loot tables, equipment drop rates**: [economy](economy.md) and [equipment](equipment.md)
- **Recruitment hiring costs**: [economy](economy.md)
- **Healing service costs and availability**: [groups](groups.md) and [economy](economy.md)
- **Crowd Appeal tournament effects**: [tournaments](tournaments.md)
- **Free Agent pool management**: [roster-management](roster-management.md)

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [combat](combat.md) | Provides Combat Scoreboard, Fallen list with overkill, active Traits list, Context Flags at combat end. Combat.md ends at victory/defeat — post-combat handles everything after. Overkill = final fall only (revived characters' previous falls don't accumulate). |
| [characters](characters.md) | **NEW derived stat: Injury Resistance** (50% Endurance + 30% Luck + 20% Willpower). Injury outcomes drive state transitions: Available (no injury), Recovering (minor/major/critical), Dead (death). Recovering characters fight with penalties. Recovery times in game ticks. |
| [traits-and-perks](traits-and-perks.md) | Perk Discovery model: per-Trait end-of-combat roll. Active Traits only. Formula: `(Base Rate + Luck Bonus + Perk Bonuses) × Trait Level Multiplier`. Resolution: accept/reject per discovery. **Death prevention Perks**: rare Perks that cap worst injury outcome at critical. **Injury Resistance Perks**: flat bonuses to the new derived stat. |
| [tournaments](tournaments.md) | Full post-combat fires per tournament round. Injuries apply immediately (persist across rounds, no recovery between rounds). Discoveries usable next round. No recruitment in tournaments. Crowd Appeal mechanics are tournament-specific. Must account for compounding injuries across rounds. |
| [economy](economy.md) | Post-combat is a primary source of gold, XP, and loot. XP pool per event type needed. Recruitment hiring costs. Healing service costs for injury recovery acceleration. Gold costs for vendor training fights. Placement-proportional reward scaling. |
| [roster-management](roster-management.md) | Injury outcomes feed into roster availability and recovery. Declined recruitment candidates become Free Agents — a primary Free Agent generation source. Recruitment phase produces new roster candidates at event-defined Star Ratings. |
| [equipment](equipment.md) | Loot phase distributes equipment drops. Drop tables and quality scaling defined jointly with equipment spec. Placement-proportional scaling applies to drop rates. |
| [groups](groups.md) | Healing services from Groups can accelerate or instant-cure injuries. Vendor training fights use per-event post-combat configuration (XP + discovery, no injury, gold cost). Event archetype pools reference Group-specific archetypes. |
| [meta-balance](../architecture/meta-balance.md) | Combat Scoreboard data (passed through from combat) feeds win/loss tracking per Trait. |
| [data-model](../architecture/data-model.md) | Must support: tiered injury roll data (diminishing returns formula with Injury Resistance), named injury type catalog (tier, effects, recovery time — content data), post-combat profile schema (per-event-type configuration with defaults + overrides), discovery roll results, recruitment candidate data (archetype, Star Rating range), per-team placement and reward scaling, Injury Resistance as derived stat. |

---

_Last updated: 2026-02-18 — 8 interrogation rounds, 18 decisions. Injury mechanics fully specified: diminishing returns formula, Injury Resistance derived stat (50% End + 30% Luck + 20% Will), tiered weighted random severity, hybrid named injury catalog, fixed game-tick recovery, Perk-based death immunity. XP: equal share + ticks-survived participation bonus. Recruitment: NPC generation from event archetype pool, flat chance per event type, declined → Free Agent, not in tournaments. Per-event post-combat configuration: inherited defaults + overrides. All teams get post-combat with placement-proportional reward scaling. Tournament: full post-combat per round, immediate effects. Crowd Appeal: tournaments only. Loot/gold: deferred to economy/equipment specs._
