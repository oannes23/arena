# Characters — Domain Specification

**Status**: 🟡 In progress
**Last interrogated**: —
**Last verified**: —
**Depends on**: None (primitive)
**Depended on by**: [traits-and-perks](traits-and-perks.md), [combat](combat.md), [equipment](equipment.md), [economy](economy.md), [roster-management](roster-management.md)

---

## Overview

Characters are the fundamental game entity — fighters and support staff in the player's gladiatorial household. This domain covers character identity (Attributes, Star Rating), physical form (Anatomical Slots), and generation rules. Traits and Perks are a separate domain; this domain defines the "chassis" that Traits attach to.

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

### Attributes: Current and Potential

Every character has **Current** and **Potential** values for each of 4 primary Attributes:

- **Current**: Actual stat value used in combat and checks. Average ~50 for a starting character.
- **Potential**: Maximum trainable cap. Average ~100 for a starting character.
- **Inverse correlation**: Lower Current often correlates with higher Potential (growth opportunity vs. ready-made veteran).
- **Training**: Spend XP to increase Current toward Potential cap.

### Primary Attributes

| Attribute | Governs |
|-----------|---------|
| **Strength** | Physical power, melee damage, carry capacity, heavy equipment requirements |
| **Agility** | Speed, dodge, accuracy, light/finesse weapon scaling |
| **Willpower** | Mental resistance, spell power, resource pools, status resistance |
| **Awareness** | Detection (stealth counterplay), trap awareness, initiative tie-breaking, targeting intelligence |

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

- **Decision**: Exactly 4 primary Attributes: Strength, Agility, Willpower, Awareness.
- **Rationale**: Four Attributes is enough for meaningful build differentiation without creating an overwhelming stat allocation problem. Each maps cleanly to a play archetype.
- **Implications**: All derived stats (Speed, Defense, Attack, Soak, etc.) must derive from these four. No hidden stats beyond these primaries and their derivatives.

### Current/Potential Dual-Track

- **Decision**: Every Attribute has both a Current and a Potential value. Training spends XP to raise Current toward Potential.
- **Rationale**: Creates interesting hiring decisions — a cheap rookie with high Potential may outperform an expensive veteran long-term. Supports "growth opportunity" vs. "ready now" roster strategies.
- **Implications**: Character generation must produce both values. Training system needs an XP cost curve for Current increases.

### Star Rating = Slot Capacity (No Budget Math)

- **Decision**: Star Rating determines the count of Trait Slots per category. Each slot holds one Trait of any star level. No "budget points" to allocate.
- **Rationale**: Simple and legible. Players immediately understand capacity. Avoids fiddly min-maxing of slot budgets.
- **Implications**: Character Star Rating is the single most important power axis — a 5★ character has 5× the Trait capacity of a 1★. Character star generation and acquisition cost must reflect this.

### Anatomical Slots Defined by Core Traits

- **Decision**: A character's available anatomical slots are determined by their Core Traits (ancestry). Default is humanoid (the table above); ancestry Traits modify this.
- **Rationale**: Enables non-humanoid character builds (four arms, no head, etc.) as emergent gameplay rather than hardcoded exceptions.
- **Implications**: Equipment system must handle variable slot availability. Core Trait definitions must specify any slot modifications.

---

## Open Questions

1. How is character Star Rating determined at generation? Fixed by vendor tier, random, or influenced by metacurrency/Backgrounds?
2. Is there a mechanism for a character's Star Rating to change after generation (e.g., prestigious achievements grant a star)?
3. What is the exact distribution/correlation model for Current vs. Potential at generation? (e.g., `Current + Potential = constant`, or independent rolls?)
4. Are there derived stats beyond the four primaries (e.g., Speed = f(Agility), Defense = f(Attributes + equipment))? If so, what are their formulas?
5. Minimum number of ancestry Core Traits needed for MVP? (Determines content volume for non-humanoid anatomical variations.)
6. Can a character have 0 Core Traits (completely "blank slate") or is at least one ancestry Trait required?

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [traits-and-perks](traits-and-perks.md) | Trait Slot count comes from Character Star Rating. Core Traits define anatomical slots. |
| [combat](combat.md) | Primary Attributes feed into all combat calculations (Attack, Defense, Initiative, etc.) |
| [equipment](equipment.md) | Anatomical Slot availability determines what equipment a character can wear. Attribute requirements gate equipment access. |
| [economy](economy.md) | Character value (star rating, attributes, traits) determines hiring cost and upkeep. |
| [roster-management](roster-management.md) | Character generation rules define what vendors produce. Injury system modifies Attributes and anatomical slots. |

---

_Last updated: 2026-02-11_
