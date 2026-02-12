# Characters — Domain Specification

**Status**: 🟢 Complete
**Last interrogated**: 2026-02-11
**Last verified**: —
**Depends on**: None (primitive)
**Depended on by**: [traits-and-perks](traits-and-perks.md), [combat](combat.md), [equipment](equipment.md), [economy](economy.md), [groups](groups.md), [roster-management](roster-management.md), [tournaments](tournaments.md)

---

## Overview

Characters are the fundamental game entity — fighters and support staff in the player's gladiatorial household. This domain covers character identity (Attributes, Star Rating), physical form (Anatomical Slots), generation rules, progression tracking, and resource pools. Traits and Perks are a separate domain; this domain defines the "chassis" that Traits attach to.

---

## Core Concepts

### Star Rating as Slot Capacity

A character's Star Rating (1–5★) determines how many Trait Slots they have in each of three categories (Core, Role, Bond). A 3★ character has 3 Core slots, 3 Role slots, and 3 Bond slots. Each slot holds exactly one Trait of any star level — no budget math. A 1★ character can hold a 5★ Trait; they just only have one slot per category.

| Character Stars | Core Slots | Role Slots | Bond Slots |
|-----------------|-----------|-----------|-----------|
| 1★ | 1 | 1 | 1 |
| 2★ | 2 | 2 | 2 |
| 3★ | 3 | 3 | 3 |
| 4★ | 4 | 4 | 4 |
| 5★ | 5 | 5 | 5 |

Stars also serve as narrative depth markers: 1★ = goons/minions with minimal identity, 5★ = main character energy with rich personalities and backstories.

Note: 0★ exists for Equipment only, not Characters. The minimum Character Star Rating is 1★.

### Star Rating Attribute Scaling

Stars modestly boost attribute averages AND widen variance. Stars are primarily Trait capacity; the stat boost is a secondary bonus.

| Star Rating | Avg Current | Avg Potential |
|-------------|------------|---------------|
| 1★ | ~35 | ~75 |
| 2★ | ~42 | ~85 |
| 3★ | ~50 | ~95 |
| 4★ | ~57 | ~105 |
| 5★ | ~65 | ~115 |

A lucky 3★ can out-stat an unlucky 5★ — variance is wide enough that stars are not deterministic for attributes.

### Attributes: Current and Potential

Every character has **Current** and **Potential** values for each of 9 primary Attributes on a **0–200 scale**:

- **Current**: Actual stat value used in combat and checks. Average ~50 for a starting character (varies by star rating). Range 0–200.
- **Potential**: Maximum trainable cap. Average ~100 for a starting character (varies by star rating). Range 0–200.
- **Inverse correlation**: Loose inverse correlation — lower Current often correlates with higher Potential (growth opportunity vs. ready-made veteran). Unicorn characters (high both) can appear rarely.
- **Training**: Spend XP to increase Current toward Potential cap. Cost per +1 = current value of the attribute (e.g., raising from 50 to 51 costs 50 XP).
- **Potential mutability**: Potential is mostly fixed at generation, but can change:
  - **Promotion**: +10% to all Potentials (percentage-based).
  - **Rare events**: Legendary items, special achievements, or expensive vendor services can provide small Potential increases.
  - **Injuries/death**: Can permanently reduce Potential.

### Primary Attributes

| Attribute | Governs |
|-----------|---------|
| **Might** | Melee damage, equipment requirements (heavy), forced movement, physical intimidation |
| **Speed** | Initiative gain (primary), dodge/evasion, movement-related perks, Action Speed modifier |
| **Accuracy** | Attack rolls (hit chance), ranged damage scaling, targeting quality |
| **Endurance** | Health pool (primary), Stamina pool (co-primary), physical damage soak, attrition resistance |
| **Charisma** | Morale (own + allies), vendor pricing, crowd management buffs/debuffs, crowd appeal, leadership aura |
| **Awareness** | Detection (anti-stealth), trap awareness, Initiative gain (minor), tie-breaking, critical hit chance |
| **Intellect** | Spell power, crafting quality, XP efficiency, Mana pool, Initiative gain (tiny) |
| **Willpower** | Status resistance, mental damage soak, Focus pool, Stamina pool (co-primary), concentration/anti-interrupt |
| **Luck** | Critical hit chance (with Awareness, Accuracy), all resistance rolls, loot table bonuses, crowd favor decisions |

### Multi-Attribute Blending

Derived stats are computed from weighted combinations of multiple primary attributes rather than a single source. Example blends:

- **Initiative Rate**: mostly Speed + some Awareness + small Intellect
- **Health Pool**: mostly Endurance + some Willpower + small Might
- **Stamina Pool**: Willpower + Endurance equally

This prevents dump stats and makes diverse builds viable — every attribute contributes to multiple derived stats, so no attribute is ever truly worthless.

### Derived Stats

Derived stats are computed from primary attributes via multi-attribute blends. Weight ratios are locked:

**Combat — 15 Derived Stats:**

| Derived Stat | Weight Formula |
|---|---|
| Health Pool | 60% Endurance + 25% Willpower + 15% Might |
| Stamina Pool | 50% Willpower + 50% Endurance |
| Initiative Rate | 60% Speed + 25% Awareness + 15% Intellect |
| Melee Hit | 60% Accuracy + 25% Might + 15% Awareness |
| Ranged Hit | 60% Accuracy + 25% Awareness + 15% Luck |
| Magic Hit | 60% Intellect + 25% Awareness + 15% Willpower |
| Melee Damage | 60% Might + 25% Accuracy + 15% Speed |
| Ranged Damage | 60% Accuracy + 25% Might + 15% Awareness |
| Magic Damage | 60% Intellect + 25% Willpower + 15% Awareness |
| Defense Value | 50% Speed + 35% Awareness + 15% Luck |
| Magic Defense | 60% Willpower + 25% Awareness + 15% Luck |
| Soak Value | 70% Endurance + 20% Willpower + 10% Might |
| Physical Crit | 40% Awareness + 35% Luck + 25% Accuracy |
| Magic Crit | 40% Awareness + 35% Luck + 25% Intellect |
| Crowd Appeal | 60% Charisma + 25% Luck + 15% Awareness |

**Utility — Single-Source Stats:**

| Derived Stat | Source |
|---|---|
| Vendor Modifier | Charisma |
| Training Speed | Intellect |

### Resources

This spec defines universal resources only — pools every character has:

- **Health**: Damage capacity. Reduced by taking damage. Character dies at 0.
- **Stamina**: Physical exertion pool. Spent by physical Actions.

**Stamina Mechanics:**
- **Exhaustion**: When Stamina reaches 0, physical actions cost Health instead (last-stand mechanic).
- **In-combat regen**: Slow passive regeneration per combat tick. The Defend action provides a larger Stamina recovery burst.
- **Between-fight reset**: Configurable per event type:
  - Exhibition tournaments: full Health/Stamina reset between fights.
  - Championship tournaments: partial reset (percentages deferred to tournaments spec).
  - PvE encounters: full reset between fights.
- Stamina regen rate and Defend recovery amount are tuning values deferred to the combat spec.

Trait-unlocked resources (Mana, Focus, Rage, Faith, etc.) are defined in the [traits-and-perks](traits-and-perks.md) spec. Those resources are granted by specific Traits and only exist on characters who possess those Traits.

### Anatomical Slots

Physical body locations where equipment can be worn. Defined per-character based on Core Traits (ancestry).

| Slot | Default Count | Notes |
|------|--------------|-------|
| Hand | 2 | Main hand, off-hand |
| Head | 1 | |
| Torso | 1 | |
| Arms | 1 | |
| Legs | 1 | |
| Feet | 1 | |
| Finger | 2 | |
| Neck | 1 | |
| Back | 1 | |
| Belt | 1 | |

**Biological variations** (determined by Core Traits / ancestry):
- Missing limbs (from injury or racial anatomy)
- Non-humanoid anatomy (quadrupeds, serpentine bodies)
- Extra limbs (four-armed species = 4 hand slots)
- No head (oozes, constructs)

---

## Decisions

### Attribute Count and Selection

- **Decision**: Exactly 9 primary Attributes: Might, Speed, Accuracy, Endurance, Charisma, Awareness, Intellect, Willpower, Luck.
- **Rationale**: Nine attributes provide rich build differentiation. Each maps to distinct play archetypes while multi-attribute blending ensures no dump stats. Expanded from original 4 (Strength, Agility, Willpower, Awareness) to support crowd mechanics, economic interaction, crafting, and luck-based gameplay.
- **Implications**: All derived stats must use multi-attribute blends from these nine. Traits/Perks reference 9 attributes for Stat Adjustments. Equipment requirements reference 9 attributes.

### Multi-Attribute Blending

- **Decision**: Derived stats are computed from weighted combinations of multiple primary attributes, not single-source. Weight ratios are locked (see Derived Stats table above).
- **Rationale**: Prevents dump stats. Makes diverse builds viable. A "pure Might" build still benefits from Endurance (Health), Awareness (crit), etc.
- **Implications**: Every attribute contributes to multiple derived stats. The 15-stat formula table provides locked ratios for implementation.

### Current/Potential Dual-Track

- **Decision**: Every Attribute has both a Current and a Potential value on a 0–200 scale. Training spends XP to raise Current toward Potential.
- **Rationale**: Creates interesting hiring decisions — a cheap rookie with high Potential may outperform an expensive veteran long-term. Supports "growth opportunity" vs. "ready now" roster strategies.
- **Implications**: Character generation must produce both values. Averages (~50 Current, ~100 Potential) are midpoints, not caps — the 0–200 range allows for extreme characters.

### Current/Potential Distribution

- **Decision**: Loose inverse correlation between Current and Potential. Unicorn characters (high both) can appear rarely.
- **Rationale**: Most characters have clear trade-offs (ready now vs. growth potential). Finding a unicorn is exciting — not impossible, just uncommon. Supports both evaluation skill and exciting discovery moments.
- **Implications**: Generation model must support variable correlation strength. Unicorn frequency is a tuning knob.

### Star Rating = Slot Capacity (No Budget Math)

- **Decision**: Star Rating (1–5★) determines the count of Trait Slots per category. Each slot holds one Trait of any star level. No "budget points" to allocate. 0★ is for equipment only, not characters.
- **Rationale**: Simple and legible. Players immediately understand capacity. Avoids fiddly min-maxing of slot budgets. Stars double as narrative depth — 1★ goons vs. 5★ heroes.
- **Implications**: Character Star Rating is the single most important power axis — a 5★ character has 5× the Trait capacity of a 1★. Character star generation and acquisition cost must reflect this.

### Star Rating Generation

- **Decision**: Star Rating at generation is determined by three factors:
  1. **Group tier pools**: Different Groups have different star probability tables.
  2. **Gold investment**: Recruiter Group where more gold = better quality candidates.
  3. **Metacurrency**: Spend metacurrency for major probability boosts on star rating.
- **Rationale**: Multiple levers give players agency over recruit quality. Gold investment creates economic tension. Metacurrency provides long-term progression toward better recruits.
- **Implications**: Economy spec must define gold-to-quality curves and metacurrency star boost costs. Groups spec defines which Groups offer recruitment.

### Rare Promotion

- **Decision**: Characters can gain +1 Star Rating via an extremely expensive metacurrency-only promotion process.
- **Cost**: Metacurrency only (very expensive). No other gates. Exact cost deferred to economy spec.
- **Bonus**: +10% to all Potentials (percentage-based). Also gains +1 Trait slot per category (from the star increase).
- **Rationale**: Rewards deep investment in a specific character. Creates attachment and narrative moments. Heavy metacurrency cost prevents trivial power inflation while keeping the gate simple.
- **Implications**: Economy spec must define metacurrency promotion cost. Roster management spec must track promotion eligibility and apply Potential increases.

### Character Generation Model

- **Decision**: Group-specific invisible archetype templates + high noise for base attribute Current/Potential.
- **Archetypes are per-Group**: Each Group that offers recruitment defines its own themed archetypes. There is NO universal archetype list.
- **Archetype definition**: Each archetype specifies:
  - Attribute bumps and dumps (bonuses/penalties to specific attributes)
  - Likely trait pool (weighted trait selection)
  - Common equipment themes
- **Examples**:
  - *Necromancer's Guild*: Death Knight (high Might/Endurance/Willpower, low Charisma/Speed), Necromancer (high Intellect×2/Willpower, low Might/Charisma)
  - *Arena Recruitment Center*: Pit Fighter, Arena Brawler, Crowd Pleaser
  - *Tavern*: Wanderer, Sellsword, Drifter
- **Variable count per source**: Major Groups offer 4–6 archetypes; minor Groups offer 1–2.
- **High noise**: ±20–25 per attribute on both Current and Potential.
- **Real dump stats**: Archetype penalties weighted toward 0 but up to −20. Sub-50 Potential possible for 1–2 attributes on min-max archetypes.
- **Invisible to players**: Players see resulting stats and traits, not the archetype label. Character evaluation is a player skill (pattern recognition).
- **Rationale**: Group-specific archetypes create meaningful recruitment source differentiation. Players learn which Groups produce which kinds of fighters. High noise ensures no two characters are identical even from the same archetype.
- **Implications**: Each Group that offers recruitment must define its archetype set (see [groups.md](groups.md)). Traits-and-perks spec must support per-Group likely trait pools.

### Anatomical Slots Defined by Core Traits

- **Decision**: A character's available anatomical slots are determined by their Core Traits (ancestry). Default is humanoid (the table above). A character with 0 Core Traits gets blank-slate humanoid anatomy. Species are defined entirely via Core Traits — no hardcoded species list.
- **Rationale**: Enables non-humanoid character builds as emergent gameplay. "Orcish Blood," "Fey-Touched" etc. are just Core Traits with stat modifiers + anatomical slot changes. Blank slate default means Core Traits are always optional.
- **Implications**: Equipment system must handle variable slot availability. Core Trait definitions must specify any slot modifications.

### XP Total as Implicit Level

- **Decision**: Track total XP invested in a character. Not a visible "level" — no level number displayed.
- **Rationale**: Avoids the "I'm level 7" identity that makes players reluctant to retire or replace characters. XP total is a behind-the-scenes metric for promotion eligibility, upkeep cost scaling, and retirement value.
- **Implications**: UI never shows a level number. Backend tracks cumulative XP for gating and calculations.

### Career Milestones

- **Decision**: Track fights/tournaments participated in. Characters can optionally "retire" for metacurrency bonuses.
- **Rationale**: Retirement is not forced — it's an opt-in reward for letting go of a developed character. Creates interesting tension between keeping a strong fighter vs. cashing them out. Career milestone tracking provides retirement value scaling.
- **Implications**: Roster management spec must implement retirement flow. Economy spec must define metacurrency rewards for retirement.

### Character Identity

- **Decision**: Progressive implementation across project phases.
  - **Phase 1**: Procedural name + short physical description (cosmetic only).
  - **Phase 3–4**: Personality traits that affect combat AI behavior.
  - **Phase 4+**: Rich narrative hooks (backstory fragments, relationships).
- **Rationale**: Identity depth scales with star rating — 1★ goons need minimal personality, 5★ characters deserve rich backstories. Phased approach avoids over-investing in narrative systems before combat is fun.
- **Implications**: Combat AI spec must eventually support personality-driven behavior modifications.

### Resources

- **Decision**: Characters spec defines universal resources only: Health, Stamina. Trait-unlocked resources (Mana, Focus, Rage, etc.) are defined in the traits-and-perks spec.
- **Rationale**: Clean separation — the character chassis has Health and Stamina; everything else comes from build choices.
- **Implications**: Traits-and-perks spec must define all non-universal resource pools and the Traits that grant them.

### Attribute Numeric Range

- **Decision**: All attributes use a 0–200 scale for both Current and Potential.
- **Rationale**: Wide enough for granular differentiation between characters. Averages (~50 Current, ~100 Potential) sit at midpoints, leaving room for extreme characters in both directions.
- **Implications**: All formulas, thresholds, and UI displays must accommodate the 0–200 range.

### Bonus Modifier System

- **Decision**: Trait/Perk/Equipment bonuses are a tracked layer separate from base Current. Removing a source removes only its bonus; base Current is never affected.
- **Stacking**: Additive within each source type (Perks, Equipment, Status Effects). All source types combine additively with each other.
- **Categories**: Bonuses are categorized by source type for potential future per-type caps.
- **Rationale**: Clean separation between trained stats and bonus stats prevents confusing stat drops when equipment changes. Additive stacking is simple and predictable. Source-type categorization future-proofs balance tuning.
- **Implications**: Character stat display should show base + bonus breakdown. Equipment/Perk removal is safe and predictable.

### Training System

- **Decision**: XP cost per +1 Current = current value of the attribute being trained (e.g., raising from 50→51 costs 50 XP). Can train any attribute where Current < Potential. No per-tick throttle — purely gated by XP availability.
- **Rationale**: Escalating cost creates natural diminishing returns. High-stat attributes are expensive to improve further. No throttle keeps training simple — economic pressure (XP scarcity) is the gate.
- **Implications**: Economy spec must define XP sources and rates. Training Speed (derived from Intellect) may modify XP costs or provide bonus XP.

### Stamina Mechanics

- **Decision**: When Stamina reaches 0, physical actions cost Health instead (exhaustion/last-stand mechanic). Slow passive regen per combat tick; Defend action gives a larger Stamina recovery burst. Reset between fights is configurable per event type.
- **Rationale**: Exhaustion creates dramatic late-fight moments without completely disabling characters. Defend as Stamina recovery adds tactical depth to a basic action. Per-event reset creates different strategic demands across event types.
- **Implications**: Combat spec must implement Health-drain-on-exhaustion, regen rates, and Defend recovery. Tournaments spec must define partial reset percentages for championship events.

---

## Deferred Tuning Items

All character-domain design questions are resolved. The following tuning values are deferred to other specs:

- **Stamina regen rate and Defend recovery amount** → deferred to [combat](combat.md) spec
- **Championship partial reset percentages** → deferred to [tournaments](tournaments.md) spec
- **Promotion metacurrency cost** → deferred to [economy](economy.md) spec

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [traits-and-perks](traits-and-perks.md) | Now 9 attributes for Stat Adjustments to reference. Species = Core Traits pattern (no hardcoded species list). Trait-unlocked resources (Mana, Focus, etc.) defined there. Group archetype system defines likely trait pools per recruitment source. Bond Traits map to Groups. |
| [combat](combat.md) | 15 derived stat formulas with locked weight ratios. Stamina exhaustion mechanic (0 Stamina → Health drain). Stamina regen per tick + Defend recovery burst. Magic Defense stat. Luck affects crit chance and resistance rolls. |
| [equipment](equipment.md) | Equipment requirements reference 9 attributes (especially Might for heavy gear). Bonus Modifier system: equipment bonuses are a tracked layer separate from base Current. |
| [economy](economy.md) | Star generation involves Group tiers + gold + metacurrency. Promotion = metacurrency only (very expensive). Training cost = Current value per +1. Career milestones affect retirement value. |
| [groups](groups.md) | Group-specific recruitment archetypes. Each Group that offers recruitment defines its archetype set. Vendor pricing uses Charisma-based Vendor Modifier. |
| [roster-management](roster-management.md) | Promotion gives +1★ and +10% all Potentials. Potential reduction from injuries. Career milestone tracking. Retirement for metacurrency. |
| [tournaments](tournaments.md) | Configurable Health/Stamina reset per event type (Exhibition: full, Championship: partial, PvE: full). Crowd/momentum section needed (Charisma-driven). |
| [combat-ai](combat-ai.md) | Personality traits (Phase 3–4) modify AI behavior. |

---

_Last updated: 2026-02-11_
