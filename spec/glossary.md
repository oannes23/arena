# Glossary

Canonical definitions for domain terms used in this project. When a term appears in specs, it means exactly what's defined here.

---

## How to Use This Glossary

- **When reading specs**: If a term seems unclear, check here first
- **When writing specs**: Use terms exactly as defined; add new terms as needed
- **During interrogation**: The agent will add terms that emerge from discussion

---

## Characters & Attributes

### Character
A fighter or support staff member in the player's household. Has Attributes, Traits, Perks, equipment, and a Star Rating.

### Star Rating
A 0–5★ universal quality/power scale applied to Characters, Traits, Perks, and Equipment. See [overview.md](architecture/overview.md) for the full table. For Characters, determines Trait slot capacity.

### Attribute
A numeric stat defining a character's capabilities. Each Attribute has a Current value (used in play) and a Potential value (maximum trainable cap).

### Current (Attribute)
The actual stat value used in combat and checks. Starts below Potential; increased by spending XP. Average ~50 for a starting character.

### Potential (Attribute)
The maximum trainable value for an Attribute. Cannot be exceeded by training. Average ~100 for a starting character. Lower Current often correlates with higher Potential (growth opportunity vs. ready-made veteran).

### Strength
Primary Attribute. Physical power, melee damage, carry capacity, heavy equipment requirements.

### Agility
Primary Attribute. Speed, dodge, accuracy, light/finesse weapon scaling.

### Willpower
Primary Attribute. Mental resistance, spell power, resource pools, status resistance (Stun, Psychic, etc.).

### Awareness
Primary Attribute. Detection (stealth counterplay), trap awareness, initiative tie-breaking, targeting intelligence.

### Anatomical Slot
A body location where equipment can be worn (Head, Torso, Hands, etc.). Slot availability varies by ancestry/biology. Distinct from Equipment Slot (the 5-item limit).

---

## Traits & Perks

### Trait
A core identity building block attached to a Character via a Trait Slot. Each Trait belongs to one of three categories (Core, Role, Bond) and unlocks a tree of Perks. Has a minimum star level for purchase; levels up to 5★.

### Core Trait
Trait category representing innate character identity — ancestry, personality, unusual gifts/destinies. Examples: Orc, Berserker's Fury, Fated Hero.

### Role Trait
Trait category representing functional capabilities, both combat and support. Examples: Warrior, Pyromancer, Blacksmith, Medic. Characters can fight AND provide household support if their Role Trait Perks allow it.

### Bond Trait
Trait category representing organizational affiliations. Unlocks vendor access, themed content, and pricing benefits. Examples: Pyromancer's Guild Member, Priest of Tharzul, Nanna-Sin Monastery Disciple.

### Trait Slot
A slot on a Character for holding one Trait. Characters have slots in each of three categories (Core, Role, Bond); count equals their Star Rating (1–5 per category). Each slot holds one Trait of any star level.

### Perk
The atomic unit of character capability, unlocked by owning a parent Trait. A multi-component package that can include any combination of: Actions, Stat Adjustments, and Triggers. Has a minimum star level; levels to 5★.

### Action (Perk Component)
An active combat ability granted by a Perk. Used on the character's turn during combat. Has an Action Speed, optional cooldown, and optional resource cost.

### Stat Adjustment (Perk Component)
A permanent attribute bonus applied while the Perk is owned. Passive — always active.

### Trigger (Perk Component)
A conditional automatic effect from a Perk that fires when specific combat events occur. Not activated by the player or AI — resolves automatically.

### Respec
An expensive vendor service to remove a Trait from a character. Intentionally costly to encourage roster turnover over endlessly iterating a single character.

---

## Combat

### Zone
A spatial region on the battlefield. Default map: 5 zones (North, South, East, West, Center). Characters occupy a zone and can interact with others based on range.

### Range
Distance classification between zones. **Short**: same zone (melee). **Medium**: adjacent zones. **Long**: non-adjacent zones.

### Initiative
A meter that accumulates per tick based on Speed. When Initiative ≥ 100, the character takes a turn. After acting, Initiative is reduced (base −100, modified by Action Speed).

### Tick (Combat)
A single time step in combat resolution. Each tick, all characters add `sqrt(Speed)` to their Initiative meters and any per-tick effects (status decay, DoTs) resolve.

### Action Speed
A property of an Action (Perk component) that modifies how much Initiative is subtracted after use. Fast actions let a character act again sooner; slow actions impose a longer delay.

### Roll-vs-Static Defense
The combat resolution model. Attacker rolls 1 to [Attack Value]; defender's Defense is subtracted as a flat threshold. Negative result = miss. Non-negative result = hit bonus that carries into the damage step.

### Soak
Armor/resistance value used in the damage step. Damage is rolled 1 to [Damage Value]; Soak is subtracted. Negative result = fully absorbed.

### Rider Effect
A modular effect (from a Perk or equipment affix) that triggers after a successful hit. Each rider has its own resolution roll. Examples: Stunning Blow, Flaming Sword proc, Armor Penetration.

### Status Effect
A temporary condition applied to a character, tracked as stacks with a decay model. Unified system for buffs, debuffs, DoTs, and crowd control. Max stacks effectively unlimited (9999 cap).

### Sneaking
A status that prevents a character from being targeted by enemies. Enemies must pass a contested Awareness check to detect a Sneaking character. Implemented within the standard stack-based status system.

### Morale
A fighter stat affected by combat events (ally death, kills, heavy damage). May cause debuffs, fleeing, or bonuses. Full design TBD (Phase 4).

### Defend
A basic action available to all characters. Increases defensive stats temporarily until the character's next turn.

### Damage Type
A classification for damage. 10 types in three families: Physical (Blunt, Piercing, Slashing), Elemental (Fire, Cold, Lightning), Magical (Poison, Shadow, Light, Psychic). No inherent rock-paper-scissors — differences come from affiliated status effects and Perk/equipment interactions.

### Resource
A pool spent to use certain Actions. Default resources (all characters): Health, Stamina. Trait-defined resources: Mana (elemental magic), Faith (divine), others per Trait family.

---

## Equipment

### Equipment Slot
One of a character's 5 item slots. Each equipped item occupies exactly 1 Equipment Slot regardless of how many Anatomical Slots it covers.

### Multi-Slot Item
An item that occupies multiple Anatomical Slots but counts as only 1 Equipment Slot. Example: Plate Armor covers Head + Torso + Arms + Legs. Higher base stats but same affix budget as single-slot equivalents.

### Quality
A 0–500+ numeric scale for equipment effectiveness. Acts as a percentage modifier to base stats (100 = 1.0× baseline). Maps to Star Rating tiers: 0–99 = 0★, 100 = 1★, 200 = 2★, 300 = 3★, 400 = 4★, 500+ = 5★.

### Durability
Equipment's maximum quality ceiling. Current quality drops during combat use; repair restores current to (max − 1), permanently reducing max by 1. Long-term gear degradation.

### Affix
A modifier on equipment (Diablo-style prefix/suffix). Provides attribute bonuses, damage type bonuses, special effects, or Perk enhancements. Budget scales inversely with tier: fewer affixes = stronger each.

### Tag
A shared mechanical keyword used by weapons, armor, consumables, and Perks to create synergies. Tags can stack on a single item. Examples: Reach, Two-Handed, Light, Heavy, Finesse, Ranged.

---

## Consumables

### Consumable Slot
One of 5 slots a character can fill with consumables before battle. Separate from Equipment Slots.

### Potion
A consumable type. Fast use (minimal Initiative penalty), guaranteed success, moderate power cap. Recipes define effects.

### Bomb
A consumable type. Fast use like potions, offensive effects (damage, status application, AoE). Tags define damage types and targeting.

### Scroll
A consumable type. Slow use (high Initiative penalty), spell failure chance (requires check), much cheaper than potions for equivalent effect, no power ceiling. Created by characters with the Scribe Perk.

### Vendor Service
A pre-combat purchase (not carried into battle). Cheaper than scrolls, no Perk requirements. One-time service effects like temporary buffs or pre-fight healing.

---

## Economy & Time

### Tick (Game Time)
A discrete update cycle where passive systems resolve. Single-player: advances on player command. Multiplayer: real-time on server schedule. Resolves: passive income, vendor turnover, natural healing, upkeep, tournament scheduling.

### Upkeep
An ongoing gold cost per character deducted each tick. Creates economic pressure — over-hiring leads to bankruptcy.

### Support Staff
Characters with support Role Traits who provide passive household bonuses (reduced repair costs, XP discounts, gold generation, etc.) while potentially also fighting in combat.

### Metacurrency
Persistent account-level currency earned through achievements, milestones, tournament victories, and character retirement. Spent on Backgrounds and premium character generation options.

### Background
A starter template purchasable with Metacurrency. Provides pre-configured Trait packages for new recruits. Examples: "Orc Warrior," "Noble Lineage," "Battle-Scarred Veteran."

---

## Tournaments & Encounters

### Exhibition Tournament
Frequent competitive event with no/low injury risk and smaller rewards. Used for testing builds and grinding XP.

### Championship Tournament
Infrequent competitive event with full injury risk and significant rewards (gold, loot, metacurrency, prestige). Elimination or Swiss-bracket format.

### NPC Backfill
When a tournament fires, unfilled participant slots are filled with procedurally generated NPC teams.

### PvE Encounter
An on-demand combat encounter available between ticks for grinding. Set difficulty tiers, reduced injury risk, rewards loot/XP/gold. Predefined stages with themed opponents.

---

## Roster

### Injury
A persistent negative condition resulting from defeat in combat. Types: stat penalties, lost limbs (permanent anatomical slot loss), time-limited debuffs. Severity determined by injury resistance check.

### Raise Dead
A temple vendor service to resurrect a dead character. Three cost tiers with decreasing attribute penalties, from cheapest (halve all Current, −1 all Potentials) to most expensive (no permanent penalties).

### Base Building
A future system for facility upgrades (barracks, training grounds, infirmary, forge). Provides bonuses to roster size, training efficiency, healing speed, etc. Design TBD.

---

## Abbreviations

| Abbrev | Expansion |
|--------|-----------|
| MVP    | Minimum Viable Product |
| TBD    | To Be Determined |
| AI     | Artificial Intelligence (in-game combat AI, not the development tool) |
| AoE    | Area of Effect |
| DoT    | Damage over Time |
| HP     | Health Points |
| XP     | Experience Points |
| TUI    | Terminal User Interface |
| PvE    | Player versus Environment |
| PvP    | Player versus Player |
| NPC    | Non-Player Character |

---

_Last updated: 2026-02-11_
