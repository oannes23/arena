# Inventory & Loot

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
- **Placement scaling**: Per [post-combat](../post-combat.md), loot drop rates scale with placement (1st=100%, 2nd=60%, etc.)

### Enemy Equipment Drops

- Small chance per defeated enemy to drop one equipped item
- Dropped items have a **quality penalty** (e.g., −20% from original quality)
- Affixes preserved from the enemy's item
- Only applies to enemies that had visible equipment (not all Ephemeral Combatants)
