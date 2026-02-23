# Armor System

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
