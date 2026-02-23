# Quality & Affixes

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
