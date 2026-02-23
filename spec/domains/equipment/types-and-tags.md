# Types & Tags

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
