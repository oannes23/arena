# Equipment — Domain Specification

**Status**: 🟢 Complete
**Last interrogated**: 2026-02-21
**Last verified**: —
**Depends on**: [characters](characters.md), [traits-and-perks](traits-and-perks.md), [combat](combat.md)
**Depended on by**: [economy](economy.md), [consumables](consumables.md), [groups](groups.md)

---

## Overview

Equipment is the gear system — weapons, focuses, armor, accessories, and shields that characters equip to enhance combat capabilities. Each character has a set of **Anatomical Slots** determined by ancestry (default humanoid: 11 slots). Hand slots hold two parallel implement categories: **Weapons** (physical combat implements) and **Focuses** (magical combat implements). Equipment provides implement-type-defined combat formulas, quality-scaled base stats, Diablo-style affixes with tiered pools, and optional granted Actions. Equipment degrades through usage-proportional wear, creating a permanent economic sink with tiered repair options.

---

## Slots & Equipping

### Anatomical Slot Constraints

- **Decision**: Each item occupies one or more Anatomical Slots. Anatomical Slots prevent illogical stacking and naturally limit loadout size.
- **Rationale**: Anatomical slots alone create sufficient loadout constraints. A humanoid character can fill ~12 anatomical positions. Multi-slot items (like Full Plate covering Head+Torso+Arms+Legs) reduce total item count as a natural consequence.
- **Implications**: Equipment UI shows Anatomical Slot coverage.

### Anatomical Slots (Default Humanoid)

- **Decision**: Default humanoid characters have 11 Anatomical Slots: Head, Torso, Arms, Legs, Hands, Feet, 2x Hand (weapon/shield), 2x Finger, Neck, Back.
- **Rationale**: Full coverage for interesting equipment puzzles. Separating Hands/Feet from Arms/Legs enables granular armor choices. Accessories (Finger, Neck, Back) provide pure affix slots without competing with armor.
- **Implications**: Non-humanoid ancestry Core Traits can modify this set (extra limbs, missing slots, merged slots). Characters spec should define this as the default anatomy.

| Slot | Count | Equipment Types |
|------|-------|----------------|
| Head | 1 | Helmets, Hoods, Coifs |
| Torso | 1 | Chest armor, Robes |
| Arms | 1 | Bracers, Pauldrons, Wraps |
| Legs | 1 | Greaves, Leggings |
| Hands | 1 | Gauntlets, Gloves |
| Feet | 1 | Boots, Sandals, Sabatons |
| Hand | 2 | Weapons, Focuses, Shields |
| Finger | 2 | Rings |
| Neck | 1 | Amulets |
| Back | 1 | Cloaks |

### Equipment Requirements

- **Decision**: Acquisition-only gates. Requirements checked at equip time; removing the requirement source (e.g., respec removing a Trait) does NOT unequip the item.
- **Rationale**: Consistent with [traits-and-perks](traits-and-perks.md) acquisition-only philosophy. Prevents disruptive auto-unequip cascades.

Requirements can include:
- **Attributes**: Minimum values (e.g., Might 60 for Heavy Armor)
- **Traits**: Must possess a specific Trait
- **Perks**: Must know a specific Perk

### Equipment Combat Lock

- **Decision**: Equipment is locked at combat start for **all fight types** (not just tournaments). No mid-combat gear swaps. Degradation is calculated post-combat, not during.
- **Rationale**: Simple, consistent. Equipment decisions happen between fights. No mid-fight affix deactivation from degradation. Equipment is snapshotted at combat start.
- **Implications**: Tournament lock at registration is a separate, stricter constraint (locks between rounds too).

### Item Identification

- **Decision**: All affixes are always fully visible immediately on acquisition. No identification/appraisal mechanic.
- **Rationale**: Respects player time. Players can evaluate loot instantly. Evaluation skill is build synergy recognition, not stat discovery.

---

## Hand Slots & Implements

### Wielding Modes

- **Decision**: 2 Hand anatomical slots with flexible assignment. Two parallel implement categories occupy Hand slots: **Weapons** (physical combat — Sword, Dagger, Mace, Axe, Spear, Greatsword, Greataxe, Warhammer, Bow, Crossbow, Hand Crossbow, Sling) and **Focuses** (magical combat — Wand, Tome, Staff, Orb, Holy Symbol, Artifact, Rod). Shields also occupy a Hand slot. Any combination of implements is valid.
- **Rationale**: Weapons are to physical attacks what Focuses are to magical attacks. Clean parallel system. Maximum build flexibility.

**Implement categories:**
- **Weapons**: Define base damage and Attack/Damage formulas for physical Actions.
- **Focuses**: Define base magic power and formulas for magical Actions. Work identically to weapons mechanically — each focus type defines base magic power, formula weights, damage type, tags, range, and combat role. The magical parallel to physical weapons.
- **Shields**: Defensive items. Block chance + block value. Can pair with either a weapon or a focus.

**Wielding configurations** (any combination of 2 Hand-slot items is valid):
- **Weapon + Shield**: 1 one-hand weapon + 1 shield. Physical combat with defense.
- **Focus + Shield**: 1 one-hand focus + 1 shield. Magical combat with defense.
- **Weapon + Focus**: 1 weapon + 1 focus. **Hybrid build** — physical Actions use the weapon's base stats, magical Actions use the focus's base stats. Off-hand item fires a generic bonus Attack of its type after each Action (see Off-Hand Bonus Attack below).
- **Two-Handed Weapon**: 1 two-hand weapon (Greatsword, Greataxe, Warhammer, Bow, Crossbow). Both Hand slots. Higher base damage.
- **Two-Handed Focus**: 1 two-hand focus (Staff, Tome). Both Hand slots. Higher base magic power.
- **Dual-Wield Weapons**: 2 one-hand weapons. See Dual-Wield rules below.
- **Dual-Wield Focuses**: 2 one-hand focuses. Same dual-wield rules as weapons, applied to magic.
- **Dual Shield**: 2 shields. Valid but unusual. No offensive implement — uses Unarmed/Unfocused defaults.
- **Empty Hands**: Unarmed physical + Unfocused magic. Valid build path with Perks.

**Off-hand restrictions (default):** Off-hand is restricted to **Light-tagged** items by default (applies to both weapons and focuses). Light weapons: Dagger, Hand Crossbow, Sling. Light focuses: Wand, Orb, Holy Symbol, Artifact, Rod. Two-Handed items (Greatsword, Greataxe, Warhammer, Bow, Crossbow, Staff, Tome) use both slots. A Perk (e.g., Ambidextrous) can override this restriction, allowing any one-hand item in the off-hand. Shields are exempt from the Light restriction.

**Versatile**: Applies to both weapons and focuses. Automatically uses two-hand mode if off-hand is empty; one-hand mode if off-hand has an item. Each Versatile type defines **two stat profiles**: one-hand mode (lower base, faster) and two-hand mode (higher base, slower or stronger). The type template carries both profiles.

### Off-Hand Bonus Attack

- **Decision**: When two Hand-slot items are equipped (any combination), each Action triggers a **bonus generic Attack** from the off-hand implement. The bonus attack is always a basic Attack Action — even when the main-hand uses a special Perk Action.
- **Rationale**: Creates satisfying hybrid combat. A warrior with a Sword+Orb uses Power Strike (physical, main-hand) and the Orb automatically fires a generic magic Attack afterward. Reversal: Fireball (magic, off-hand focus) and the Sword fires a generic physical Attack.

**Mechanics:**
- Main-hand Action resolves first with full effects (whether generic Attack or special Perk Action)
- Off-hand fires a **generic Attack** using the off-hand implement's base stats and type (physical for weapons, magic for focuses)
- **Dual-wield penalties apply**: main-hand takes a **−15%** Attack/Damage penalty; off-hand takes a **−35%** Attack/Damage penalty
- Perks can mitigate penalties (Ambidextrous, Two-Weapon Fighting — apply to both weapon and focus dual-wielding)
- **Shield off-hand**: No bonus attack (shields don't have offensive stats)
- **Weapon Triggers fire per-weapon/focus**: Each implement's proc Triggers (OnHit, etc.) fire only on that implement's hit, not on both

**Examples:**
- Sword (main) + Orb (off): Attack → Sword strike (small penalty) + Orb generic magic attack (moderate penalty). Fireball → Orb magic (moderate off-hand penalty) + Sword generic physical attack (small main-hand penalty).
- Wand (main) + Dagger (off): Magic Missile → Wand magic (small penalty) + Dagger generic physical attack (moderate penalty).
- Dagger (main) + Dagger (off): Attack → main Dagger (small penalty) + off Dagger (moderate penalty). Classic weapon dual-wield.

### Implement Resolution

- **Decision**: Actions specify an **implement type** (physical or magical). The system resolves which equipped item provides base stats using a fallthrough: main-hand → off-hand → default.
- **Rationale**: Clean parallel system. Hybrid builds naturally use the right implement for each Action type.

**Resolution order:**
1. Check **main-hand** for a matching implement type (weapon for physical, focus for magical)
2. If main-hand doesn't match, check **off-hand**
3. If neither hand has a matching implement, use the **default**: Unarmed (physical) or Unfocused (magical)

**Examples:**
- Sword (main) + Shield (off): Physical Actions → Sword. Magic Actions → Unfocused default.
- Sword (main) + Orb (off): Physical Actions → Sword. Magic Actions → Orb.
- Wand (main) + Shield (off): Magic Actions → Wand. Physical Actions → Unarmed default.
- Wand (main) + Orb (off): Magic Actions → Wand (main-hand priority). Off-hand Orb provides affixes/procs only.

### Default Attack Adaptation

- **Decision**: A single default **Attack** Action (from the Combatant system Trait) adapts to the main-hand implement type. Not two separate "physical Attack" and "magic Attack" defaults.
- **Rationale**: Simple. One button. The implement determines behavior. Clean parallel to how implement resolution already determines which base stats to use.

**Adaptation by implement type:**
- **Weapon main-hand** → physical Attack (uses weapon base damage, weapon's damage type, weapon's range)
- **Focus main-hand** → magical Attack (uses focus base magic power, focus's damage type, focus's range, focus's targeting pattern)
- **No implement** → Unarmed (physical, Short range, single target)

**The Multihit tag changes default Attack targeting:**
- **Without Multihit** (Wand, Artifact, Rod, all weapons): Attack targets `[Enemy, Single]`
- **With Multihit** (Tome, Orb, Holy Symbol): Attack hits 1–5 enemies in the target zone (see Implement Tags for Multihit mechanics)

**Focus as damage basis:** The equipped focus provides the default damage basis for magical Actions, just like weapons for physical Actions. Individual magical Perk Actions commonly override the focus's default range and/or targeting tags (more frequently than physical Perk Actions override weapon defaults), but still use the focus's base magic power in the damage calculation.

### Unarmed & Unfocused Defaults

- **Decision**: Characters can intentionally fight with empty Hand slots. The Combatant system Trait provides both **Unarmed** (physical default) and **Unfocused** (magic default).
- **Rationale**: Unarmed is a build path, not just a fallback. A Monk character with empty Hand slots and Unarmed-enhancing Perks is a valid and potentially strong build.

**Unarmed**: Physical combat default. Blunt damage, Short range, single target. Might-scaled. Functional but weaker than weapons. Perks (e.g., Monk builds) can enhance to competitive levels.

**Unfocused**: Magical combat default. Weak base magic power. Slightly stronger/less punishing than physical Unarmed. A warrior with no focus can still use a magic Perk, just weakly.

---

## Weapon Types (MVP)

Weapons are **physical combat implements**. Each weapon type defines: base damage, Attack formula weights, Damage formula weights, default damage type, tags, range, and Action Speed modifier. Individual items of a weapon type vary only by quality and affixes. Weapon types are content data.

| Weapon Type | Damage Type | Range | Tags | Formula Focus |
|-------------|-------------|-------|------|--------------|
| **Sword** | Slashing | Short | Versatile, Defend | Might + Accuracy blend |
| **Dagger** | Piercing | Short | Light, Finesse, Deadly | Speed + Accuracy |
| **Mace** | Blunt | Short | Sunder | Might-heavy |
| **Axe** | Slashing | Short | Versatile, Deadly | Might + Speed blend |
| **Spear** | Piercing | Short | Reach, Versatile, Defend | Might + Accuracy + Speed blend |
| **Greatsword** | Slashing | Short | Two-Handed, Heavy, Defend | Might + Accuracy |
| **Greataxe** | Slashing | Short | Two-Handed, Heavy, Deadly | Might + Speed |
| **Warhammer** | Blunt | Short | Two-Handed, Heavy, Sunder | Might-dominant |
| **Bow** | Piercing | Long | Two-Handed, Ranged, Deadly | Accuracy + Speed |
| **Crossbow** | Piercing | Long | Two-Handed, Ranged, Sunder | 65% Accuracy + 20% Might + 15% Awareness |
| **Hand Crossbow** | Piercing | Long | Light, Ranged | 50% Accuracy + 50% Speed |
| **Sling** | Blunt | Long | Light, Ranged | 50% Speed + 35% Accuracy + 15% Might |
| **Unarmed** | Blunt | Short | — | Might-scaled (system default, always available) |

**Notes:**
- Unarmed is the system default from the Combatant system Trait. Functional but weaker than weapons. Perks can enhance (Monk builds).
- **Three two-handed melee archetypes**: Greatsword (Defend — defensive parrier), Greataxe (Deadly — crit-focused berserker), Warhammer (Sunder — armor-piercing crusher). Each has a distinct tactical identity.
- **Ranged weapons**: Bow (Two-Handed, Deadly — precision crits), Crossbow (Two-Handed, Sunder — armor-piercing bolts), Hand Crossbow (Light, no implement tags — identity is one-handed ranged for off-hand dual-wield), Sling (Light, blunt damage at range, cheap option). All Long range.
- **Action Speed is tag-derived**: Light items get a positive Action Speed bonus, Heavy items get a penalty, Two-Handed items get a small penalty. No per-type individual modifiers — type identity comes from damage, formulas, and implement tags. Exact tag-to-Action-Speed values are tuning.
- **Thrown weapons** (javelins, throwing axes): Deferred. When added, they will be **Consumables** (limited use, fits consumable slot system) rather than equipment.
- **Staff, Wand, and Tome** are **Focuses** (magical implements), not weapons. See Focus Types below.

---

## Focus Types (MVP)

Focuses are **magical combat implements** — the magical parallel to weapons. Each focus type defines: base magic power, formula weights, default damage type (varies per item), tags, range, Action Speed modifier, and **combat role** (targeting pattern, special bonuses). Focuses provide the default damage basis for magical Actions the same way weapons provide the default damage basis for physical Actions. Magical Perk Actions commonly override focus defaults for range and targeting while still using the focus's base magic power.

| Focus Type | Range | Tags | Formula Focus | Combat Role |
|------------|-------|------|--------------|-------------|
| **Wand** | Long | Focus, Light, Deadly | Intellect + Accuracy | Sniper — lower base damage, crit bonus, single target |
| **Tome** | Long | Focus, Two-Handed, Multihit | Intellect + Awareness | AoE lobber — hits 1–5 enemies in same zone |
| **Staff** | Medium | Focus, Two-Handed, Defend, Deflect | Intellect + Willpower | Tanky caster — high base damage, Phys + Magic Defense |
| **Orb** | Medium | Focus, Light, Multihit | Intellect-weighted | AoE — hits 1–5 enemies in same zone |
| **Holy Symbol** | Short | Focus, Light, Deflect, Multihit | Willpower-weighted | AoE with Magic Defense bonus |
| **Artifact** | Long | Focus, Light | Varies per item | Wild card — template overrides per specific artifact |
| **Rod** | Short | Focus, Light, Defend | Willpower + Endurance | Caster tank — Physical Defense bonus, one-handed |
| **Unfocused** | Short | — | Weak default | System default, always available |

**Notes:**
- All focus damage types vary by item instance (Fire, Shadow, Light, etc.) — set per individual item, not per focus type.
- **Base magic power hierarchy (parallel to weapons)**: Staff/Tome highest (Two-Handed premium). Rod/Orb moderate (one-handed baseline). Wand lower (offset by Deadly crit). Holy Symbol lower (offset by Deflect+Multihit). Artifact moderate default (overridable per template).
- **Wand**: Long-range single-target sniper. Lower base magic power compensated by **Deadly** tag (crit chance bonus). Intellect + Accuracy formula rewards precision builds.
- **Tome**: Long-range AoE lobber. **Multihit** tag — default Attack hits 1–5 enemies in the same zone. Two-Handed. Intellect + Awareness formula.
- **Staff**: Medium-range tanky two-hander. High base magic power. **Defend + Deflect** tags grant Physical + Magic Defense bonuses. Two-Handed. Intellect + Willpower formula. The magical equivalent of Greatsword/Warhammer.
- **Orb**: Medium-range AoE. **Multihit** tag — hits 1–5 enemies. One-hand, Light. Intellect-weighted (arcane/destructive affinity).
- **Holy Symbol**: Short-range AoE with magic defense. **Deflect + Multihit** tags — hits 1–5 enemies and grants Magic Defense bonus. One-hand, Light. Willpower-weighted (divine/protective affinity).
- **Artifact**: Long-range single-target wild card. Individual Artifact item templates can **override base combat role fields** (range, tags, bonuses). The open creative category for unique magical implements. Artifacts are less a fixed type and more a category — non-unique Artifacts use Long/single-target defaults; Unique Artifact templates define their own identity. One-hand, Light.
- **Rod**: Short-range single-target caster tank. **Defend** tag grants Physical Defense bonus. One-hand, Light. Willpower + Endurance formula. The magical equivalent of a Mace — a one-handed defensive caster weapon.
- **Unfocused** is the magic equivalent of Unarmed — the system default from the Combatant system Trait. Slightly stronger than physical Unarmed. Perks can enhance.
- Focus types can have proc Triggers (OnHit for magic attacks) just like weapons.
- **Staves and Tomes** are Two-Handed (use both Hand slots). **Wands, Orbs, Holy Symbols, Artifacts, and Rods** are Light (can go in off-hand by default).

---

## Tag System

Tags are shared mechanical keywords connecting equipment, consumables, and Perks. **Tag-granting affixes**: Any implement tag can be added to any weapon or focus via an affix (e.g., a "Defending" affix adds the Defend tag to a Wand) or a Unique Item template override. This is a common affix pattern — it enables build customization beyond type defaults.

### Structural Tags

| Tag | Applies to | Effect |
|-----|-----------|--------|
| Reach | Weapons | Attack Medium range from Short position; **~20% Attack Value penalty on Reaching attacks**. Polearm Master synergy. |
| Two-Handed | Both | Both Hand slots; higher base stats; no off-hand |
| Light | Both | Faster Action Speed; reduced attribute requirements. **Can go in off-hand by default.** |
| Heavy | Weapons | High damage; Speed penalty; higher attribute requirements |
| Finesse | Weapons | Speed-weighted Attack/Damage formulas. *(Crit bonus is the Deadly tag — Finesse weapons that crit well carry both tags.)* |
| Ranged | Weapons | Attack at Long range; **~20% Attack Value penalty at Short range** (same zone) |
| Versatile | Both | Two-hand mode if off-hand empty (better stats); one-hand mode if off-hand occupied. Two stat profiles per type. |
| Focus | Focuses | Generic tag. Satisfies Focus-gated spell requirements. All focus items carry this tag. |

### Implement Tags

Implement tags define combat role mechanics. Shared across weapons and focuses. Any implement tag can be added to any weapon or focus via affixes.

| Tag | Default on | Effect |
|-----|-----------|--------|
| Defend | Sword, Spear, Greatsword, Staff, Rod | **Passive while equipped.** Grants **Physical Defense bonus** ≈ 10% of the implement's computed Attack Value (quality-scaled, attribute-blended, Perk-adjusted). |
| Deflect | Staff, Holy Symbol | **Passive while equipped.** Grants **Magic Defense bonus** ≈ 10% of the implement's computed Attack Value. |
| Deadly | Dagger, Axe, Greataxe, Bow, Wand | **Per-implement only** — grants **flat crit chance bonus** (e.g., +5%) on attacks made with this implement. Does NOT scale with quality — crit scaling comes from underlying damage. |
| Sunder | Mace, Warhammer, Crossbow | **Per-implement only** — grants **flat Penetration bonus** on attacks made with this implement. Does NOT scale with quality. |
| Multihit | Tome, Orb, Holy Symbol | Default Attack hits **1–5 enemies** in the target zone. See Multihit mechanics below. |

**Defend/Deflect formula**: `Defense bonus = Attack_Value × ~0.10` where Attack Value includes all modifiers (base × quality × formula weights × Perk bonuses). The ~10% ratio is a tuning value. A high-quality Staff with high Intellect grants substantial Physical + Magic Defense. The formula weights of the implement also contribute (e.g., Staff's Willpower weight naturally supports tanky builds).

**Defend/Deflect are passive while equipped**: Like armor Soak, the defense bonus applies as long as the implement is equipped — regardless of which hand slot it occupies or whether it's being used offensively. Just holding a Staff grants both Physical and Magic Defense.

**Deadly/Sunder are per-implement**: Unlike Defend/Deflect, offensive implement tags (Deadly, Sunder) apply only to attacks made with that specific implement. A Dagger (Deadly) in off-hand grants crit bonus on the off-hand bonus attack only — not on main-hand spells. This creates a clean split: defensive tags are character-wide, offensive tags are per-weapon/focus.

**Dual-wield stacking**: When dual-wielding two Defend/Deflect implements, **both contribute**. However, the Attack Value used to compute each implement's defense bonus already includes dual-wield penalties (main-hand −15% penalty, off-hand −35% penalty). This naturally limits stacking unless Ambidextrous Perks remove the penalties. An invested dual-wielder with both Defend items + Ambidextrous can build very high defensive parrying — a valid and intentional build path.

**Multihit mechanics** (detailed resolution owned by combat spec):
- **Target count**: Rolled with triangular distribution peaked at 3 (weights: 1/2/3/2/1 for 1/2/3/4/5 targets, i.e., ~11%/22%/33%/22%/11%).
- **First target**: Always the target selected by the character (standard accuracy check).
- **Additional targets**: Other hostile targets in the same zone. Capped by available valid targets (if 5 rolled but only 2 hostiles in zone, hits 2).
- **Accuracy penalty**: Additional hits take a **~15–20% percentage penalty** to Attack Value (compensable via affixes or Perks). Full damage on hit.
- **Hostiles only**: No friendly fire.

### Armor Tags

| Tag | Examples | Effect |
|-----|----------|--------|
| Light Armor | Hoods, vests, wraps, leggings, gloves, sandals | No Speed penalty; low protection; caster/rogue-friendly |
| Medium Armor | Coifs, hauberks, bracers, greaves, gauntlets, boots | Moderate protection; small Speed penalty |
| Heavy Armor | Helms, cuirasses, pauldrons, plate legs, plate gauntlets, sabatons | High protection; Speed/dodge penalty; attribute requirements |
| Shield | Bucklers, kite shields, tower shields | Off-hand; block chance + block value; defensive Perk synergy |

---

## Armor System

Each armor type defines: base Soak contribution, attribute-scaling weights, anatomical slot coverage, and tags. Armor provides a **hybrid** defensive contribution: flat base Soak (quality-scaled) PLUS an attribute-scaling component per armor type.

### Per-Slot Armor Pieces

Each anatomical slot has Light, Medium, and Heavy variants:

| Slot | Light | Medium | Heavy |
|------|-------|--------|-------|
| Head | Hood | Coif | Helm |
| Torso | Vest | Hauberk | Cuirass |
| Arms | Wraps | Bracers | Pauldrons |
| Legs | Leggings | Greaves | Plate Legs |
| Hands | Gloves | Gauntlets | Plate Gauntlets |
| Feet | Sandals | Boots | Sabatons |

**Armor category attribute scaling:**
- **Light**: 60% Speed + 30% Awareness + 10% Endurance (agile, perceptive, slight toughness)
- **Medium**: 40% Endurance + 30% Speed + 30% Awareness (balanced three-way)
- **Heavy**: 55% Endurance + 30% Might + 15% Speed (tanky, strong, slightly athletic)

### Multi-Slot Armor

Multi-slot items cover multiple anatomical slots. Base stats scale **less than proportionally** with slots covered (diminishing returns: ~1.0/1.7/2.2/2.6/3.0x for 1-5 slots). Same affix budget as single-slot items. Multi-slot items trade affix sources (fewer items = fewer affixes) for consolidated base stats and simpler inventory management.

| Item | Slots Covered | Category | Base Stat Multiplier |
|------|---------------|----------|---------------------|
| Robes | Torso + Arms | Light | ~1.7× |
| Chainmail | Torso + Arms | Medium | ~1.7× |
| Full Plate | Head + Torso + Arms + Legs | Heavy | ~2.6× |

### Armor Speed Penalties

- **Decision**: Flat Speed reduction per equipped armor piece. Light = 0. Medium = **−2 per piece**. Heavy = **−5 per piece** (Bonus Modifier, negative). Stacks additively with number of pieces.
- **Rationale**: Simple, predictable. Players trade Speed for Soak. Full Heavy plate (6 pieces = −30 Speed, ~60% reduction on average Speed 50) is a major Speed sacrifice. Per-piece granularity rewards mixed loadouts.
- **Implications**: Full Medium set = −12 Speed. Full Heavy = −30 Speed. A character in 6 Heavy pieces is genuinely slow — forces mixed loadouts or Speed investment to compensate. Light armor wearers pay no Speed tax.

### Armor Soak Family Contribution

- **Decision**: All armor contributes to all three Soak families (Physical, Elemental, Magical), but at different ratios per armor category.
- **Rationale**: Smoother defense than category-to-family mapping. Less punishing for mixed-enemy encounters. Characters always get some benefit from armor against any damage type.

| Category | Physical Soak | Elemental Soak | Magical Soak |
|----------|--------------|----------------|--------------|
| Heavy | Primary (~60%) | Secondary (~25%) | Tertiary (~15%) |
| Medium | Balanced (~40%) | Balanced (~35%) | Low (~25%) |
| Light | Low (~15%) | Secondary (~25%) | Primary (~60%) |

Combined with the attribute-scaling component (Heavy: Endurance+Might, Medium: Endurance+Speed, Light: Speed+Awareness), a character's total Soak from armor depends on both gear choices and attribute investment.

### Armor Mixing

- **Decision**: Free mixing of armor categories with no mechanical interaction. Each piece applies its own Soak contribution and Speed penalty independently.
- **Rationale**: Maximum build flexibility. No set bonuses, no heaviest-dominates penalty. A character with Heavy helm + Light vest + Medium boots gets exactly the sum of each piece's stats.
- **Implications**: No category-tracking needed beyond individual piece data. Build optimization comes from matching pieces to desired Soak ratios and acceptable Speed penalties.

### Shield Types & Block Mechanic

- **Decision**: Shield items provide a block mechanic separate from the Shield HP-pool combat mechanic. Block is a post-hit flat damage reduction.
- **Rationale**: Two distinct defense layers — physical shields (item-based block) and magical shields (Perk-based HP pools). Stacks for characters with both.
- **Implications**: New combat pipeline step (post-hit, pre-Soak). See combat spec.

**Block mechanic:**
1. Attack hits (passes Attack vs Defense roll)
2. **Block check**: Shield's block chance % triggers
3. On successful block: subtract shield's **block value** (flat amount, quality-scaled) from damage before Soak. For multi-component damage (e.g., dual-wield, Actions with multiple Damage components), block value is **split proportionally** across all damage components based on their relative magnitudes.
4. Remaining damage proceeds to Soak reduction, then Shield HP absorption, then Health

| Shield | Block Chance | Block Value | Notes |
|--------|-------------|-------------|-------|
| Buckler | ~20% | Low | Light, minimal Speed penalty |
| Kite Shield | ~35% | Moderate | Medium, some Speed penalty |
| Tower Shield | ~50% | High | Heavy, significant Speed penalty, high attribute req |

---

## Accessory Types (MVP)

Accessories occupy dedicated anatomical slots that don't overlap with weapons or armor. They provide **no base combat stats** — purely affix carriers.

| Type | Slot | Count | Notes |
|------|------|-------|-------|
| Ring | Finger | 2 | Pure affix item. Small but stackable. |
| Amulet | Neck | 1 | Pure affix item. Often resource pool or resistance bonuses. |
| Cloak | Back | 1 | Pure affix item. Often defensive or stealth-related. |

Accessories follow the same quality, degradation, and affix tier systems as all equipment. Their degradation usage factor is low (0.3–0.5x), **proportional to combat length** (ticks survived) since they aren't directly used in combat actions.

---

## Quality & Durability

### Quality System (0–500+)

- **Decision**: Linear scaling. Quality 100 = 1.0x baseline; quality 500 = 5.0x.
- **Rationale**: Quality multiplies weapon base damage, but base damage feeds into attribute-blend formulas from character builds. A low-level character with a 5x weapon roughly matches a high-level character with a 1x weapon on raw numbers — the high-level character wins on Perks, Actions, and options. "Squire with Excalibur vs. Musashi with an oar" — numerically similar, but Musashi wins on build depth.
- **Implications**: Gear is a significant but not dominant power axis. Build optimization (Traits, Perks, Action selection) remains the primary skill expression.

| Quality Range | Star | Color | Modifier |
|---------------|------|-------|----------|
| 0–99 | 0★ | Gray | Below baseline |
| 100 | 1★ | White | 1.0× (baseline) |
| 200 | 2★ | Green | 2.0× |
| 300 | 3★ | Blue | 3.0× |
| 400 | 4★ | Purple | 4.0× |
| 500+ | 5★ | Orange | 5.0×+ |

**Quality scales**: weapon base damage, armor base Soak, block value, and the magnitude of positive/negative affix effects.

**Affix Quality Scaling**: Affix power scales continuously with item quality: `affix_power = base × tier_multiplier × (quality / 100)`. A 3★ affix on a quality-350 item is stronger than the same affix on a quality-300 item. This makes quality meaningful beyond tier thresholds — every quality point matters for affix effectiveness.

### Durability & Degradation

- **Decision**: Usage-proportional degradation with flat-per-fight base and usage weight multiplier. No quality floor — items degrade all the way to 0 and are eventually destroyed.
- **Rationale**: Rewards efficient play (a backup weapon degrades less than a primary). Full decay ensures all gear is temporary — maximum economic pressure.
- **Implications**: Per-fight degradation = `base_degrade × usage_weight`. Base degrade ≈ **2 quality per fight** at 1.0× usage. Usage weight is a normalized factor (0.5× for light use, 1.0× average, 1.5× heavy use). A 100-quality weapon lasts ~50 fights without repair. Usage tracking per equipment category: weapons track Actions taken, armor tracks damage absorbed, accessories track ticks survived.

**Lifecycle**: Item starts at max quality → degrades per fight → repaired (reducing max) → eventually max reaches 0 → **item destroyed**.

### Tiered Repair

- **Decision**: Three repair tiers with different quality/cost trade-offs.
- **Rationale**: Gives players economic choices about gear lifespan. Premium repair is an expensive gold sink that extends top-tier gear.

| Tier | Effect | Cost |
|------|--------|------|
| Cheap | Restores current quality to max − 2 | Low |
| Standard | Restores current quality to max − 1 | Moderate |
| Premium | Restores current quality to max (no permanent loss) | Very expensive |

Premium repair preserves max durability but at significant cost — a long-term investment that delays but doesn't prevent eventual item death.

### Quality Forging & Enchanting

- **Decision**: Group-gated vendor service to increase an item's max quality. Available in MVP 1. When forging pushes quality past a tier threshold (e.g., 299→301), new affix slots open — but filling them requires a separate **Enchanting** service.
- **Rationale**: Counteracts degradation and lets players invest in favorite items. Group-gating (e.g., Blacksmith's Guild) creates natural scarcity and ties into the Groups system. Two-step forge+enchant process creates multiple gold sinks and Group service dependencies.
- **Implications**: Exponentially increasing cost per quality point. Creates "maintain vs. replace" tension — is it cheaper to forge an existing item or find/buy a new one?

**Enchanting Service**: A separate Group vendor service that adds affixes to **empty affix slots** on items. Fills slots whether empty from creation (random 1-to-max generation) or opened via forging past a tier threshold. Same affix tier pools apply — new affixes draw from the item's current quality tier. Enchanting costs gold and potentially materials.

---

## Affix System

### Affix Data Model & Budget

- **Decision**: Tiered affix pools with progressive unlock. Inverse affix budget (fewer affixes = stronger each). **Random affix count** (1 to max per tier). Source-themed generation tables. Both positive and negative (cursed) affixes exist. **Affixes use the unified Perk component data model** — an affix IS a mini-Perk bound to an item.
- **Rationale**: Each quality tier feels qualitatively different (not just "more of the same"). Random affix count creates loot variance — a 3★ item with 1 powerful affix vs. 3 moderate ones. Cursed items create risk/reward decisions. Unified component model maximizes code reuse and consistency with Perk system.

**Unified Affix Data Model**: Each affix is a container of Perk components using the exact same data structures: Stat Adjustments, Triggers, Actions, and/or Damage components. An affix can contain any combination of these. The only difference from Perks: affixes are tied to items (lost when item unequipped/destroyed), Perks are tied to Traits.

**Affix Budget by Tier (inverse scaling):**

| Tier | Max Affixes | Per-Affix Power |
|------|------------|----------------|
| 1★ White | 1 (always 1) | 1.4× |
| 2★ Green | 1–2 (random) | 1.0× each |
| 3★ Blue | 1–3 (random) | 0.7× each |
| 4★ Purple | 1–4 (random) | 0.6× each |
| 5★ Orange | 1–5 (random) | 0.5× each |

Items with fewer than max affixes have **empty affix slots** that can be filled later via the Enchanting service (see Quality Forging & Enchanting).

### Tiered Affix Pools

| Tier | Available Affix Categories |
|------|---------------------------|
| 1–2★ | Basic: Attribute bonuses (Stat Adjustments), simple damage/resistance bonuses |
| 3★ | + Intermediate: Damage type additions (secondary Damage components), proc effects (Triggers), resource pool bonuses, tag-scoped bonuses |
| 4★ | + Advanced: Formula Modifier Tags (attribute substitution, item-local), equipment-granted Actions (simple) |
| 5★ | + Rare: Unique mechanics (complex Triggers, e.g., "On kill, heal 10% HP"), powerful granted Actions, mythic-tier effects |

### Affix Component Types

All use Perk component data model:
- **Stat Adjustments**: Attribute bonuses (+Might, +Speed), tag-scoped bonuses (+10% [Fire] damage, +5 Soak vs [Poison]), resource pool bonuses (pre-multiplier +50 Mana pool), Penetration, Crit bonuses, Formula Modifier Tags (item-local attribute substitution). Negative Stat Adjustments for cursed items.
- **Triggers**: Proc effects (OnHit stun, OnHitBy thorns, lifesteal). Use the combat Trigger system directly — same event types, same effect component lists. Stealth-tagged Triggers don't break Sneaking.
- **Actions**: Granted active abilities (same data model as Perk Actions with AI considerations). Available at 4-5★ tiers. Lost when item unequipped/destroyed.
- **Damage components**: Secondary damage type additions to the weapon's Attack action. Each has its own base damage, damage type, and formula — defined as part of the affix data, same structure as Action Damage components. Resolves through its own Soak family per combat spec.

### Affix Generation (Source-Themed Tables)

- **Decision**: Each loot source has themed affix subtables that bias generation toward thematic affixes.
- **Rationale**: Fighting fire enemies biases toward fire affixes. Buying from the Mage Guild biases toward arcane affixes. Thematic loot rewards target farming.
- **Implications**: Each event type, enemy archetype, and Group vendor defines an affix weight table. Standard affix weights serve as the base; source-specific overrides adjust weights for themed affixes.

### Affix Rerolling (Vendor-Dependent)

- **Decision**: Both single-affix and full-affix rerolling exist, offered by different Group vendors. **Cursed (negative) affixes are rerollable** like any other affix.
- **Rationale**: Single-affix reroll (targeted, precise) and full-affix reroll (gambling, cheaper) serve different player needs. Different vendors offering different services enriches the Groups system. Cursed affixes being rerollable means curses are a temporary inconvenience fixable with gold, not a permanent trap.
- **Implications**: Rerolling costs gold + possibly rare materials. Quality is preserved (rerolling doesn't affect quality or durability). Rerolled affixes draw from the same tier pool as the item's quality tier.

### Cursed Items

- **Decision**: Items can have negative affixes alongside positive ones. Cursed items can be **freely unequipped** — no sticky curse mechanic. Negative affixes are **rerollable** via vendor services.
- **Rationale**: Powerful base stats or rare positive affixes offset by drawbacks (e.g., "-10% Speed", "Takes extra Fire damage"). Creates risk/reward loot decisions. Free unequip ensures player agency — the curse is the tradeoff, not a trap. Parallels the Perk tradeoff design (negative Stat Adjustments + harmful Triggers).
- **Implications**: Cursed items are identified — no hidden negatives. Players always see what they're equipping. Cursed affixes can be rerolled away at standard rerolling cost.

### Equipment-Granted Actions

- **Decision**: Equipment can grant Actions using the same data model as Perk Actions (target tags, effect component list, AI considerations).
- **Rationale**: Powerful items feel unique — finding a wand that grants Fireball is exciting. Makes high-tier loot chase-worthy. Actions are lost when item is unequipped or destroyed.
- **Implications**: Available starting at 4★ (Advanced affix pool) with simple Actions, 5★ (Rare pool) for powerful unique Actions. Equipment-granted Actions participate in the standard AI evaluation pipeline.

---

## Unique Items (Templates)

- **Decision**: Hand-authored unique item templates with fixed names, lore text, and some locked affixes, plus 1-2 random affix slots for per-drop variety.
- **Rationale**: Chase items that define builds. Recognizable but not identical between players. Parallels how Traits are content-defined. Specific drop sources (boss enemies, rare loot table entries, quest rewards) create targeted farming goals.
- **Implications**: Unique item templates are content data: fixed weapon/armor type, fixed quality range, fixed affixes (using unified component model), random affix slot count, source constraints (which events/enemies can drop them).

---

## Inventory & Lifecycle

### Household Stash

- **Decision**: Shared household stash for all unequipped items. No per-character inventory. Large capacity (unlimited or upgradeable cap).
- **Rationale**: Avoids tedious item-shuffling between characters. Any character can equip from stash. Simple management.
- **Implications**: Items in stash do **not degrade** — degradation is combat-only. Dead characters' gear stays equipped on their corpse; if not resurrected, gear returns to stash. No time-based passive decay.

### Death & Equipment

- **Decision**: Equipment preserved on dead character. If resurrected, they keep all gear. If character is permanently lost (not resurrected), gear returns to stash.
- **Rationale**: No gear loss from death. Death is already punishing (resurrection costs, stat penalties). Gear loss on top would feel excessive. Gear returning to stash on permanent death prevents it from being "trapped" on an unresurrectable character.

### Tournament Equipment Lock

- **Decision**: Equipment is locked at tournament registration. No changes between rounds.
- **Rationale**: Adds strategic depth to pre-tournament preparation. Players must plan loadouts considering potential opponents and injury risks. Injuries compound without gear-swap compensation.
- **Implications**: Tournament registration captures the full equipment snapshot. The locked-loadout constraint makes pre-tournament planning a meaningful skill.

### Item Disposal (Sell or Salvage)

- **Decision**: Two disposal paths — sell for gold (percentage of computed value) or salvage for crafting materials.
- **Rationale**: Creates an immediate gold vs. future investment decision. Salvage materials feed into quality forging and affix rerolling, creating a material loop alongside the gold economy.
- **Implications**: Material system design (tiered AND typed materials) is deferred to later phases. For now, salvage yields a generic material placeholder. The structural hooks exist for future expansion.

---

## Loot Generation

- **Decision**: Hybrid model — event loot tables for controlled baseline drops, plus a small chance for defeated enemies to drop their equipped items.
- **Rationale**: Event tables control economy flow. Enemy drops add thematic bonus loot (fight an orc with a greataxe, maybe get that greataxe). Best of both: controlled economy with thematic rewards.

### Event Loot Tables

Each event type (PvE tier, tournament tier) defines:
- **Drop rate** per quality tier (higher-tier events drop higher-quality items more often)
- **Equipment type weights** (event theme biases toward certain weapon/armor types)
- **Affix source tables** (themed affix weights per event/enemy type)
- **Placement scaling**: Per [post-combat](post-combat.md), loot drop rates scale with placement (1st=100%, 2nd=60%, etc.)

### Enemy Equipment Drops

- Small chance per defeated enemy to drop one equipped item
- Dropped items have a **quality penalty** (e.g., −20% from original quality)
- Affixes preserved from the enemy's item
- Only applies to enemies that had visible equipment (not all Ephemeral Combatants)

---

## Decisions Summary

All 8 original open questions resolved plus 95 additional structural decisions (31 rounds total):

| # | Decision | Resolution |
|---|----------|-----------|
| 1 | Quality scaling | **Linear confirmed** — gear multiplies base, but builds provide the base. Squire with Excalibur ≈ Musashi with an oar numerically. |
| 2 | Multi-slot power balance | **Less than proportional** — diminishing returns per slot (~1.0/1.7/2.2/2.6/3.0x). |
| 3 | Loot drop curves | **Event loot tables + enemy drops** — hybrid controlled/thematic system. |
| 4 | Quality forging cost | **Exponential**, Group-gated (e.g., Blacksmith's Guild). MVP 1. |
| 5 | Degradation trigger | **Flat per fight × usage weight** (weapons: Actions taken, armor: damage absorbed). |
| 6 | Quality floor | **No floor, full decay** — items degrade to 0 and are destroyed. Maximum economic pressure. |
| 7 | MVP weapon/armor types | **Moderate** — 12 weapon types, ~18 armor pieces (6 slots × 3 categories) + 3 multi-slot + 3 shields + 3 accessories. |
| 8 | Affix rerolling | **Vendor-dependent** — both single-affix and full-affix rerolling, offered by different Groups. |
| 9 | Block mechanic | **Post-hit flat reduction** — shield items provide block chance % + block value (quality-scaled). |
| 10 | Accessories | **Yes, MVP 1** — Rings, Amulet, Cloak. Pure affix carriers. |
| 11 | Weapon type model | **Weapon type defines all** — base damage, formula weights, tags, range, speed. Items vary by quality + affixes. |
| 12 | Affix formula overrides | **Rare affixes can include Formula Modifier Tags** — item-local attribute substitution (parallels Perk system). |
| 13 | Repair model | **Three tiers** — Cheap (max−2), Standard (max−1), Premium (no loss, very expensive). |
| 14 | Armor stat model | **Hybrid** — flat base Soak (quality-scaled) + attribute-scaling component per armor type. |
| 15 | Resource pool bonuses | **Yes, via affixes** — pre-multiplier pool capacity, same as Perk Stat Adjustments. |
| 16 | Affix generation | **Source-themed tables** — each loot source biases toward thematic affixes. |
| 17 | Affix pools | **Tiered progressive unlock** — 1-2★ basic, 3★ intermediate, 4★ advanced, 5★ rare/unique. |
| 18 | Damage type affixes | **Adds damage component** — +Fire affix adds Fire Damage effect component to Attack action. Per-component Soak resolution. |
| 19 | Item death | **Destroyed** at max durability 0. Clean end-of-life. |
| 20 | Equipment Actions | **Yes** — items can grant Actions (same data model as Perks). Available at 4-5★ affix pools. |
| 21 | Dual-wield | **Two attack rolls** — main-hand and off-hand each make independent Attack rolls. Small penalty on main-hand, moderate penalty on off-hand (Attack and Damage). Perks mitigate. |
| 22 | Versatile mode | **Automatic by off-hand** — two-hand if empty, one-hand if occupied. Each Versatile weapon type defines **two stat profiles** (one-hand and two-hand modes). |
| 23 | Thrown weapons | **Deferred** — will be Consumables when added. |
| 24 | Cursed items | **Yes** — negative affixes alongside positive ones. Identified, no hidden negatives. |
| 25 | Spear range | **Short + Reach tag** — Reaching attacks incur Accuracy penalty. Better than nothing, optimal to close distance. |
| 26 | Armor Speed penalties | **Flat per piece** — each Medium/Heavy piece imposes a flat Speed reduction (Bonus Modifier). Stacks additively. |
| 27 | Armor Soak families | **All armor boosts all three** — different ratios per category (Heavy: Physical-weighted, Light: Magical-weighted, Medium: balanced). |
| 28 | Armor mixing | **Free, no interaction** — each piece applies independently. No set bonuses, no heaviest-dominates. |
| 29 | Inventory | **Household stash (shared)** — all unequipped items in shared pool. No per-character inventory. |
| 30 | Death & gear | **Preserved on corpse** — resurrect = keep gear. Permanent death = gear returns to stash. |
| 31 | Stash decay | **Combat only** — items in stash don't degrade. No time-based passive decay. |
| 32 | Proc effects | **Trigger-based** — proc affixes are equipment-granted Triggers using the combat Trigger system. No new mechanic. |
| 33 | Ranged penalty | **Accuracy penalty at Short** — Ranged weapons take Accuracy penalty at Short range. Mirrors Reach penalty. |
| 34 | Affix data model | **Unified with Perk components** — affixes use exact same data structures as Perks (Stat Adjustments, Triggers, Actions, Damage). |
| 35 | Salvage materials | **Tiered + typed (deferred)** — will have quality tiers and material types. Detailed system deferred; placeholder for now. |
| 36 | Unique items | **Templates** — fixed name/lore/some affixes + 1-2 random slots. Chase items from specific sources. |
| 37 | Selling | **Sell or salvage** — sell for gold (% of value) or salvage for materials (forging/rerolling). |
| 38 | Tournament gear | **Locked at registration** — no equipment changes between tournament rounds. |
| 39 | Unarmed choice | **Always available** — empty Hand slots = Unarmed (Combatant system Trait). Valid build path with Perks. |
| 40 | Affix quality scaling | **Continuous** — `affix_power = base × tier_multiplier × (quality/100)`. Every quality point matters for affix effectiveness, not just tier thresholds. |
| 41 | Block vs multi-component | **Proportional split** — block value is split across damage components proportionally by magnitude. |
| 42 | Dual-wield Action model | **One Action, bonus attack** — main-hand is primary Action (full Initiative cost), off-hand fires a generic bonus Attack. Not two separate Actions. |
| 43 | Weapon Triggers in DW | **Per-weapon** — each implement's proc Triggers fire only on that implement's hit. Independent proc sources. |
| 44 | Cursed item removal | **Free unequip** — no sticky curses. Player always has agency. |
| 45 | Hand slot combos | **Any 2 hand items** — weapon+focus, focus+focus, weapon+weapon, shield+anything. No restrictions on combinations. |
| 46 | Off-hand weapon restrict | **Light only (default)** — Perk override (e.g., Ambidextrous) allows any one-hand item in off-hand. |
| 47 | Forge + affixes | **Two-step: forge + enchant** — forging past tier threshold opens affix slots; separate Enchanting service fills them. |
| 48 | Affix count per item | **Random 1 to max** — affix count is random within 1 to tier max. Fewer affixes = stronger each (inverse budget). |
| 49 | Cursed affix rerolling | **Rerollable** — negative affixes participate in normal rerolling. Curses fixable with gold. |
| 50 | Weapon vs Focus categories | **Parallel categories** — Weapons (physical) and Focuses (magical) are two distinct Hand-slot item types. Staff/Wand/Tome are Focuses, not weapons. |
| 51 | Equipment combat lock | **All fights locked** — equipment snapshotted at combat start for all fight types. Degradation post-combat. |
| 52 | Item identification | **Always visible** — no identification mechanic. All affixes visible on acquisition. |
| 53 | Focus parallel to weapons | **Same mechanical role** — focuses define base magic power and formulas for magic Actions, just as weapons define base damage and formulas for physical Actions. |
| 54 | Unfocused magic default | **Weak default** — magic without a focus = "Unfocused" (slightly stronger than physical Unarmed). System Trait provides both defaults. |
| 55 | Focus MVP types | **7 types: Wand, Tome, Staff, Orb, Holy Symbol, Artifact, Rod** — Staff/Tome two-handed; Wand/Orb/Holy Symbol/Artifact/Rod one-hand Light. (Revised Round 24 — was Staff/Wand/Tome/Orb/Relic.) |
| 56 | Enchanting fills slots | **Yes** — enchanting adds affixes to any empty slot (creation-empty or forge-opened). Sub-max items upgradeable. |
| 57 | Implement resolution | **Fallthrough: main→off→default** — physical Actions check for weapon, magic Actions check for focus. Main-hand priority when both hands match. |
| 58 | Focus tag | **Generic "Focus"** — replaces "Arcane Focus". All focuses carry it. Perk synergies use other tags for specificity. |
| 59 | Off-hand bonus attack | **Always generic Attack** — off-hand fires generic Attack (physical or magic per implement type) even when main-hand uses special Perk Action. |
| 60 | Focus type redesign | **7 types with combat roles** — Wand (Long, sniper, crit bonus), Tome (Long, AoE multi-target), Staff (Medium, high damage, defense bonuses, two-handed), Orb (Medium, multi-target), Holy Symbol (Short, multi-hit, magic defense), Artifact (Long, wild card with per-item overrides), Rod (Short, caster tank, defense bonuses, one-handed). Relic split into Holy Symbol + Artifact. |
| 61 | Focus as damage basis | **Focus = magic weapon for damage** — equipped focus provides default damage basis for magic Actions. Perk Actions commonly override range/tags but use focus base magic power. More frequent overrides than physical Actions. |
| 62 | Default Attack adaptation | **One adaptive Attack** — single default Attack Action adapts to main-hand implement type (physical for weapons, magical for focuses). Multihit tag changes default Attack to AoE. |
| 63 | Implement defense/crit tags | **Defend, Deflect, Deadly tags** — implement tags granting Defense bonuses or crit bonus. Defend = +~10% Attack Value → Physical Defense. Deflect = +~10% Attack Value → Magic Defense. Deadly = crit chance bonus. Apply to both weapons and focuses. |
| 64 | Multihit unified | **Same mechanic** — multi-target and multi-hit are one mechanic called "Multihit." Hits 1–5 enemies in the zone. |
| 65 | Artifact template overrides | **Per-item overrides** — individual Artifact item templates can override base combat role fields (range, tags, targeting, bonuses). Artifact is a category of unique magical implements. |
| 66 | Tag assignments (initial) | **Staff** (Defend+Deflect), **Holy Symbol** (Deflect+Multihit), **Rod** (Defend), **Wand** (Deadly), **Tome** (Multihit), **Orb** (Multihit), **Axe** (Deadly), **Dagger** (Deadly). *(Mace initially Defend here; changed to Sunder in #74.)* |
| 67 | Finesse loses crit | **Deadly is the crit tag** — Finesse tag now means Speed-weighted formulas only. Crit bonus separated into Deadly tag. Weapons wanting both carry both tags (e.g., Dagger: Light, Finesse, Deadly). |
| 68 | Weapon tag expansion | **Dagger** gets Deadly (crits fit rogue fantasy). *(Mace initially got Defend here; changed to Sunder in #74.)* Implement tags apply to both weapons and focuses. |
| 69 | Spear gets Defend | **Reach + Versatile + Defend** — phalanx fighter. Defensive reach weapon. Three tags is fine for a versatile type. |
| 70 | DW defense stacking | **Both contribute** — dual-wielding two Defend/Deflect implements stacks both bonuses. But Attack Value basis already includes DW penalties, naturally limiting stacking. Ambidextrous Perks unlock full-defense dual-wield builds. |
| 71 | Defense is passive | **Always while equipped** — Defend/Deflect bonuses are passive like armor Soak. Apply regardless of hand slot or offensive use. Just holding a Staff grants Defense. |
| 72 | Deadly is flat | **Flat crit bonus** — does not scale with quality. Crit scaling comes from underlying damage already being quality-scaled. |
| 73 | Sword gets Defend | **Versatile + Defend** — sword can parry (has a hilt). Balanced defensive weapon. |
| 74 | Mace → Sunder | **Sunder replaces Defend** on Mace — blunt weapons bypass armor (Penetration), not parry (no hilt). |
| 75 | Sunder tag | **New implement tag** — grants flat Penetration bonus. Effective against armored targets. On blunt weapons (Mace, Warhammer). Does not scale with quality. |
| 76 | Two-handed archetypes | **Greatsword** (Defend), **Greataxe** (NEW, Deadly), **Warhammer** (Sunder). Three distinct two-handed melee identities: defensive, crit-focused, armor-piercing. |
| 77 | New ranged weapons | **Crossbow** (Two-Handed, Ranged), **Hand Crossbow** (Light, Ranged, one-handed ranged), **Sling** (Light, Ranged, blunt at range). 12 weapon types + Unarmed. |
| 78 | Tag-granting affixes | **Any implement tag via affix** — affixes can add Defend, Deflect, Deadly, Sunder, Multihit to any weapon or focus. Common affix pattern. Unique Item templates can also override tags. |
| 79 | *(Duplicate of #67)* | — |
| 80 | Tag assignments expanded | **Sword** (Defend), **Greatsword** (Defend), **Spear** (Defend), **Mace** (Sunder), **Warhammer** (Sunder), **Greataxe** (Deadly), **Dagger** (Deadly), **Axe** (Deadly). |
| 81 | All ranged Long range | **All ranged weapons are Long range** with no Action Speed penalties. Crossbow's higher damage is its identity, not slow speed. |
| 82 | Bow Deadly, Crossbow Sunder | **Bow** gets Deadly (precision crits). **Crossbow** gets Sunder (armor-piercing bolts). Parallels melee differentiation. **Hand Crossbow and Sling** have no implement tags — identity from Light + Ranged. |
| 83 | Hand Crossbow no tags | **Light + Ranged** is sufficient identity. One-handed ranged for off-hand dual-wield is unique enough. |
| 84 | Channeling tag removed | **Channeling removed** from all focus types and tag lists. Was a placeholder with no defined mechanics. Can be re-added if a specific purpose is identified. |
| 85 | Multihit mechanics | **Random 1–5 targets** with triangular distribution (peak 3, weights 1/2/3/2/1). First target chosen normally; additional targets are other hostiles in zone. Additional hits take flat Accuracy penalty, full damage on hit. Capped by valid targets. Hostiles only, no friendly fire. |
| 86 | Focus base power hierarchy | **Parallel to weapons** — Staff/Tome highest (Two-Handed premium). Rod/Orb moderate (one-handed baseline). Wand lower (offset by Deadly). Holy Symbol lower (offset by Deflect+Multihit). Artifact moderate default (overridable per template). |
| 87 | Ranged weapon formulas | **Specific weights locked** — Crossbow: 65% Accuracy + 20% Might + 15% Awareness. Hand Crossbow: 50% Accuracy + 50% Speed. Sling: 50% Speed + 35% Accuracy + 15% Might. |
| 88 | Equipment Slot limit removed | **Anatomical slots only** — removed the 5-item Equipment Slot limit. Characters equip items into Anatomical Slots directly. Loadout size is naturally constrained by available anatomy. |
| 89 | Deadly/Sunder scope | **Per-implement only** — crit bonus (Deadly) and Penetration bonus (Sunder) apply to attacks made with that specific implement. Defend/Deflect remain passive (character-wide while equipped). Clean split: defensive tags = always on, offensive tags = per-weapon/focus. |
| 90 | Action Speed source | **Tag-derived only** — Light (+bonus), Heavy (−penalty), Two-Handed (−small penalty). No per-type individual Action Speed values. Type identity from damage, formulas, and implement tags. |
| 91 | Enchanting theme | **Source-themed** — Enchanting uses the same source-themed affix tables as loot generation. Group vendor theme biases available affixes. Differentiates Group enchanters. |
| 92 | Versatile focuses | **None** — focuses are strictly Light or Two-Handed. Versatile is weapons-only (Sword, Axe, Spear). |
| 93 | Accessory degradation | **Combat-length proportional** — accessories track ticks survived for usage weight. Longer fights = more accessory wear. |
| 94 | DW penalty range | **Main −15%, Off −35%** Attack and Damage. Raw DW ≈ 75% total output (85% + 65%). Perk-gated investment model. |
| 95 | Armor Speed penalties | **Medium −2, Heavy −5 per piece**. Light = 0. Full Heavy (6 pieces) = −30 Speed. Harsh — forces mixed loadouts or Speed investment. |
| 96 | Degradation base rate | **~2 quality per fight** at 1.0× usage. A 100-quality item lasts ~50 fights without repair. Gentle economic pressure. |
| 97 | Range penalty format | **~20% percentage** of Attack Value for Reach at Medium range and Ranged at Short range. Scales with character power — doesn't become irrelevant at high Accuracy. |
| 98 | Multihit penalty format | **~15–20% percentage** of Attack Value per additional target. Consistent with range penalty format. |
| 99 | Economic tuning values | **Fully deferred** to economy spec / implementation — repair costs, forging cost curve, affix rerolling, sell value, enemy drop rates. |
| 100 | Heavy Armor scaling | **55% Endurance + 30% Might + 15% Speed** — tanky, strong, slightly athletic. |
| 101 | Medium Armor scaling | **40% Endurance + 30% Speed + 30% Awareness** — balanced three-way split for jack-of-all-trades. |
| 102 | Light Armor scaling | **60% Speed + 30% Awareness + 10% Endurance** — agile and perceptive with slight toughness. |
| 103 | Characters spec cleanup | **Flag** "5 Equipment Slots" reference for removal — Anatomical Slots only. |

---

## Open Questions

All structural, mechanical, and key tuning range questions are resolved. 103 decisions across 31 rounds.

### Remaining Tuning Values (Implementation-Phase)

These approximate ranges are decided; exact values require playtesting:

1. **Usage weight exact formula**: How Actions taken / damage absorbed / ticks survived map to the 0.5–1.5× range.
2. **Multi-slot base stat multipliers**: Exact values within the ~1.0/1.7/2.2/2.6/3.0× framework.
3. **Block chance/value per shield type**: Exact percentages and scaling (Buckler ~20%, Kite ~35%, Tower ~50% are starting points).
4. **Armor Soak family ratios**: Exact percentages per category (Heavy ~60/25/15, Medium ~40/35/25, Light ~15/25/60 are starting points).
5. **Focus base magic power values**: Exact numbers per focus type following the established hierarchy.
6. **Action Speed tag bonuses**: Exact values for Light (+X), Heavy (−Y), Two-Handed (−Z).

### Deferred to Economy Spec / Implementation
- **Repair tier costs**, **quality forging cost curve**, **affix rerolling costs**, **enchanting costs** — all economic numbers.
- **Enemy equipment drop chance and quality penalty** — economy/loot balancing.
- **Item sell value formula** — economy balancing.

### Deferred Systems
- **Salvage material system (tiered + typed)**: Structural hooks exist, detailed material economy deferred to later phases.
- **Unique item content**: Template definitions deferred to content authoring during implementation.

### Deferred to Other Specs
- **Loot table contents and drop rates**: [economy](economy.md) and [post-combat](post-combat.md)
- **Group-specific vendor inventories, forging access, and enchanting access**: [groups](groups.md)
- **Thrown weapons as consumables**: [consumables](consumables.md) (future)
- **Multihit detailed combat resolution**: [combat](combat.md)

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [characters](characters.md) | **Default humanoid anatomy: 11 slots** (Head, Torso, Arms, Legs, Hands, Feet, 2x Hand, 2x Finger, Neck, Back). Attribute values gate equipment requirements. Non-humanoid ancestries can modify slot list. Combatant system Trait must provide both **Unarmed** (physical default) and **Unfocused** (magic default). **Equipment Slot concept removed** — characters equip into Anatomical Slots only (update any references to "5 Equipment Slots"). |
| [combat](combat.md) | **Implement type on Actions**: Actions specify physical or magical implement type. Combat pipeline checks equipped implement via fallthrough resolution (main→off→default). **Off-hand bonus attack**: After each Action, off-hand implement fires a generic Attack of its type (physical/magical). **One adaptive Attack**: Single default Attack Action adapts to main-hand implement type. **Multihit tag**: Attack hits 1–5 enemies in zone (triangular distribution, peak 3, ~15–20% Attack Value penalty per additional hit, capped by available targets). **Defend/Deflect tags (passive)**: Must compute Physical/Magic Defense bonus from ~10% of implement's Attack Value (quality × base × formula × Perks). Character-wide while equipped. **Deadly/Sunder tags (per-implement)**: Crit chance bonus and Penetration bonus apply only to attacks made with that specific implement. **Equipment combat lock**: All fights snapshot equipment at start. **Block Chance step** in pipeline (post-hit, pre-Soak; proportional split for multi-component). **Reach/Ranged penalties**: ~20% Attack Value reduction (percentage, not flat). **Dual-wield** = one Action + off-hand bonus generic Attack, −15% main / −35% off-hand (Attack and Damage). Perks mitigate. Per-weapon/focus Trigger firing. **Versatile** types carry two stat profiles. **Armor**: hybrid Soak + Speed penalties (Medium −2/piece, Heavy −5/piece). Three-attribute scaling per category. Affix power scales continuously with quality. |
| [traits-and-perks](traits-and-perks.md) | Equipment requirements are acquisition-only (gear stays after respec). Affixes can enhance specific Perks. Tags create Perk synergies. Equipment Formula Modifier Tags use same representation as Perk system. **New Perk archetype: off-hand override** (Ambidextrous) — removes Light-only off-hand restriction for both weapons and focuses. **Dual-wield enhancement Perks** for both physical and magical off-hand attacks. **Cross-implement Perks** (use weapon as magical tool, focus as physical weapon). |
| [combat-ai](combat-ai.md) | AI must evaluate equipment-granted Actions through standard Gate/Scorer pipeline. **Dual-wield = one Action** (AI evaluates as single Action, not two). Off-hand bonus attack is automatic (not AI-chosen). Block chance factors into defensive calculations. AI must account for hybrid weapon+focus builds (Actions may use different implements). **Multihit**: AI should prefer Multihit when multiple enemies are in zone. |
| [consumables](consumables.md) | Equipment and consumables share the Tag vocabulary. **Thrown weapons** (future) will be consumables, not equipment. Affixes may interact with consumable effectiveness. |
| [economy](economy.md) | Equipment acquisition (loot, market), **three-tier repair** (cheap/standard/premium), **quality forging** (exponential cost, Group-gated), **enchanting** (fills empty affix slots, Group service), and **affix rerolling** (vendor-dependent, gold + materials, cursed affixes rerollable) are major gold sinks. Enemy drop quality penalty. |
| [groups](groups.md) | **Quality forging** requires specific Group service (e.g., Blacksmith's Guild). **NEW: Enchanting** service — fills empty affix slots on items. Different Groups offer different **affix reroll services** (single-affix vs. full-affix). **Source-themed affix tables** — Group vendors bias toward their theme. Different Groups offer different **repair tier access**. |
| [post-combat](post-combat.md) | Loot phase distributes equipment via event loot tables + enemy drops. Placement-proportional scaling applies. |
| [tournaments](tournaments.md) | **Equipment locked at registration** — no changes between tournament rounds. Pre-tournament loadout planning is a strategic decision. |
| [roster-management](roster-management.md) | Household stash (shared, no per-character inventory). Dead characters' gear preserved on corpse; returns to stash if permanently lost. |
| [data-model](../architecture/data-model.md) | Must support: **two implement categories** (weapons and focuses) with parallel type definitions (base stats, formula weights, tags, range; Versatile types carry two stat profiles). **Action Speed is tag-derived** (Light/Heavy/Two-Handed), not per-type. **7 focus types** with base power hierarchy. **Implement tags with scope field**: Defend/Deflect are `passive` (character-wide), Deadly/Sunder are `per_implement` (per-attack). Tags are data on implement type templates. **Artifact template overrides**: per-item override fields for range, tags, targeting, bonuses. **Implement type field** on Actions (physical/magical). Focus type definitions (base magic power, formulas, tags). **Armor type definitions**: base Soak per family, three-attribute scaling weights (Heavy: 55/30/15 End/Mig/Spd, Medium: 40/30/30 End/Spd/Awr, Light: 60/30/10 Spd/Awr/End), slot coverage, category, Speed penalty (Medium −2, Heavy −5 per piece). **Unified affix schema = Perk component schema** (Stat Adjustments, Triggers, Actions, Damage components). Continuous affix quality scaling. **Variable affix count** per item (1 to max, with empty slots trackable). Item instances (type, quality, max durability, affix list, empty slot count). Multi-slot mapping. Block chance/value on shield items (proportional split). Unique item templates. Household stash. Loot table schema (per-event, per-enemy, with affix source overrides). Enchanting uses same source-themed affix tables. **No Equipment Slot tracking** — items map to Anatomical Slots only. |

---

_Last updated: 2026-02-21 — 31 interrogation rounds, 103 decisions. Rounds 1-15: core equipment system. Rounds 16-23: MAJOR RESTRUCTURE — parallel weapon/focus categories. Round 24: FOCUS TYPE REDESIGN — 7 types. Round 25: IMPLEMENT TAGS — Defend, Deflect, Deadly, Multihit. Rounds 26-29: Finesse/Deadly split, Sunder tag, Greataxe/Crossbow/Hand Crossbow/Sling, implement tags finalized, Channeling removed. Round 30: Multihit mechanics (1–5 triangular), focus power hierarchy, ranged formulas locked, Equipment Slot limit removed, full spec restructure. Round 31: CLEANUP — Deadly/Sunder per-implement scope, tag-derived Action Speed, enchanting source-themed, no Versatile focuses, accessory degradation tracks ticks, DW penalties (−15%/−35%), armor Speed penalties (Med −2/Hvy −5), degradation ~2/fight, percentage-based range/Multihit penalties (~20%), armor attribute scaling locked (3 attributes each). Status → 🟢._
