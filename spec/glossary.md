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
A numeric stat (0–200 scale) defining a character's capabilities. There are 9 primary Attributes: Might, Speed, Accuracy, Endurance, Charisma, Awareness, Intellect, Willpower, Luck. Each Attribute has a Current value (used in play) and a Potential value (maximum trainable cap).

### Current (Attribute)
The actual stat value used in combat and checks. Range 0–200. Starts below Potential; increased by spending XP (cost per +1 = current value). Average ~50 for a starting character.

### Potential (Attribute)
The maximum trainable value for an Attribute. Range 0–200. Cannot be exceeded by training. Average ~100 for a starting character. Loose inverse correlation with Current — lower Current often correlates with higher Potential (growth opportunity vs. ready-made veteran). Unicorn characters (high both) can appear rarely. Mostly fixed at generation, but can change: Promotion (+10% all Potentials), rare events (small increases), injuries/death (permanent reduction).

### Might
Primary Attribute. Melee damage, equipment requirements (heavy), forced movement, physical intimidation.

### Speed
Primary Attribute. Initiative gain (primary), dodge/evasion, movement-related perks, Action Speed modifier.

### Accuracy
Primary Attribute. Attack rolls (hit chance), ranged damage scaling, targeting quality.

### Endurance
Primary Attribute. Health pool (primary), Stamina pool (co-primary), physical damage soak, attrition resistance.

### Charisma
Primary Attribute. Morale influence, vendor pricing, crowd management, crowd appeal, leadership aura effects.

### Awareness
Primary Attribute. Detection (anti-stealth), trap awareness, Initiative gain (minor), tie-breaking, critical hit chance.

### Intellect
Primary Attribute. Spell power, crafting quality, XP efficiency, Mana pool size.

### Willpower
Primary Attribute. Status resistance, mental damage soak, Focus pool, Stamina pool (co-primary), concentration/anti-interrupt.

### Luck
Primary Attribute. Contributes to critical hit chance (with Awareness, Accuracy), increases all resistance rolls, provides bonuses on loot table rolls, and affects crowd favor decisions.

### Multi-Attribute Blending
Design principle where derived stats are computed from weighted combinations of multiple primary attributes, preventing dump stats and making diverse builds viable.

### Archetype (Generation)
A Group-specific internal character generation template that sets base attribute distributions (bumps, dumps, likely trait pool, equipment themes). Each Group that offers recruitment defines its own themed archetypes — there is no universal archetype list. Not visible to players — they see resulting stats, not the template label.

### Promotion (Character)
An extremely expensive metacurrency-only process to increase a character's Star Rating by 1. Grants +10% to all Potentials and +1 Trait slot per category. Rewards deep investment in a specific character.

### Career Milestone
A tracked threshold of fights/tournaments participated in. Enables optional retirement for metacurrency bonuses.

### Crowd Appeal
A derived stat. 60% Charisma + 25% Luck + 15% Awareness. Determines how much crowd excitement/momentum a character generates during fights.

### Bonus Modifier
A tracked attribute bonus from Perks, Equipment, or Status Effects. Separate from base Current — removing the source removes only its bonus. Stacks additively within each source type; all source types combine additively.

### Effective Attribute
The value used in all derived stat formulas: base Current + all Bonus Modifiers (from Perks, Equipment, Status Effects). Not capped by Potential — Potential only gates training. Hard ceiling is the scale max (200).

### Exhaustion (0 Stamina)
When Stamina reaches 0, physical actions cost Health instead. A last-stand mechanic that creates dramatic late-fight moments without completely disabling the character.

### Fallen
In-Combat sub-state. A character whose HP dropped below 1 during combat. Out of the fight for its remainder. Post-combat fate depends on event type: exhibitions recover normally (no injury risk), non-exhibition fights trigger an injury/death roll after combat resolves (outcomes: Available, Recovering, or Dead). See [characters.md](domains/characters.md) for the full state machine.

### Recovering
Character state indicating injuries with stat penalties. The character CAN still fight (at the player's risk, with penalties applied). Recovers passively over ticks (slow) or via healing services (fast/instant). See [characters.md](domains/characters.md) for the full state machine.

### Retired
Terminal character state. The character persists in a "hall of fame" — viewable but permanently inactive. Grants metacurrency rewards based on career milestones. Cannot be reversed.

### Magic Defense
A derived stat. Resistance to magical attacks. 60% Willpower + 25% Awareness + 15% Luck.

### Anatomical Slot
A body location where equipment can be worn (Head, Torso, Hands, etc.). Slot availability varies by ancestry/biology. Distinct from Equipment Slot (the 5-item limit).

### Ephemeral Combatant
An unnamed battle opponent generated for a specific combat encounter (e.g., "5× 1★ Bandits"). Uses the same attribute and derived stat model as Characters but is not persistent — ceases to exist after combat resolves. No state machine, identity fields, or career tracking.

### Named NPC
A persistent non-player Character entity tracked identically to player-owned characters — same attributes, state machine, identity, career history, and Trait slots. Generated via post-battle recruitment mechanics or as Group members. Can be recruited by players, become Free Agents, or serve in Groups.

### Free Agent
A Named NPC not currently owned by any player or attached to any Group. Part of a persistent pool that receives turnover from unrecruited post-battle NPCs and Group departures. Recruitable by players. Defined in downstream specs ([groups](domains/groups.md), [roster-management](domains/roster-management.md)).

---

## Groups

### Group
A named organization in the city — guild, temple, tavern, faction, mercenary company, arena office, monastery, etc. The unified entity type behind vendors, recruiters, trainers, and factions. Each Group offers a service menu with public services (available to anyone) and member services (unlocked by holding the corresponding Bond Trait). See [groups.md](domains/groups.md).

### Service (Group)
A specific offering provided by a Group. Service types include: vendor (buy/sell), training (attributes/traits/perks), recruitment (hire characters via Group-specific archetypes), spellcasting, crafting, and quests (future). A single Group can offer multiple service types.

---

## Traits & Perks

### Trait
A core identity building block attached to a Character via a Trait Slot. Each Trait belongs to one of three categories (Core, Role, Bond) and unlocks a tree of Perks. Has a minimum star level for purchase; levels up to 5★.

### Core Trait
Trait category representing innate character identity — ancestry, personality, unusual gifts/destinies. Examples: Orc, Berserker's Fury, Fated Hero.

### Role Trait
Trait category representing functional capabilities, both combat and support. Examples: Warrior, Pyromancer, Blacksmith, Medic. Characters can fight AND provide household support if their Role Trait Perks allow it.

### Bond Trait
Trait category representing Group membership. A Bond Trait connects a character to a specific Group, unlocking that Group's member services (exclusive inventory, better prices, specialized training, recruitment, etc.). Examples: Pyromancer's Guild Member, Priest of Tharzul, Nanna-Sin Monastery Disciple.

### Trait Slot
A slot on a Character for holding one Trait. Characters have slots in each of three categories (Core, Role, Bond); count equals their Star Rating (1–5 per category). Each slot holds one Trait of any star level.

### Perk
The atomic unit of character capability, unlocked by owning a parent Trait. A multi-component package that can include any combination of: Actions, Stat Adjustments, and Triggers. Has a minimum star level; levels to 5★. A Perk's level cannot exceed its parent Trait's level (exception: Perks acquired via Perk Discovery override this cap).

### Action (Perk Component)
An active combat ability granted by a Perk. Used on the character's turn during combat. Has an Action Speed, optional cooldown, and optional resource cost.

### Stat Adjustment (Perk Component)
A permanent attribute bonus applied while the Perk is owned. Passive — always active.

### Trigger (Perk Component)
A conditional automatic effect from a Perk that fires when specific combat events occur. Not activated by the player or AI — resolves automatically.

### Respec
An expensive vendor service (from a Group) to remove an entire Trait from a character. All XP invested in the removed Trait and its Perks is lost. Operates at the Trait level only — there is no individual Perk removal. Intentionally costly to encourage roster turnover over endlessly iterating a single character.

### Starter Perk
The designated Perk auto-granted (free, no XP cost) when a Trait is acquired. Every Trait has exactly one Starter Perk — it defines the Trait's basic capability and ensures immediate usefulness.

### Perk Discovery
A rare chance (~0.1%) when using a Perk's Action or when a Perk's Trigger fires in combat to instantly discover and unlock a random unowned Perk from the same Trait's tree. The discovery roll is rarity-weighted (lower-minimum-star Perks more likely). The discovered Perk is acquired at its minimum star level, free of XP cost, and overrides the Perk level cap. Passive Stat Adjustments do not trigger discovery.

### Resource Family
A category of non-universal resource pool unlocked by owning Traits in that family. Five families exist: Arcane (Mana), Divine (Faith), Primal (Spirit), Psychic (Focus), Martial (Stamina). Each family has a named resource and an attribute-derived pool formula. Each Trait belongs to at most one family; multiple Traits in the same family share one pool.

### Mana
Arcane family resource pool. Formula: 60% Intellect + 25% Willpower + 15% Awareness (× scaling multiplier). Granted by owning any Arcane-family Trait.

### Faith
Divine family resource pool. Formula: 60% Willpower + 25% Charisma + 15% Luck (× scaling multiplier). Granted by owning any Divine-family Trait.

### Spirit
Primal family resource pool. Formula: 50% Endurance + 30% Awareness + 20% Willpower (× scaling multiplier). Granted by owning any Primal-family Trait.

### Focus
Psychic family resource pool. Formula: 60% Awareness + 30% Willpower + 10% Endurance (× scaling multiplier). Granted by owning any Psychic-family Trait.

### Openness (Archetype)
A generation parameter on an archetype's Trait loot table. A stacking percentage chance per star level to zero out a Trait slot before rolling on the generation table. Low openness produces veteran archetypes (most slots pre-filled); high openness produces open-potential archetypes (many empty slots for the player to fill).

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
A pool spent to use certain Actions. Default resources (all characters): Health, Stamina. Trait-defined resources belong to five Resource Families: Mana (Arcane), Faith (Divine), Spirit (Primal), Focus (Psychic), and Stamina bonus (Martial). See Resource Family.

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
A shared mechanical keyword used by Perks, equipment, and consumables to create emergent synergies. Tags live on individual Perks and their component Actions; Traits derive their tags from the union of all their Perks' tags (no independent Trait tags). Tags can stack on a single item. Examples: Fire, Healing, Reach, Two-Handed, Light, Heavy, Finesse, Ranged.

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
A pre-combat purchase from a Group's service menu (not carried into battle). Cheaper than scrolls, no Perk requirements. One-time service effects like temporary buffs or pre-fight healing.

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

_Last updated: 2026-02-14 — Updated Perk (level cap), Perk Discovery (triggers, rarity weighting, cap override), Respec (Trait-level only)_
