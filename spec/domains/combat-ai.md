# Combat AI — Domain Specification

**Status**: 🟡 In progress
**Last interrogated**: —
**Last verified**: —
**Depends on**: [combat](combat.md)
**Depended on by**: [tournaments](tournaments.md)

---

## Overview

Combat AI is the player-facing build tool for configuring how characters behave during automated combat. While the combat domain defines what characters *can* do, Combat AI defines the rules for *what they choose to do* on each turn. This is separate from combat because AI configuration is a player-facing UX concern with its own complexity — it's a build optimization surface, not part of the resolution engine.

---

## Core Concepts

### AI as Build Tool

AI configuration is a core part of character optimization. A well-built character with bad AI orders will underperform. AI config is the "strategy" layer that sits on top of the "build" layer (Traits, Perks, Equipment).

### Configuration Axes

**Target Priority**: Who to attack.
- Closest enemy
- Lowest HP enemy
- Highest threat enemy
- Specific role targeting (e.g., "target healers first")

**Range Preference**: Where to position.
- Prefer Short range (melee fighters)
- Prefer Medium range (polearm users, mid-range casters)
- Prefer Long range (archers, long-range casters)

**Action Priority Lists**: What to do, context-dependent.
- Ordered preference list per range bracket
- Example: At Short range — Flame Aura → Attack → Defend
- Example: At Long range — Fireball → Move closer → Defend
- Falls through the list until a valid action is found

**Consumable Triggers**: When to use consumables.
- Use healing potion below X% HP
- Use buff potion on first turn
- Use bomb when 3+ enemies in zone
- Custom trigger conditions

**Personality Modifiers**: Character Traits that override or modify AI behavior.
- A Berserker might ignore defensive orders when low HP
- A Coward might prioritize fleeing over attacking
- Personality modifiers come from Core Traits, not player configuration

### Phased Implementation

| Phase | AI Capability |
|-------|--------------|
| Phase 1 (MVP 0) | Basic AI: attack nearest, move toward nearest, defend when low HP. No player configuration. |
| Phase 2 (MVP 1) | Basic config: target priority + action priority lists. Enough to influence fights meaningfully. |
| Phase 4 | Full customization: range preferences, action priorities per range bracket, consumable triggers, personality system. |

---

## Decisions

### AI Configuration is Player-Facing

- **Decision**: Players explicitly configure AI behavior before combat rather than the game choosing "smart" AI automatically.
- **Rationale**: AI configuration is part of the build optimization loop. Players should feel ownership over their characters' tactical choices.
- **Implications**: UI needs clear AI configuration screens. Default AI behaviors must be reasonable so unconfigured characters aren't useless.

### Priority List Model (Not Decision Tree)

- **Decision**: AI uses ordered priority lists (try A, then B, then C) rather than complex decision trees or scripting.
- **Rationale**: Legible, predictable, and debuggable. Players can understand why their character made a specific choice. No "the AI did something stupid" frustration from opaque decision-making.
- **Implications**: Priority lists must cover enough granularity to express diverse strategies without becoming overwhelming.

### Personality Overrides from Traits

- **Decision**: Core Traits can impose personality modifiers that override player-set AI in specific situations.
- **Rationale**: Creates character flavor — a Berserker acts berserk. Creates tension: do you want the powerful Berserker Trait knowing it might override your careful defensive strategy?
- **Implications**: Players must be able to see what personality modifiers a Trait imposes before acquiring it. Override conditions must be specific and predictable.

---

## Open Questions

1. Specific AI consumable trigger configuration syntax — how granular? (HP%, turn number, enemy count in zone, stack count of status X?)
2. How does AI handle multi-target abilities? (Priority list for single-target is clear; what about "which zone to AoE?")
3. Personality modifier implementation: hard override (ignores player config) or soft bias (modifies priority weights)?
4. Can AI be configured per-match (before each fight) or only per-character (persistent config)?
5. AI behavior when no valid action in priority list: default to Attack? Defend? Skip turn?
6. How does AI decide movement when not at preferred range? (Move toward nearest enemy? Move toward a specific zone?)
7. NPC team AI: same system as player AI, or separate (potentially smarter/dumber) logic?
8. Can players preview/simulate AI behavior before a match? ("Dry run" mode?)

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [combat](combat.md) | AI must understand all available actions, their costs, cooldowns, and valid targets to make priority decisions. |
| [traits-and-perks](traits-and-perks.md) | Personality modifiers come from Core Traits. Action priority lists reference Perk-granted Actions by name. |
| [consumables](consumables.md) | AI needs trigger conditions for consumable usage. Consumable slot contents affect AI decision-making. |
| [tournaments](tournaments.md) | AI configuration quality directly affects tournament performance. NPC teams need AI generation. |

---

_Last updated: 2026-02-11_
