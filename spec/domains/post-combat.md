# Post-Combat — Domain Specification

**Status**: 🟡 In progress
**Last interrogated**: 2026-02-18
**Last verified**: —
**Depends on**: [combat](combat.md), [characters](characters.md), [traits-and-perks](traits-and-perks.md)
**Depended on by**: [tournaments](tournaments.md), [roster-management](roster-management.md), [economy](economy.md)

---

## Overview

Post-combat is the resolution phase after a fight concludes. When combat declares victory or defeat (last team standing), the post-combat system takes over to resolve consequences: injuries, Perk discovery, recruitment opportunities, and loot distribution. This phase is presented as a sequential dramatic narrative, not a bulk summary.

Post-combat receives data from the combat system: the [Combat Scoreboard](combat.md#combat-scoreboard), Fallen characters with overkill values, active Traits list, and [Combat Context Flags](combat.md#combat-context-flags).

---

## Core Concepts

### Phased Presentation Flow

Post-combat resolves as a **phased dramatic presentation**:

1. **Combat Resolution**: Final blow, victory/defeat declared. Summary statistics from the Combat Scoreboard.
2. **Injury & Death Phase**: Fallen characters' fates revealed one by one (injury rolls, severity outcomes).
3. **Perk Discovery Phase**: Discovery results presented per qualifying Trait (accept/reject per discovery).
4. **Recruitment Phase**: If applicable, defeated Named NPCs or generated NPCs offered for recruitment.
5. **Loot Phase**: Equipment, gold, and other rewards distributed.

Each phase is presented sequentially for dramatic effect — the player experiences the aftermath as a narrative sequence, not a bulk summary. Phases with no content (e.g., no Fallen characters → skip injury phase) are skipped silently.

### Injury Mechanics

After combat resolves, Fallen characters in **non-exhibition** events undergo a tiered injury check:

**Tier 1: Injury Roll**
- Roll to determine if an injury occurs at all
- Factors: overkill magnitude, character Endurance, Luck, Perks that improve injury resistance
- **No injury**: Character returns to Available state
- **Injury**: Proceed to Tier 2

**Tier 2: Severity Roll**
- Roll to determine injury severity
- Factors: same as Tier 1, plus overkill magnitude further increases severity chance
- **Outcomes**:
  - **Minor injury**: Temporary stat penalty, short recovery time → Recovering state
  - **Major injury**: Significant stat penalty, long recovery time → Recovering state
  - **Critical injury**: Permanent Potential reduction or anatomical slot loss → Recovering state
  - **Death**: Character enters Dead state

**Exhibition events** skip injury checks entirely — all Fallen characters return to Available.

**Overkill severity**: The amount of excess damage beyond 0 HP (tracked by the combat system) directly influences both Tier 1 and Tier 2 rolls. A character barely knocked out has much better injury odds than one obliterated by massive overkill.

### Perk Discovery

Perk Discovery uses a **single end-of-combat check per qualifying Trait**, not per-Action/Trigger-use during combat.

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

### Recruitment Phase

*Placeholder — details from [characters](characters.md) and [roster-management](roster-management.md).*

When applicable (certain event types, defeated Named NPCs), the recruitment phase offers the player a chance to recruit new characters:

- **Named NPC recruitment**: Defeated Named NPCs may be offered for recruitment based on a Charisma/Luck/Awareness check
- **Generated NPC recruitment**: Some events generate potential recruits from the defeated side
- Recruitment decisions are made per-character (accept/decline)
- Recruitment costs and mechanics are defined in [economy](economy.md) and [roster-management](roster-management.md)

### Loot Phase

*Placeholder — details from [equipment](equipment.md) and [economy](economy.md).*

The final phase distributes rewards from the combat:

- **Gold**: Base gold reward modified by event type, difficulty, and character Crowd Appeal
- **XP**: Per-character XP earned (based on participation, kills, ticks survived)
- **Equipment drops**: Loot table rolls based on event type and difficulty
- **Other rewards**: Metacurrency (from tournaments), consumables, materials
- Distribution formulas are defined in [economy](economy.md) and [equipment](equipment.md)

---

## Decisions

### Phased Post-Combat Presentation

- **Decision**: Post-combat flow is a sequential dramatic presentation: combat resolution → injury/death → Perk discovery → recruitment → loot.
- **Rationale**: Sequential presentation creates narrative moments and gives each phase appropriate attention. Bulk summaries feel rushed and don't generate player attachment.
- **Implications**: UI must support phased presentation. Each phase can be expanded with detail as the game matures.

### Tiered Injury Checks

- **Decision**: Post-combat injury uses two rolls: first determines if injury occurs, second determines severity. Both are modified by overkill, Endurance, Luck, and Perks.
- **Rationale**: Two-tier system creates more nuanced outcomes than a single roll. Most Falls result in no injury (minor events). When injuries do occur, the severity spectrum (minor → major → critical → death) provides dramatic range.
- **Implications**: Characters built for durability (high Endurance, Luck, injury-resistance Perks) have a meta-advantage beyond just surviving combat.

### Overkill Affects Injury Severity

- **Decision**: Excess damage beyond 0 HP (overkill) is tracked by combat and increases injury severity in the post-combat injury roll.
- **Rationale**: Creates meaningful stakes for getting hit hard. A character barely knocked out vs. one obliterated have different injury expectations. Encourages protecting characters from massive single hits.
- **Implications**: Equipment and Perks that reduce spike damage (shields, damage caps) have indirect injury-prevention value. Tournament strategy must consider opponent's burst damage potential.

### Perk Discovery — Per-Trait End-of-Combat Roll

- **Decision**: Perk Discovery uses a single end-of-combat check per qualifying Trait, not per-Action/Trigger-use during combat. One roll per Trait. Only Traits that had at least one Action used or Trigger fire qualify ("active Traits").
- **Rationale**: Per-use rolls during combat were implementation-complex and created a bias toward high-Action-count builds. A single per-Trait roll is simpler, fairer, and creates a cleaner handoff to the post-combat phase.
- **Implications**: Discovery probability per Trait per fight is higher than the old per-use base rate (retuned to produce similar per-fight discovery rates). The "active Traits only" rule replaces the old "passive Stat Adjustments don't trigger discovery" rule — same intent, cleaner implementation.
- **Alternatives considered**: Per-Action/Trigger-use during combat (original model — rejected for complexity and Action-count bias), per-character single roll (rejected — would not reward diverse Trait investment).

### Post-Combat as Separate Spec

- **Decision**: Post-combat resolution lives in its own spec, separate from combat.md.
- **Rationale**: Post-combat is a distinct gameplay phase with its own mechanics, UI flow, and dependencies (economy, roster-management, groups). Separating it reduces combat.md's scope and allows each phase to evolve independently.
- **Implications**: Combat.md ends at victory/defeat and hands off structured data. This spec owns injury checks, discovery resolution, recruitment, and loot.

---

## Open Questions

1. **Injury roll probability tables**: Exact thresholds for Tier 1 (injury yes/no) and Tier 2 (severity) rolls. How do overkill magnitude, Endurance, Luck, and Perks combine?
2. **Injury outcome tables**: What are the specific minor/major/critical injury effects? Stat penalty magnitudes, recovery durations, permanent reduction amounts.
3. **Perk Discovery base rate**: What per-Trait base rate produces similar per-fight discovery rates to the old ~0.1% per-use model? Likely in the 1–5% range per qualifying Trait.
4. **Recruitment formulas**: Charisma/Luck/Awareness check thresholds for Named NPC recruitment. Event type modifiers.
5. **Loot distribution formulas**: Gold/XP/equipment drop rates per event type and difficulty tier.
6. **XP distribution model**: How is per-character XP calculated from Combat Scoreboard data (ticks survived, damage dealt, kills)?
7. **Crowd Appeal loot bonus**: Does Crowd Appeal affect loot quality/quantity in post-combat?

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [combat](combat.md) | Provides Combat Scoreboard, Fallen list with overkill, active Traits list, Context Flags at combat end. Combat.md ends at victory/defeat — post-combat handles everything after. |
| [characters](characters.md) | Injury outcomes drive state transitions: Available (no injury), Recovering (minor/major/critical), Dead (death). Overkill tracking from combat affects injury severity. |
| [traits-and-perks](traits-and-perks.md) | Perk Discovery model changed to per-Trait end-of-combat roll. Active Traits only. Formula unchanged: `(Base Rate + Luck Bonus + Perk Bonuses) × Trait Level Multiplier`. Resolution: accept/reject per discovery. |
| [tournaments](tournaments.md) | Post-combat phases apply per tournament round. Exhibition events skip injury checks. Context Flags determine event type behavior. |
| [economy](economy.md) | Post-combat is a primary source of gold, XP, and loot. Distribution formulas affect economic balance. Recruitment costs interact with economy. |
| [roster-management](roster-management.md) | Injury outcomes feed into roster availability. Recruitment phase produces new roster candidates. |
| [equipment](equipment.md) | Loot phase distributes equipment drops. Drop tables and quality scaling defined jointly with equipment spec. |
| [groups](groups.md) | Recruitment may involve Group-affiliated Named NPCs. Bond Trait implications for recruited characters. |
| [meta-balance](../architecture/meta-balance.md) | Combat Scoreboard data (passed through from combat) feeds win/loss tracking per Trait. |
| [data-model](../architecture/data-model.md) | Must support: tiered injury roll data (two-tier check with overkill factor), post-combat phase state machine, discovery roll results, recruitment candidate data. |

---

_Last updated: 2026-02-18 — Initial creation from combat.md round 8 interrogation. Injury mechanics, Perk Discovery (per-Trait model), and phased presentation flow moved from combat.md. Recruitment and loot phases are placeholders pending deeper interrogation._
