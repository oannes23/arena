# Open Questions & Implications

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
- **Loot table contents and drop rates**: [economy](../economy.md) and [post-combat](../post-combat.md)
- **Group-specific vendor inventories, forging access, and enchanting access**: [groups](../groups.md)
- **Thrown weapons as consumables**: [consumables](../consumables.md) (future)
- **Multihit detailed combat resolution**: [combat](../combat/index.md)

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [characters](../characters.md) | **Default humanoid anatomy: 11 slots** (Head, Torso, Arms, Legs, Hands, Feet, 2x Hand, 2x Finger, Neck, Back). Attribute values gate equipment requirements. Non-humanoid ancestries can modify slot list. Combatant system Trait must provide both **Unarmed** (physical default) and **Unfocused** (magic default). **Equipment Slot concept removed** — characters equip into Anatomical Slots only (update any references to "5 Equipment Slots"). |
| [combat](../combat/index.md) | **Implement type on Actions**: Actions specify physical or magical implement type. Combat pipeline checks equipped implement via fallthrough resolution (main→off→default). **Off-hand bonus attack**: After each Action, off-hand implement fires a generic Attack of its type (physical/magical). **One adaptive Attack**: Single default Attack Action adapts to main-hand implement type. **Multihit tag**: Attack hits 1–5 enemies in zone (triangular distribution, peak 3, ~15–20% Attack Value penalty per additional hit, capped by available targets). **Defend/Deflect tags (passive)**: Must compute Physical/Magic Defense bonus from ~10% of implement's Attack Value (quality × base × formula × Perks). Character-wide while equipped. **Deadly/Sunder tags (per-implement)**: Crit chance bonus and Penetration bonus apply only to attacks made with that specific implement. **Equipment combat lock**: All fights snapshot equipment at start. **Block Chance step** in pipeline (post-hit, pre-Soak; proportional split for multi-component). **Reach/Ranged penalties**: ~20% Attack Value reduction (percentage, not flat). **Dual-wield** = one Action + off-hand bonus generic Attack, −15% main / −35% off-hand (Attack and Damage). Perks mitigate. Per-weapon/focus Trigger firing. **Versatile** types carry two stat profiles. **Armor**: hybrid Soak + Speed penalties (Medium −2/piece, Heavy −5/piece). Three-attribute scaling per category. Affix power scales continuously with quality. |
| [traits-and-perks](../traits-and-perks/index.md) | Equipment requirements are acquisition-only (gear stays after respec). Affixes can enhance specific Perks. Tags create Perk synergies. Equipment Formula Modifier Tags use same representation as Perk system. **New Perk archetype: off-hand override** (Ambidextrous) — removes Light-only off-hand restriction for both weapons and focuses. **Dual-wield enhancement Perks** for both physical and magical off-hand attacks. **Cross-implement Perks** (use weapon as magical tool, focus as physical weapon). |
| [combat-ai](../combat-ai.md) | AI must evaluate equipment-granted Actions through standard Gate/Scorer pipeline. **Dual-wield = one Action** (AI evaluates as single Action, not two). Off-hand bonus attack is automatic (not AI-chosen). Block chance factors into defensive calculations. AI must account for hybrid weapon+focus builds (Actions may use different implements). **Multihit**: AI should prefer Multihit when multiple enemies are in zone. |
| [consumables](../consumables.md) | Equipment and consumables share the Tag vocabulary. **Thrown weapons** (future) will be consumables, not equipment. Affixes may interact with consumable effectiveness. |
| [economy](../economy.md) | Equipment acquisition (loot, market), **three-tier repair** (cheap/standard/premium), **quality forging** (exponential cost, Group-gated), **enchanting** (fills empty affix slots, Group service), and **affix rerolling** (vendor-dependent, gold + materials, cursed affixes rerollable) are major gold sinks. Enemy drop quality penalty. |
| [groups](../groups.md) | **Quality forging** requires specific Group service (e.g., Blacksmith's Guild). **NEW: Enchanting** service — fills empty affix slots on items. Different Groups offer different **affix reroll services** (single-affix vs. full-affix). **Source-themed affix tables** — Group vendors bias toward their theme. Different Groups offer different **repair tier access**. |
| [post-combat](../post-combat.md) | Loot phase distributes equipment via event loot tables + enemy drops. Placement-proportional scaling applies. |
| [tournaments](../tournaments.md) | **Equipment locked at registration** — no changes between tournament rounds. Pre-tournament loadout planning is a strategic decision. |
| [roster-management](../roster-management.md) | Household stash (shared, no per-character inventory). Dead characters' gear preserved on corpse; returns to stash if permanently lost. |
| [data-model](../../architecture/data-model.md) | Must support: **two implement categories** (weapons and focuses) with parallel type definitions (base stats, formula weights, tags, range; Versatile types carry two stat profiles). **Action Speed is tag-derived** (Light/Heavy/Two-Handed), not per-type. **7 focus types** with base power hierarchy. **Implement tags with scope field**: Defend/Deflect are `passive` (character-wide), Deadly/Sunder are `per_implement` (per-attack). Tags are data on implement type templates. **Artifact template overrides**: per-item override fields for range, tags, targeting, bonuses. **Implement type field** on Actions (physical/magical). Focus type definitions (base magic power, formulas, tags). **Armor type definitions**: base Soak per family, three-attribute scaling weights (Heavy: 55/30/15 End/Mig/Spd, Medium: 40/30/30 End/Spd/Awr, Light: 60/30/10 Spd/Awr/End), slot coverage, category, Speed penalty (Medium −2, Heavy −5 per piece). **Unified affix schema = Perk component schema** (Stat Adjustments, Triggers, Actions, Damage components). Continuous affix quality scaling. **Variable affix count** per item (1 to max, with empty slots trackable). Item instances (type, quality, max durability, affix list, empty slot count). Multi-slot mapping. Block chance/value on shield items (proportional split). Unique item templates. Household stash. Loot table schema (per-event, per-enemy, with affix source overrides). Enchanting uses same source-themed affix tables. **No Equipment Slot tracking** — items map to Anatomical Slots only. |
