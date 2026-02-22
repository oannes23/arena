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
When Stamina reaches 0, physical Actions cost Health at a **1:1 ratio** (no penalty multiplier). A 30-Stamina Action costs 30 Health instead. **Only Stamina** has this Health substitution mechanic — all other resource types (Mana, Faith, Spirit, Focus, etc.) hard-gate their Actions via `resource_gate` when depleted (Actions become unavailable, no fallback). A last-stand mechanic that creates dramatic late-fight moments without completely disabling the character.

### Fallen
In-Combat sub-state. Determined by a **pure HP check at tick end** (step 4 of tick resolution) — if HP < 1 at that point, the character enters Fallen. **Healing saves**: if a character was reduced below 0 HP but healed above 0 before step 4, they do not enter Fallen. Characters enter Fallen in **reverse Initiative order** (lowest first); `OnFallen` Triggers fire sequentially. Status effects are **frozen while Fallen** (no tick-down). Revival clears all statuses (**clean slate**). Post-combat fate depends on event type: exhibitions recover normally (no injury risk), non-exhibition fights trigger an injury/death roll after combat resolves (outcomes: Available, Recovering, or Dead). See [characters.md](domains/characters.md) for the full state machine.

### Recovering
Character state indicating injuries with stat penalties. The character CAN still fight (at the player's risk, with penalties applied). Recovers passively over ticks (slow) or via healing services (fast/instant). See [characters.md](domains/characters.md) for the full state machine.

### Retired
Terminal character state. The character persists in a "hall of fame" — viewable but permanently inactive. Grants metacurrency rewards based on career milestones. Cannot be reversed.

### Magic Defense
A derived stat. Resistance to magical attacks. 60% Willpower + 25% Awareness + 15% Luck.

### Anatomical Slot
A body location where equipment can be worn (Head, Torso, Hands, etc.). Slot availability varies by ancestry/biology. Distinct from Equipment Slot (the 5-item limit).

### Ephemeral Combatant
An unnamed battle entity generated for a specific combat encounter (e.g., "5× 1★ Bandits") or created mid-combat by the Summon effect component. Uses the same attribute and derived stat model as Characters but is not persistent — ceases to exist after combat resolves. No state machine, identity fields, or career tracking. Summoned Ephemeral Combatants have their own Initiative, preset AI, and team membership matching the summoner; they disappear when killed or when the summoner enters Fallen. They do not trigger post-combat phases (injury, discovery, recruitment). Summoners are limited by a **Summon Control Capacity** budget — a Resource Pool that caps total active summons by cost. If a summoner is revived, their summons remain dead and must be re-summoned.

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
Trait category representing Group membership. A Bond Trait connects a character to a specific Group, unlocking that Group's member services (exclusive inventory, better prices, specialized training, recruitment, etc.). One Bond Trait per Group per character — the Bond levels (1–5★) to deepen the relationship. Examples: Pyromancer's Guild Member, Priest of Tharzul, Nanna-Sin Monastery Disciple.

### Trait Slot
A slot on a Character for holding one Trait. Characters have slots in each of three categories (Core, Role, Bond); count equals their Star Rating (1–5 per category). Each slot holds one Trait of any star level.

### Perk
The atomic unit of character capability, unlocked by owning a parent Trait. A multi-component package that can include any combination of: Actions, Stat Adjustments, and Triggers (no hard component count limit). Has a minimum star level; levels to 5★ using the same scaling curve as Trait amplification (×1.0/×1.2/×1.4/×1.7/×2.0), stacking multiplicatively with Trait level. A Perk's level cannot exceed its parent Trait's level (exception: Perks acquired via Perk Discovery override this cap at acquisition; further leveling remains capped). When the same Perk is owned from two Traits, the higher Trait's amplification applies.

### Action (Perk Component)
An active combat ability granted by a Perk. Used on the character's turn during combat. Has an Action Speed, optional cooldown, optional resource cost, and tag-based targeting (e.g., `[Enemy, Single]`, `[Ally, AoE-Zone]`). All cooldowns reset fully between fights (per-combat only — no cross-fight persistence). Outputs are expressed as an effect component list — an ordered list of typed effect components (Damage, Heal, ApplyStatus, etc.) that resolve simultaneously. See [traits-and-perks.md](domains/traits-and-perks.md) for the full model.

### Stat Adjustment (Perk Component)
A permanent bonus applied while the Perk is owned. Passive — always active. Three subtypes: **flat bonuses** (direct numeric bonus to any stat — Attributes, derived stats, resource pools, crit chance, etc.), **tag-scoped bonuses** (conditional bonus when a matching tag is present — can be flat or percentage, e.g., "+5 Soak vs [Poison]" or "+10% damage to [Fire] Actions"), and **formula modifier tags** (attribute substitution in formula evaluation, e.g., "use Intellect instead of Accuracy in Attack formulas" — highest value wins on conflict). Multiple percentage bonuses on the same stat/tag stack additively. Application order: flat bonuses first, then combined percentage as a single multiplier on the already-level-scaled value.

### Trigger (Perk Component)
A conditional automatic effect from a Perk that fires when specific combat events occur (e.g., OnHit, OnHitBy, OnAllyFallen, OnTurnStart). Not activated by the player or AI — resolves automatically. Uses the same effect component list as Actions for its outputs. The canonical Trigger event vocabulary is owned by the combat spec.

### Effect Component
A typed unit of effect output used by Actions and Triggers. Each component has a type tag and key-value parameters. Core types (illustrative): Damage, Heal, ApplyStatus, RemoveStatus, Move, Summon, Shield, Revive. The **Revive** component works on any ally target — on non-Fallen targets it does nothing, but other components in the Action still resolve (enabling hybrid heal+revive Actions). All effect components within a single Action or Trigger resolve simultaneously. Numeric output values scale with Perk/Trait level multipliers; costs and cooldowns stay flat. The full type catalog is owned by the combat spec.

### Target Tag
A tag on an Action specifying what it can target. Examples: `[Self]`, `[Enemy, Single]`, `[Ally, AoE-Zone]`. The combat spec defines how target tags resolve to valid battlefield targets. New targeting modes are new tags — no schema changes required.

### Respec
An expensive vendor service (from a Group) to remove an entire Trait from a character. All XP invested in the removed Trait and its Perks is lost. Operates at the Trait level only — there is no individual Perk removal. Intentionally costly to encourage roster turnover over endlessly iterating a single character.

### Starter Perk
The designated Perk auto-granted (free, no XP cost) when a Trait is acquired. Every Trait has exactly one Starter Perk — it defines the Trait's basic capability and ensures immediate usefulness.

### Perk Discovery
A rare end-of-combat mechanic. At the end of combat, each **active Trait** (Traits that had at least one Action used or Trigger fire) gets a single discovery roll to find a random unowned Perk from that Trait's tree. Formula: `(Base Rate + Luck Bonus + Perk Bonuses) × Trait Level Multiplier` — where Trait Level Multiplier uses the standard amplification curve (×1.0/×1.2/×1.4/×1.7/×2.0). One roll per qualifying Trait, not per-use. No hard rate cap — small tree size (2-4 discoverable Perks) is the natural limit. **Resolved in [post-combat](domains/post-combat.md)**: the player can **accept or reject** each discovered Perk; rejected Perks remain in the pool for future rolls. Accepted Perks are acquired at minimum star level, free of XP cost, and override the Perk level cap at acquisition (further leveling capped by Trait level). Rarity-weighted (lower-minimum-star more likely). Traits with only passive Stat Adjustments active do not qualify. Full tree = silently skipped.

### Resource Pool
A non-universal resource granted to characters through Perk content. A resource pool activates when a character first owns a Perk that references it (costs it or grants pool capacity). The attribute-derived formula provides the base pool size; Perk Stat Adjustments add bonus capacity **pre-multiplier** (bonus capacity adds to the weighted attribute blend before the per-stat scaling multiplier is applied, meaning it is amplified by the multiplier). Five default resource types ship (Mana, Faith, Spirit, Focus, Stamina); content authors can define additional types. Resource pool ownership is emergent from Perk content, not a Trait-level property — a single Trait can have Perks referencing multiple resource types. See [traits-and-perks.md](domains/traits-and-perks.md).

### Mana
Default resource pool (Arcane auto-tag). Formula: 60% Intellect + 25% Willpower + 15% Awareness (× scaling multiplier). Activates when a character owns any Perk that costs or grants Mana.

### Faith
Default resource pool (Divine auto-tag). Formula: 60% Willpower + 25% Charisma + 15% Luck (× scaling multiplier). Activates when a character owns any Perk that costs or grants Faith.

### Spirit
Default resource pool (Primal auto-tag). Formula: 50% Endurance + 30% Awareness + 20% Willpower (× scaling multiplier). Activates when a character owns any Perk that costs or grants Spirit.

### Focus
Default resource pool (Psychic auto-tag). Formula: 60% Awareness + 30% Willpower + 10% Endurance (× scaling multiplier). Activates when a character owns any Perk that costs or grants Focus.

### Openness (Archetype)
A generation parameter on an archetype's Trait loot table. A stacking percentage chance per star level to zero out a Trait slot before rolling on the generation table. Low openness produces veteran archetypes (most slots pre-filled); high openness produces open-potential archetypes (many empty slots for the player to fill).

---

## Combat

### Zone
A spatial region on the battlefield. Default map: 5 zones (North, South, East, West, Center) with center + ring adjacency. Center is adjacent to all four cardinals. Cardinals are adjacent to ring neighbors (N↔E, E↔S, S↔W, W↔N). Opposite cardinals (N↔S, E↔W) are NOT adjacent = Long range. Characters occupy a zone and can interact with others based on range.

### Range
Distance classification between zones. **Short**: same zone (melee). **Medium**: adjacent zones. **Long**: non-adjacent zones.

### Initiative
A meter that accumulates per tick based on Speed. Each tick, characters add `sqrt(Speed) × global_initiative_multiplier` to their meter. When Initiative ≥ 100, the character takes a turn. After acting, Initiative is reduced by `100 - Action Speed modifier`. Haste/Slow effects use two channels: Speed modification (permanent, affects sqrt calculation) and Initiative gain modification (temporary multiplier).

### Tick (Combat)
A single time step in combat resolution — the atomic time unit. Each tick resolves in fixed order: (1) Effects Phase — per-tick status effects resolve (DoTs, decay, regen); (2) Initiative Phase — all characters accumulate Initiative; (3) Turns Phase — characters with Initiative ≥ 100 act (one turn per tick maximum); (4) Fallen Resolution — characters at ≤0 HP enter Fallen state. Deaths from any source are deferred to end of tick. See [combat.md](domains/combat.md) for full tick resolution order.

### Global Initiative Multiplier
A per-combat tuning value (default ~3.0) applied to Initiative gain each tick. Controls match pacing without affecting relative character balance. Higher = more turns per tick = faster combat.

### Action Speed
A flat modifier to the base −100 Initiative cost after acting. Positive values = faster recovery (Action Speed +30 → only −70 Initiative cost). Negative values = longer delay (Action Speed −40 → −140 Initiative cost). Default Actions (Attack, Defend, Move, Search) have Action Speed 0.

### Roll-vs-Static Defense
The combat resolution model. Attacker rolls 1 to [Attack Value]; defender's Defense is subtracted as a flat threshold. Uses **dual Defense**: Physical Defense for physical attacks (Blunt, Piercing, Slashing), Magic Defense for elemental and magical attacks. The Action's primary damage type determines which Defense is checked. Negative result = miss. Non-negative result = hit bonus that carries into the damage step. Attack Value and Damage Value are weapon-type-defined — each weapon type has its own attribute blend formula (no fixed derived stats). Unarmed attacks use Might-scaled Blunt damage as a baseline.

### Formula Modifier Tag
A Stat Adjustment subtype that substitutes one attribute for another in formula evaluation (e.g., "Intelligent Strikes" lets Intellect replace Accuracy in Attack formulas). Each modifier specifies a **formula category scope** (attack, damage, defense, healing, etc.) to prevent universal attribute replacement. When multiple modifiers target the same formula slot, the **highest attribute value wins** — the player always benefits from their best option. Enables build diversity without creating new Action variants.

### Physical Defense
A derived combat stat. 40% Speed + 35% Accuracy + 25% Awareness (× scaling multiplier). Represents dodge, evasion, and reading physical attacks. Used as the Defense threshold for Actions with a physical primary damage type (Blunt, Piercing, Slashing).

### Soak
Per-damage-type damage reduction stat used in the damage step. Uses **three family-level base formulas**: Physical Soak (60% Endurance + 25% Might + 15% Speed), Elemental Soak (45% Endurance + 35% Awareness + 20% Luck), Magical Soak (55% Willpower + 25% Intellect + 20% Luck). Each damage type uses its family's Soak value. Formula: `Reduction % = Soak[family] / (Soak[family] + K)` where K is a global tuning constant. Per-type bonuses via tag-scoped Stat Adjustments (e.g., "+5 Soak vs [Fire]") add to the family base for specific types. Penetration reduces effective Soak before the formula: `Effective Soak = Soak[family] - Penetration` (minimum 0).

### Physical Soak
Family Soak derived stat for Blunt, Piercing, and Slashing damage. 60% Endurance + 25% Might + 15% Speed (× scaling multiplier).

### Elemental Soak
Family Soak derived stat for Fire, Cold, and Lightning damage. 45% Endurance + 35% Awareness + 20% Luck (× scaling multiplier).

### Magical Soak
Family Soak derived stat for Poison, Shadow, Light, and Psychic damage. 55% Willpower + 25% Intellect + 20% Luck (× scaling multiplier).

### Penetration
A universal stat (from Perks or equipment affixes) that reduces the defender's effective Soak (for whichever family applies) before the diminishing returns formula: `Effective Soak = Soak[family] - Penetration` (minimum 0). One value — applies against all damage families equally. Does NOT affect Shields. Creates a meaningful counter-stat to Soak stacking.

### Critical Hit
A bonus triggered after a successful hit (or when any Action resolves). Chance derived from Physical Crit (40% Awareness + 35% Luck + 25% Accuracy) or Magic Crit (40% Awareness + 35% Luck + 25% Intellect), based on the Action's primary damage type. **Universal**: crits apply to ALL effect component types — crit damage hits harder, crit heals heal more, crit shields are stronger. **Formula**: `output = base × (1 + crit_multiplier)` where `crit_multiplier` is a tuning value (e.g., 0.5 for +50%). Percentage of base, not a flat addition or separate roll.

### Rider Effect
A modular effect (from a Perk or equipment affix) that triggers as part of the status/rider application step (Step 3 of the combat pipeline — before the damage step). Each rider has its own resolution. Examples: Stunning Blow, Flaming Sword proc, Armor Shred.

### Status Effect
A temporary condition applied to a character, tracked as stacks with a decay model. Unified system for buffs, debuffs, DoTs, and crowd control. Max stacks effectively unlimited (9999 cap). **Minimum 1 stack**: status effects always apply at least 1 stack even after Resistance reduction. DoT damage bypasses Soak (already gated by Resistance at application) but Shields still absorb it. Stun prevents Initiative gain entirely (0 gain while stacks > 0); stacks decay proportionally to Willpower per tick. **Frozen while Fallen**: status effects do not tick down while a character is in the Fallen state. Revival clears all statuses (clean slate).

### Sneaking
A status that prevents a character from being targeted by single-target or selected-target enemy abilities. AoE effects hit Sneaking characters. Break conditions: using a non-stealth-tagged Action, taking damage (break formula: `break_chance = damage_taken / (stealth_stacks × K)` — linear, transparent). Stealth-tagged Actions can be used without breaking Sneaking. Detection (passive or active) allows targeting but does not remove the status. Implemented within the standard stack-based status system.

### Search
A default Action (via the Combatant system Trait) for active stealth detection. Costs a full turn. Searches the entire map for Sneaking enemies, with detection chance decreasing by range (same zone = high, adjacent = moderate, distant = low). See also: Passive Detection (automatic, same-zone only, no action cost).

### Morale
A fighter stat affected by combat events (ally death, kills, heavy damage). May cause debuffs, fleeing, or bonuses. Full design TBD (Phase 4).

### Defend
A default Action (via the Combatant system Trait) available to all characters. Provides a **fixed percentage boost** to Defense and Soak until the character's next turn, plus a Stamina recovery burst equal to **Y% of max Stamina pool** (tuning value). Both the defensive boost percentages and Stamina burst percentage are tuning values.

### Damage Type
A classification for damage. 10 types in three families: Physical (Blunt, Piercing, Slashing), Elemental (Fire, Cold, Lightning), Magical (Poison, Shadow, Light, Psychic). No inherent rock-paper-scissors — differences come from affiliated status effects and Perk/equipment interactions.

### Combatant (System Trait)
A hidden, immutable system Trait at level 1 present on all characters. Not displayed in UI, does not occupy a Trait Slot. Contains a single Perk with default Actions (Attack, Defend, Move, Search) as standard Perk Action components. Ensures all Actions go through the same resolution pipeline — no special-case code for basic actions.

### Effect Resolution Order
Within a single Action or Trigger, effect components resolve in type order: (1) status/buff/debuff effects, (2) damage/healing effects, (3) movement effects. Components within the same category resolve simultaneously. Defined authoritatively by the [combat](combat.md) spec. Multi-Trigger resolution remains simultaneous (all qualifying Triggers fire together).

### Summon Control Capacity
A Resource Pool granted by summoner Traits that acts as a **budget** for active summons. Each summon type has a control cost; the summoner can maintain summons up to their total capacity. Dying summons free capacity. Example: "Necro Control" = 0.8×Willpower + 0.2×Charisma. A budget-based system, not per-Perk cap — a summoner with 100 capacity can field five 20-cost skeletons or two 50-cost golems. Revived summoners start with full capacity but no active summons (must re-summon).

### Fallen Target Tag
A target tag (`[Fallen]`) used by Actions that exclusively target Fallen allies (e.g., `[Fallen, Ally, Single]`). Only Revive-type Actions use this tag. The Revive effect component itself works on any ally — on non-Fallen targets it does nothing but other components still resolve. This enables hybrid heal+revive Actions (using `[Ally, Single]`) alongside dedicated Revive-only Actions (using `[Fallen, Ally, Single]`).

### Overkill
The amount of excess damage dealt beyond 0 HP when a character is reduced to Fallen. Overkill = `abs(final_HP)` regardless of damage source — DoTs, direct damage, and environmental effects all contribute. Tracked per Fallen character. If a character falls multiple times (fell, revived, fell again), only the **final fall's overkill** is used for post-combat injury rolls — previous falls negated by revival don't accumulate. Used as a factor in post-combat injury severity rolls — higher overkill increases both the chance and severity of injuries. See [post-combat](domains/post-combat.md).

### Injury Check
A tiered post-combat roll for Fallen characters when injury is enabled for the event type. Only characters **Fallen at combat end** are checked — revived characters are treated as alive. **Tier 1**: determines if an injury occurs using `injury_chance = overkill / (overkill + IR × K)` where IR = Injury Resistance and K is a tuning constant (consistent with Soak/Resistance diminishing returns formula). **Tier 2**: weighted random severity draw — higher overkill shifts toward worse outcomes, higher IR shifts toward better. Severity tiers: minor (temporary stat penalty, 3–5 game ticks), major (significant penalty, 10–15 game ticks), critical (permanent Potential reduction or anatomical slot loss, 20–30 game ticks), or death. Named injury types within each tier are content data. Specific rare Perks can grant death immunity (worst outcome = critical). Recovery happens on game ticks; Group healing services can accelerate. See [post-combat](domains/post-combat.md).

### Injury Resistance
A derived stat used in post-combat injury rolls. **Formula**: 50% Endurance + 30% Luck + 20% Willpower (× scaling multiplier). Endurance is physical toughness (primary), Luck provides fortune, Willpower represents mental grit. Perk bonuses add flat bonuses on top. Used in both Tier 1 (`overkill / (overkill + IR × K)`) and Tier 2 (shifts severity weights toward better outcomes) of the injury check. See [post-combat](domains/post-combat.md).

### Post-Combat Configuration
A per-event-type profile that determines which post-combat phases are active and their parameters. Uses an **inherited defaults + overrides** model: each event category (tournament, PvE, exhibition, vendor practice) defines a default profile; individual event templates override specific fields. Profile fields include: injury/discovery/recruitment/loot/XP enabled flags, recruitment chance, recruit Star Rating range, gold reward/cost, and placement reward scaling. See [post-combat](domains/post-combat.md).

### Resistance (Combat)
A per-damage/status-type defensive value. Status effects always apply (no binary resist roll). Resistance provides dual reduction using **percentage with diminishing returns**: `value × (1 - resist_ratio)` where `resist_ratio = Resistance / (Resistance + K)`. Same formula and same K per type for both **stack reduction** and **damage reduction**. Parallels the Soak formula. **Minimum 1 stack** always applies regardless of Resistance. **Enemy-sourced only**: resistance applies only to effects from enemy sources — friendly buffs and heals always apply at full strength. Sources: Attributes, Perks, equipment, status effects.

### Judgment
A derived combat stat. 40% Awareness + 35% Willpower + 25% Intellect (× scaling multiplier). Controls AI decision quality via three parameters: tactical-vs-personality blend (how much the character weighs objective effectiveness vs personality impulses, range ~0.3 to ~0.95), selection sharpness (how reliably the character picks the top-scoring Action, range ~1 to ~10), and lookahead depth (how many ticks ahead the character can project deterministic events, range 1 to 10). Modified by Star Rating, Perks, equipment affixes, and status effects. High Judgment = near-optimal play with strategic foresight; low Judgment = personality-driven, impulsive behavior with minimal prediction.

### Consideration
The atomic unit of AI reasoning in the Utility AI system. A function that evaluates game state and returns a score for a specific Action. Two types: Gates (binary pass/fail) and Scorers (continuous preference signal). Defined as part of each Perk Action's data model. See [combat-ai](domains/combat-ai.md).

### Gate (AI)
A binary Consideration (0.0 or 1.0) that acts as a hard prerequisite for an Action. Any Gate returning 0 vetoes the Action entirely. Standard Gates: resource_gate, cooldown_gate, range_gate, target_exists_gate.

### Scorer (AI)
A continuous Consideration (0.01–1.0, never exactly 0) that evaluates how good an Action is in the current context. Scorers within a track are multiplied together and normalized via geometric mean. Two tracks: Tactical (objective effectiveness) and Personality (character temperament).

### Tactical Score
The normalized product of all Tactical Scorers for an Action. Represents objective combat effectiveness — "how good is this Action right now?" All Actions have Tactical Scorers.

### Personality Score
The normalized product of all Personality Scorers for an Action. Represents how well an Action fits the character's temperament. Derived from personality archetype tags on Core Traits (Aggressive, Cautious, Protective, Vindictive, Showoff). Defaults to 0.5 (neutral) for Actions without Personality Scorers or characters without personality archetypes.

### Utility Score
The blended final score for an Action: `judgment_blend × tactical_score + (1 - judgment_blend) × personality_score`. Used by the selection mechanism (weighted random with Judgment-controlled sharpness) to choose which Action the character takes.

### Selection Sharpness
A parameter derived from Judgment (range ~1 to ~10) that controls how reliably the AI picks the top-scoring Action. Applied as an exponent: `selection_weight = final_score ^ (1 + sharpness)`. Low sharpness = unpredictable, characterful; high sharpness = consistent, optimal.

### Response Curve
A function that maps raw game-state values to the 0.01–1.0 output range for Scorers. Common shapes: linear, logistic (S-curve), exponential, step. Each Scorer definition includes curve parameters (thresholds, slopes, inflection points) as part of its data.

### Personality Archetype
A content-level tag on Core Traits that represents a character's temperament. Feeds the combat AI's Personality Score track. Five standard archetypes: Aggressive, Cautious, Protective, Vindictive, Showoff. A character can have multiple archetypes from multiple Core Traits — all contribute equally, and contradictory archetypes create internally conflicted characters. New archetypes can be added by extending the vocabulary.

### Combat Context Flags
An explicit metadata object passed from the combat system to the AI system at fight start. Contains fight circumstances: `is_tournament`, `round`, `total_rounds`, `consumables_replenish`, `is_exhibition`, `pve_tier`, `team_sizes: [int]` (list of team sizes, own team first), etc. Any AI Scorer can query these flags to adjust behavior based on context (e.g., consumable conservation in tournaments, risk tolerance in PvE). Defined by the combat system; consumed read-only by the AI. See [combat-ai](domains/combat-ai.md).

### Attrition Ramp
An anti-stalemate combat mechanic. After a configurable onset tick (~100), a stacking global damage bonus (+0.5–1% per tick) applies to all combatants. Healing is unaffected. Onset tick and escalation rate are per-event-type, specified in [Combat Context Flags](domains/combat.md#combat-context-flags). No hard tick limit — the exponentially increasing damage makes extended fights increasingly lethal. The onset tick also serves as `estimated_max_ticks` for the Fight Phase Signal.

### Shield (Combat)
A damage absorption layer applied after Soak reduction in the combat pipeline. Each Shield instance has its own HP pool, optional duration, and optional damage type filter. Multiple shields stack — damage drains the **oldest shield first** (FIFO). Penetration does NOT affect shields (only reduces Soak). When a shield's HP reaches 0, remaining damage carries through to the next shield or Health. Created by Perks, equipment, and consumables via the Shield effect component type.

### Combat Scoreboard
Structured per-character stat tracking during combat. Records: damage dealt/taken/healed, kills, Actions used (by type), Fallen events, revival events, Dying Blows, ticks survived. Available to the [post-combat](domains/post-combat.md) system for Perk Discovery eligibility (determining "active Traits") and to [meta-balance](architecture/meta-balance.md) for win/loss tracking. Does not affect combat mechanics.

### Fight Phase Signal
A normalized ratio (`current_tick / estimated_max_ticks`) exposed to the AI system and Triggers during combat, where `estimated_max_ticks` = the attrition ramp onset tick from [Context Flags](domains/combat.md#combat-context-flags). Values < 1.0 indicate normal combat; ≥ 1.0 indicates the attrition phase is active. Used by the `resource_efficiency` Scorer for conservation strategy and available to Triggers for phase-dependent effects.

### Consumable Scarcity
An AI Tactical Scorer (`consumable_scarcity`) that manages limited-use consumable conservation. Penalizes consumable usage unless the situation warrants it, scaling with remaining uses (fewer left = higher bar), impact threshold (must be impactful to score high), and tournament awareness (conserve across rounds when consumables don't replenish). Part of the standard Scorer library. See [combat-ai](domains/combat-ai.md).

### Stealth Awareness (AI)
An AI Tactical Scorer (`stealth_awareness`) that adjusts scoring when Sneaking enemies are present. Boosts AoE Actions (which hit Sneaking characters), boosts Search for high-value stealth targets, and reduces single-target damage scores when the best available targets are Sneaking and untargetable. Part of the standard Scorer library. See [combat-ai](domains/combat-ai.md).

### Future State Value
An AI Tactical Scorer (`future_state_value`) that implements Judgment-gated lookahead. Projects deterministic future events (Effects Phase outcomes + Initiative timing) up to the character's lookahead depth (1–10 ticks based on Judgment). Scores Actions based on how favorable the projected future state is. Existing Scorers evaluate current state only; this Scorer adds prediction as an independent signal. See [combat-ai](domains/combat-ai.md).

### Lookahead Depth
The number of ticks ahead a character's AI can project deterministic game-state events. Controlled by the Judgment stat: range 1 tick (low Judgment) to 10 ticks (high Judgment). Projects Effects Phase outcomes (DoTs, regen, buff/debuff expiry) and Initiative timing (when characters will act). Does NOT include threat estimation (guessing enemy Actions) or probability projection. With fights lasting 25–150 ticks, 10-tick lookahead represents ~7–40% of a fight. See [combat-ai](domains/combat-ai.md).

### Dying Blow
An Action taken by a character while at ≤0 HP, before end-of-tick Fallen resolution. Because deaths are deferred to the end of each tick, a character reduced below 0 HP by another character's Action can still take their turn — this final action is a Dying Blow. Tracked in the Combat Scoreboard. Available as the `OnDyingBlow` Trigger event type, enabling Perks like "Last Stand" (+100% damage on Dying Blows).

### Primary Damage Type
A field on Actions with Damage components that designates which damage type determines the Defense check (Physical Defense vs Magic Defense) in Step 1 of the combat pipeline. All Damage components in the Action share the single hit/miss result from the primary type's Defense check. Content authors set the primary type to match the Action's theme.

### Combat Event Stream
A stream of structured typed events emitted by the combat engine during simulation. Events include AttackEvent, DamageEvent, FallenEvent, MovementEvent, HealEvent, StatusEvent, DyingBlowEvent, etc. Combat simulates the entire fight to completion (deterministic with seeded PRNG), then the TUI presents results via dramatized summary and/or detailed event logs. The event stream is the authoritative combat record, enabling replays, statistics, and flexible presentation.

### Resource
A pool spent to use certain Actions. Default resources (all characters): Health, Stamina. Trait-defined resources belong to five default types: Mana (Arcane), Faith (Divine), Spirit (Primal), Focus (Psychic), plus content-author-defined types. See Resource Pool in Traits & Perks section.

---

## Equipment

### Multi-Slot Item
An item that occupies multiple Anatomical Slots. Example: Full Plate covers Head + Torso + Arms + Legs. Base stats scale **less than proportionally** with slots covered (~1.0/1.7/2.2/2.6/3.0× for 1-5 slots). Same affix budget as single-slot items of equivalent quality. Trade-off: consolidated base stats vs. multiple affix sources from separate pieces.

### Weapon Type
A content-defined template for **physical combat implements**. Carries base damage, Attack formula weights (attribute blend), Damage formula weights, default damage type, tags, and range. **Action Speed is tag-derived** (Light=+bonus, Heavy=−penalty, Two-Handed=−small), not per-type. Individual weapon items of a type vary only by quality and affixes. 12 MVP types: Sword, Dagger, Mace, Axe, Spear, Greatsword, Greataxe, Warhammer, Bow, Crossbow, Hand Crossbow, Sling (+ Unarmed default). Implement tags (Defend, Deadly, Sunder) provide combat role identity. Rare affixes can override formula weights (item-local Formula Modifier Tags). **Weapons are distinct from Focuses** — weapons handle physical Actions, focuses handle magical Actions. **Versatile** applies to weapons only (Sword, Axe, Spear) — no Versatile focuses. See [equipment.md](domains/equipment.md).

### Focus Type
A content-defined template for **magical combat implements** — the magical parallel to Weapon Types. Carries base magic power, formula weights (attribute blend), default damage type (varies per item), tags, range, and **combat role** (targeting pattern, defense bonuses, crit bonuses). **Action Speed is tag-derived** (same as weapons). Focus types provide the default damage basis for magical Actions the same way weapon types provide the default damage basis for physical Actions. Magical Perk Actions commonly override focus defaults for range and targeting tags while still using focus base magic power. 7 MVP types: Wand (Long, Light, sniper with crit bonus, Intellect+Accuracy), Tome (Long, Two-Handed, AoE multi-target, Intellect+Awareness), Staff (Medium, Two-Handed, tanky with defense bonuses, Intellect+Willpower), Orb (Medium, Light, multi-target, Intellect-weighted), Holy Symbol (Short, Light, multi-hit with magic defense bonus, Willpower-weighted), Artifact (Long, Light, wild card with per-item overrides), Rod (Short, Light, caster tank with defense bonuses, Willpower+Endurance). All carry the Focus tag. **No Versatile focuses** — focuses are strictly Light or Two-Handed. See [equipment.md](domains/equipment.md).

### Unfocused
The magical equivalent of Unarmed. Using a magical Action without an equipped Focus gives a weak base magic power — slightly stronger/less punishing than physical Unarmed. Provided by the Combatant system Trait alongside Unarmed. Perks can enhance.

### Implement Resolution
The system for determining which equipped item provides base stats for an Action. Actions specify an implement type (physical or magical). Resolution order: (1) check main-hand for matching implement, (2) check off-hand, (3) use default (Unarmed for physical, Unfocused for magical). When both hands have matching implements, main-hand takes priority.

### Enchanting
A Group vendor service that adds affixes to **empty affix slots** on items. Uses the **same source-themed affix tables** as loot generation — each Group vendor biases enchanting toward their theme (e.g., Fire-themed Group enchants fire-biased affixes). Fills slots whether empty from creation (random 1-to-max generation) or opened via Quality Forging past a tier threshold. A two-step upgrade path: forge to increase quality and open slots, then enchant to fill them. See [equipment.md](domains/equipment.md).

### Quality
A 0–500+ numeric scale for equipment effectiveness. Acts as a **linear** percentage modifier to base stats (100 = 1.0× baseline, 500 = 5.0×). Maps to Star Rating tiers: 0–99 = 0★, 100 = 1★, 200 = 2★, 300 = 3★, 400 = 4★, 500+ = 5★. Linear scaling is balanced because quality multiplies base stats that are themselves built on character attribute blends — a low-level character with top gear roughly matches a high-level character with basic gear on raw numbers.

### Durability
Equipment's maximum quality ceiling. Current quality drops per fight via **usage-proportional degradation**: `base_degrade × usage_weight`. Base degrade ≈ **~2 quality per fight** at 1.0× usage. Usage weight reflects how actively the item was used: weapons track Actions taken, armor tracks damage absorbed, accessories track ticks survived. Range: 0.5× light use to 1.5× heavy use. **No quality floor** — items degrade all the way to 0. A 100-quality item lasts ~50 fights without repair. When max durability reaches 0, the **item is destroyed**. Three repair tiers: Cheap (restores to max−2), Standard (restores to max−1), Premium (restores to max, no permanent loss, very expensive). See [equipment.md](domains/equipment.md).

### Quality Forging
A Group-gated vendor service (e.g., Blacksmith's Guild) that increases an item's max quality. Exponentially increasing cost per quality point. Available in MVP 1. Creates "maintain vs. replace" tension — is it cheaper to forge an existing item or find a new one?

### Block Chance
A defensive mechanic provided by equipped **shield items** (Buckler, Kite Shield, Tower Shield). Separate from the Shield HP-pool combat mechanic (which is Perk/spell-only). After a successful hit, the shield's block chance % triggers; on success, the shield's **block value** (flat amount, quality-scaled) is subtracted from damage before Soak reduction. Two stats: block chance % (~20% Buckler, ~35% Kite, ~50% Tower) and block value. See [equipment.md](domains/equipment.md) and [combat.md](domains/combat.md).

### Affix
A modifier on equipment using the **unified Perk component data model** — an affix IS a mini-Perk bound to an item. Each affix can contain any combination of Stat Adjustments, Triggers, Actions, and/or Damage components, using the exact same data structures as Perks. The only difference: affixes are tied to items (lost when unequipped/destroyed), Perks to Traits. **Tiered progressive pools**: 1-2★ basic (Stat Adjustment bonuses), 3★ intermediate (Damage components, Triggers/proc effects, resource pool bonuses), 4★ advanced (Formula Modifier Tags, simple granted Actions), 5★ rare (complex Triggers, powerful Actions, mythic effects). Budget scales inversely with tier: fewer affixes = stronger each (1★ at 1.4×, 5★ at 0.5× per affix). Generated via **source-themed tables** — each loot source biases toward thematic affixes. **Rerolling** available via Group vendors (single-affix and full-affix options). Can include **negative components** (cursed items). Proc effects use the combat Trigger system directly (OnHit, OnHitBy, etc.).

### Unique Item
A hand-authored item template with fixed name, lore text, and some locked affixes, plus 1-2 random affix slots for per-drop variety. Found from specific sources (boss drops, rare loot entries, quest rewards). Chase items that define builds — recognizable but with per-drop variation. Unique item templates are content data with: fixed weapon/armor type, fixed quality range, fixed affixes (using unified component model), random slot count, and source constraints.

### Household Stash
A shared storage pool for all unequipped items in a player's household. Any character can equip from stash. No per-character inventory. Items in stash do **not degrade** — degradation is combat-only, no passive time-based decay. Dead characters' gear stays equipped on their corpse; if permanently lost (not resurrected), gear returns to stash. Large capacity (unlimited or upgradeable).

### Dual-Wield
Equipping two one-hand items of the same or mixed categories (weapons, focuses) in the two Hand slots. The main-hand item's Action resolves first; the off-hand fires a **generic bonus Attack** of its implement type (physical for weapons, magical for focuses). Main-hand takes a **−15%** Attack/Damage penalty; off-hand takes a **−35%** penalty. Raw DW ≈ 75% total output — Perks (Ambidextrous, Two-Weapon Fighting) mitigate. Applies equally to weapon+weapon, focus+focus, and weapon+focus combinations. Off-hand restricted to **Light-tagged** items by default; Perks override this restriction. Each implement's proc Triggers fire independently (per-weapon/focus). **Offensive implement tags (Deadly, Sunder) are per-implement** — they only apply to attacks made with that specific weapon/focus. **Defensive implement tags (Defend, Deflect) stack from both** implements, but computed from each implement's Attack Value (which already includes DW penalties). See [equipment.md](domains/equipment.md).

### Tag
A shared mechanical keyword used by Perks, equipment, and consumables to create emergent synergies. Tags live on individual Perks and their component Actions; Traits derive their tags from the union of all their Perks' tags (no independent Trait tags). Perks that grant or cost a resource type automatically gain the corresponding auto-tag (e.g., Mana → Arcane). Tags can stack on a single item. Equipment tags include weapon tags (Reach, Two-Handed, Light, Heavy, Finesse, Ranged, Versatile), focus tags (Focus, Two-Handed, Light), implement tags shared across both (Defend, Deflect, Deadly, Sunder, Multihit), and armor tags (Light/Medium/Heavy Armor, Shield). Implement tags define combat role mechanics with two scope types: **passive** tags (Defend, Deflect — character-wide while equipped, like armor) and **per-implement** tags (Deadly, Sunder — apply only to attacks made with that specific weapon/focus). Defend (+Physical Defense from ~10% Attack Value), Deflect (+Magic Defense from ~10% Attack Value), Deadly (flat +crit chance, per-implement), Sunder (flat +Penetration, per-implement), Multihit (default Attack hits 1–5 enemies, triangular distribution). **Any implement tag can be added to any weapon/focus via affixes or Unique Item template overrides** — a common affix pattern. **Focus** is a generic tag on all magical implements (replaces former "Arcane Focus"). **Light** items can go in off-hand by default. **Reach** attacks incur an Accuracy penalty. **Versatile** items automatically use two-hand mode if off-hand is empty, one-hand mode if occupied. Examples: Fire, Healing, Reach, Two-Handed, Light, Heavy, Finesse, Ranged, Focus, Arcane, Divine.

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

_Last updated: 2026-02-21 — Equipment round 31 cleanup: Deadly/Sunder per-implement scope, tag-derived Action Speed, enchanting source-themed, DW penalties (−15%/−35%), degradation ~2/fight with accessory ticks-survived tracking, no Versatile focuses. Previous: round 30 Multihit, focus hierarchy, ranged formulas, Equipment Slot removed. 2026-02-18 post-combat and combat terms._
