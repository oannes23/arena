# Tournaments — Domain Specification

**Status**: 🟡 In progress
**Last interrogated**: —
**Last verified**: —
**Depends on**: [combat](combat.md), [post-combat](post-combat.md), [economy](economy.md)
**Depended on by**: None

---

## Overview

Tournaments are structured competitive events that give combat purpose and drive the game loop. They range from low-stakes Exhibition matches (for testing builds and grinding) to high-stakes Championship events (for prestige and major rewards). This domain also covers PvE encounters (on-demand grinding) and NPC backfill (procedurally filling empty tournament slots). Tournaments connect combat outcomes to the economy and roster management systems.

---

## Core Concepts

### Exhibition Tournaments

- Frequent (every tick or on-demand)
- No injury risk (or greatly reduced)
- Smaller rewards (gold, XP)
- Good for testing builds and grinding
- Low barrier to entry

### Championship Tournaments

- Less frequent (daily, weekly in multiplayer)
- Full injury risk
- Significant rewards (gold, loot, metacurrency, prestige)
- Elimination or Swiss-bracket format
- Entry requirements (minimum character star rating, roster size, etc.)

### Tournament Structure

- Multi-round elimination (lose = out)
- When the tournament fires, unfilled slots are **backfilled with NPC-generated teams**
- Different tournaments have different average participant levels and entry requirements
- Characters registered for a tournament are **committed until it resolves** (cannot be used elsewhere)

### PvE Encounters

On-demand combat available between ticks:
- Set difficulty tiers with scaling rewards
- Training/exhibition style — no injury risk (or reduced)
- Loot drops, XP, and gold rewards
- Predefined stages with themed opponents and loot tables
- Used to test builds before committing to tournaments

### NPC Backfill

When a tournament fires with empty slots:
- NPC teams are procedurally generated to fill remaining spots
- NPC quality scales with tournament tier/entry requirements
- NPC teams use the same character/trait/equipment/AI systems as player teams
- Ensures tournaments always have enough participants to run

### Scheduling

- **Single-player**: Player triggers tournaments on demand or they fire at tick boundaries
- **Multiplayer**: Tournaments scheduled on real-time intervals; players register before the deadline
- Championship tournaments may have registration periods (register during one tick window, tournament fires next tick)

---

## Decisions

### Exhibition vs. Championship Split

- **Decision**: Two distinct tournament tiers with different risk/reward profiles.
- **Rationale**: Players need a low-risk venue for testing builds and grinding without risking their best fighters. Championships provide the high-stakes drama that drives investment in roster building.
- **Implications**: Exhibition rewards must be low enough that Championships remain attractive. Injury risk differential is the primary motivator.

### Character Commitment During Tournaments

- **Decision**: Characters registered for a tournament are committed until it resolves.
- **Rationale**: Prevents gaming the system by pulling characters out after unfavorable matchups. Creates roster depth pressure — you need enough characters to cover tournament commitments plus ongoing operations.
- **Implications**: Players must plan roster allocation carefully. Need UI showing which characters are committed to what.

### NPC Backfill for All Tournaments

- **Decision**: Empty tournament slots are always filled with procedurally generated NPC teams.
- **Rationale**: Tournaments must fire on schedule regardless of player participation. In single-player, this is mandatory. In multiplayer, it ensures tournaments aren't cancelled due to low turnout.
- **Implications**: NPC team generation needs to produce reasonable, competitive teams scaled to tournament tier. NPC generation uses the same systems as player character generation.

### PvE as On-Demand Grinding

- **Decision**: PvE encounters are available between ticks with no scheduling requirement.
- **Rationale**: Players need a way to grind gold/XP/loot on their own schedule. PvE fills the gap between tournaments.
- **Implications**: PvE rewards must be tuned lower than tournament rewards to maintain tournament incentive. PvE encounter design needs themed stages with curated loot tables.

---

## Open Questions

1. Elimination format: single elimination, double elimination, or Swiss brackets? (Single = quicker but more variance; Swiss = more matches, better for skill differentiation.)
2. Injury risk timing: per-round or final-result only? (Per-round = more dangerous, creates "do I withdraw?" tension. Final-only = simpler.)
3. Can a character participate in multiple tournaments simultaneously?
4. What happens to injured characters mid-tournament? (Fight injured? Auto-withdraw? Player choice to forfeit?)
5. Exhibition vs. Championship tier definitions: how many tiers? What entry requirements per tier?
6. Should there be a "forfeit" option for players who want to withdraw mid-tournament?
7. Tournament reward scaling: how much more does a Championship pay vs. Exhibition? Linear with tier, or exponential?
8. PvE encounter variety: how many themed stages needed for MVP? Procedurally generated or hand-designed?
9. NPC team generation quality: should NPCs be intentionally slightly weaker than player teams (to reward optimization) or equal?
10. Registration period: how long before a tournament fires do players need to register?
11. Tournament history: is there a persistent record of tournament results? (Feeds into prestige/leaderboard systems.)

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [combat](combat.md) | Tournaments are sequences of combat instances. Match format, team size, and zone maps may vary by tournament tier. Attrition ramp onset/rate configured per event type via Context Flags. |
| [post-combat](post-combat.md) | Post-combat phases (injury/death checks, Perk discovery, recruitment, loot) apply per tournament round. Exhibition events skip injury checks. Non-exhibition events trigger tiered injury rolls with overkill severity. |
| [economy](economy.md) | Tournament rewards are a primary income source. Entry fees (if any) are a gold sink. |
| [roster-management](roster-management.md) | Character commitment during tournaments affects roster availability. Injury/death from Championship fights triggers the injury system. |
| [combat-ai](combat-ai.md) | NPC teams need AI configuration. AI quality may vary by tournament tier. |
| [characters](characters.md) | Entry requirements may gate by character star rating. NPC generation uses character generation rules. |

---

_Last updated: 2026-02-18 — Added post-combat.md dependency and implications (post-combat flow now in separate spec). Added combat.md attrition ramp reference. Previous: 2026-02-11._
