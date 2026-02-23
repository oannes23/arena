# Generation

## Species / Ancestry Core Traits

Species are defined entirely via Core Traits — there is no hardcoded species list in the system. A character's biology, anatomy, and innate nature emerge from whichever ancestry Core Traits they possess.

### Anatomy Modification

- Ancestry Core Traits can modify Anatomical Slots: add slots, remove slots, or change slot counts.
- Multiple ancestry Core Traits are allowed and stackable (e.g., "Orc" + "Fey-Touched").
- **Conflict resolution**: Per-slot maximum — take the highest count from any ancestry Trait. Example: if one Trait gives 4 Hand slots and another gives 2, the character has 4.
- A character with 0 ancestry Core Traits gets default humanoid anatomy (as defined in [characters](../characters.md)).

### Personality Archetype Tags on Core Traits

Core Traits that represent personality or temperament can be tagged with one or more **personality archetypes**. These tags feed the combat AI's Personality Score track (see [combat-ai](../combat-ai.md)), influencing Action selection based on character temperament.

| Archetype | AI Bias | Example Core Trait |
|-----------|---------|-------------------|
| **Aggressive** | Offensive, damage-dealing Actions | Berserker's Fury |
| **Cautious** | Defensive, positioning, evasive Actions | Vigilant Mind |
| **Protective** | Ally-supporting Actions (heals, buffs, shields) | Guardian's Oath |
| **Vindictive** | Prioritize whoever last hurt self or allies | Vengeful Spirit |
| **Showoff** | Flashy, high-impact, expensive Actions | Born Performer |

A character can have multiple personality archetypes from multiple Core Traits. Characters with no personality-tagged Core Traits have neutral AI personality (purely tactical decisions, modulated by Judgment). The Judgment derived stat (see [combat](../combat/index.md)) controls how much personality influences decisions vs tactical optimality.

Personality archetypes are a content-level tag on Core Trait definitions — not a new component type. New archetypes can be added by extending the vocabulary in the combat-ai spec.

### Design Intent

This pattern means:
- New species are added by creating new Core Traits — no schema changes needed.
- Hybrid/mixed ancestry is naturally supported by stacking multiple ancestry Traits.
- Not all Core Traits are ancestry — personality Traits (Berserker's Fury) and destiny Traits (Fated Hero) are also Core but don't modify anatomy.

---

## Tag System

Tags are a shared keyword vocabulary connecting Perks, Equipment, and Consumables. They enable emergent synergies across the build system.

### Where Tags Live

- Tags live on individual **Perks** and their component **Actions**.
- **Traits derive their tags** from the union of all their Perks' tags — Traits have no independent tags.
- **Auto-tags from resources**: Perks that grant or cost a resource type automatically gain the corresponding resource tag (e.g., Mana → Arcane, Faith → Divine). This is in addition to any manually-assigned tags.
- Equipment and Consumables also carry tags (defined in their respective specs).

### How Tags Create Synergies

Tags enable emergent interactions rather than explicit set bonuses:
- A Perk might grant "+10% damage to all [Fire]-tagged Actions" — any [Fire] Action benefits, regardless of source Trait.
- Equipment might grant "+5 Soak vs. [Poison]-tagged damage" — works against any [Poison] source.
- A consumable tagged [Healing] might interact with a Perk that says "when you use a [Healing] effect, also gain 5 Stamina."

### No Explicit Set Bonuses

There are no "if you have Trait X and Trait Y, gain bonus Z" mechanics. All cross-Trait synergies are tag-driven and emergent. This keeps the system open-ended — new Traits automatically interact with existing ones through shared tags.

---

## Trait Generation Loot Tables

When a character is generated at recruitment, their starting Traits are determined by loot table rolls (see [characters](../characters.md) for the roll mechanics). This section defines the loot table structure.

### Table Structure

- **Weighted probability pools** with nestable subtables.
- Top-level table can branch into subtables (e.g., Ancestry 20%, Personality 30%, Gift 30%, Background 20%).
- Tables are nestable — one archetype table can include another archetype's table as a subtable outcome.
- **Per-archetype definition**: Each archetype's loot table lives with the Group that offers recruitment (defined in the [groups](../groups.md) spec).

### Starting Level

- Generated Traits start at their minimum required star level. A min-3★ Trait on a newly generated character starts at 3★ (with cumulative XP cost already "paid" by generation).

### Roll Modifiers

- **Per-star-level roll overrides**: Different probability distributions for the 1st roll vs. 2nd roll vs. 3rd roll within a category. Example: 1st Core roll weighted toward ancestry; 2nd weighted toward personality; 3rd toward gifts.
- **Openness parameter**: A stacking percentage chance per star level to zero out a slot before rolling on the table.
  - Low openness = veteran archetype (most slots pre-filled with Traits).
  - High openness = open-potential archetype (many empty slots for the player to fill later).
  - Example: 15% openness means each roll has a 15% chance of becoming empty. For a 3★ character, this applies independently to each of the 3 rolls.
- **Duplicate handling**: Only discard if the exact same Trait is rolled again (same-category, different-Trait is always fine). Duplicate Trait rolls unlock a random Perk from that Trait's tree instead (see [characters](../characters.md)).

---

## Perk Discovery

A rare mechanic that rewards active use of Traits in combat.

### Discovery Model — Per-Trait End-of-Combat Roll

- **Timing**: Discovery is checked once per qualifying Trait at the **end of combat**, not during combat. This is a single roll per Trait, not per-Action-use or per-Trigger-fire.
- **Active Traits only**: Only Traits that had at least one Action used or Trigger fire during the combat qualify for a discovery roll. Traits where only passive Stat Adjustments were active do NOT qualify. The [Combat Scoreboard](../combat/index.md#combat-scoreboard) determines which Traits were active.
- **One roll per Trait**: Each qualifying Trait gets exactly one discovery roll. This replaces the old per-component-use model, eliminating bias toward high-Action-count builds.
- **Rarity weighting**: When a discovery succeeds, the specific Perk discovered is weighted by rarity — lower-minimum-star Perks are more likely to be discovered than higher-minimum-star ones.
- **Full tree**: If all Perks in the Trait's tree are already owned, the roll is silently skipped (no discovery possible).
- Even rare 5★ Perks can be discovered this way — though they are much less likely due to rarity weighting.

### Discovery Resolution (Post-Combat)

Discovery rolls are resolved during the [post-combat](../post-combat.md) Perk Discovery phase:

- The discovered Perk is presented to the player, who can **accept or reject** it.
- **Accept**: The Perk is acquired at its minimum star level, free of XP cost.
- **Reject**: The Perk is not acquired. It remains in the discovery pool for future rolls. No penalty for rejection.
- **Level cap override**: Accepted discovered Perks override the Perk level cap at the moment of acquisition — they are acquired at their minimum star level even if it exceeds the parent Trait level. However, further manual leveling of the discovered Perk is still capped by the parent Trait level.
- Multiple discoveries can occur in a single combat (from different Traits). Each is presented individually for accept/reject.

See [post-combat](../post-combat.md) for the full resolution flow.

### Design Intent

- End-of-combat per-Trait rolls are simpler to implement and fairer across builds than the old per-use model.
- Post-combat resolution avoids interrupting combat flow and gives the player time to consider.
- Accept/reject adds meaningful player agency to the random system — especially important now that Perks can have tradeoffs (negative Stat Adjustments or harmful Triggers).
- Rejection keeps the Perk in the pool, so players can't "clear out" undesirable Perks by rejecting them permanently — they'll keep appearing until the tree is full.
- Proactively buying desired Perks via Trainers shrinks the discovery pool, reducing the chance of discovering unwanted tradeoff Perks.
- The "active Traits only" rule replaces the old "passive Stat Adjustments don't trigger discovery" rule — same intent (reward active combat participation), cleaner implementation.

**Trait discovery** (acquiring entirely new Traits via combat, rather than just new Perks from existing Traits) is deferred to a later phase.
