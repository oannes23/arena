# Roguelike Arena Fighter Management Game — Complete Design Document

## Project Overview

A procedurally generated gladiatorial house management game designed as an idle/auto-battler for work downtime (particularly CI/CD pipeline waits). Players recruit fighters, optimize builds through complex Trait/Perk systems, manage equipment and economy, and watch automated tactical combat unfold in a classic roguelike TUI interface.

**Tech Stack**: Python + Claude Code, MUD-style architecture (headless server + TUI client), future expansion to web/mobile clients

**Design Philosophy**: Deep build optimization with economic pressure, played in short sessions. Check in, review results, make decisions, queue up, go back to work. The complexity lives in roster and build management, not moment-to-moment reflexes.

**Game Modes**:
- **Single-player**: Player controls time advancement between fights; idle when you want, grind when you want
- **Multiplayer**: Persistent world state, scheduled tournaments, PvP matchmaking, NPC farming encounters

---

## Core Systems

### Star Rating System (0–5 Stars)

Universal quality/power rating applied across all game systems:

| Stars | Color  | Tier        |
|-------|--------|-------------|
| 0★    | Gray   | Broken/Degraded |
| 1★    | White  | Standard/Common |
| 2★    | Green  | Superior/Uncommon |
| 3★    | Blue   | Rare |
| 4★    | Purple | Epic |
| 5★    | Orange | Legendary |

Applied to: Characters, Traits, Perks, Equipment quality tiers.

---

## Character System

### Character Star Rating

Determines Trait slot capacity across three categories:

| Character Stars | Core Slots | Role Slots | Bond Slots |
|-----------------|-----------|-----------|-----------|
| 1★ | 1 | 1 | 1 |
| 2★ | 2 | 2 | 2 |
| 3★ | 3 | 3 | 3 |
| 4★ | 4 | 4 | 4 |
| 5★ | 5 | 5 | 5 |

Each slot holds ONE Trait of any star level (no budget math — a 1★ character can hold a 5★ Trait, they just only have one slot per category).

### Attributes

Every character has **Current** and **Potential** values for each Attribute:
- **Current**: Actual stat value used in combat and checks
- **Potential**: Maximum trainable value for that Attribute
- **Generation**: Lower Current often correlates with higher Potential (growth opportunity characters vs. ready-made veterans)
- **Scale**: Average character ~50 Current, ~100 Potential
- **Training**: Spend XP to increase Current toward Potential cap

**Primary Attributes**:
- **Strength**: Physical power, melee damage, carry capacity, heavy equipment requirements
- **Agility**: Speed, dodge, accuracy, light/finesse weapon scaling
- **Willpower**: Mental resistance, spell power, resource pools, status resistance (Stun, Psychic, etc.)
- **Awareness**: Detection (stealth counterplay), trap awareness, initiative tie-breaking, targeting intelligence

### Traits (Three Categories)

Traits are the core identity building blocks of a character. Each Trait unlocks a tree of Perks that the character can purchase with XP.

#### Core Traits — Who the Character Is

Innate character aspects that define identity:
- Ancestry (Orc, Elf, Human, Dragonblood, etc.)
- Personality aspects (Stubborn, Berserker's Fury, Calm Under Pressure)
- Unusual gifts/destinies (Fated Hero, Touched by Shadow, Born Under a Red Moon)

#### Role Traits — What the Character Can Do

Functional capabilities, both combat and support:
- **Combat roles**: Warrior, Defender, Trickster, Tactician, Supporter, Pyromancer, Cryomancer, etc.
- **Support roles**: Blacksmith, Medic, Alchemist, Merchant, Trainer, Scribe
- **Dual purpose**: Characters can fight AND provide household support if their Perks allow it
- Example: A Blacksmith Trait includes combat Perks (Strength boost, Smith's Strike attack) AND support Perks (craft weapons/armor for the household)

#### Bond Traits — Who the Character Is Connected To

Organizational affiliations that unlock vendor access and themed content:
- Wizard guilds, temples, witch covens, martial arts schools
- Fighting style lineages, secret societies, criminal networks
- Higher Bond Trait levels = better vendor access, pricing, and exclusive items
- Examples: "Pyromancer's Guild Member", "Priest of Tharzul", "Nanna-Sin Monastery Disciple"

### Trait Mechanics

- **Requirements**: Can require specific Attributes, other Traits/Perks, derived stats, or achievements
- **Minimum Star Level**: Each Trait has a minimum purchase level (1–5★)
- **Leveling**: All Traits can be leveled up to 5★ regardless of minimum
- **Trait Level Effects**: Modifies ALL Perks within that Trait (higher Trait level = stronger Perks)
- **Respeccing**: Expensive vendor service to remove Traits — encourages roster turnover over endlessly iterating a single character
- **Cross-Category Sharing**: Identical Perks can appear in different Traits across different categories

### Perks (Abilities, Bonuses, and Triggers)

Perks are multi-component packages unlocked by Traits. They are the atomic unit of character capability.

**Perk Components** (any combination per Perk):

1. **Actions**: Active combat abilities the character can use on their turn
2. **Stat Adjustments**: Permanent attribute bonuses applied while the Perk is owned
3. **Triggers**: Conditional automatic effects that fire when specific combat events occur

**Example — Flame Aura Perk:**
- Action: Activate "Flame Aura" (grants 10 stacks, costs 50 Mana, 5-turn cooldown)
- Stat Adjustment: +15 Willpower (permanent while owned)
- Trigger: When struck in melee, gain 1 stack of Flame Aura

**Perk Mechanics:**
- **Requirements**: Can require Attributes, Traits, other Perks, derived stats, achievements
- **Minimum Star Level**: 1–5★ minimum purchase level
- **Leveling**: All Perks can level to 5★
- **Perk Level Effects**: Modifies only that specific Perk (independent of Trait level, but stacks with it)
- **Acquisition**: Purchase from Trainers using XP (must own the parent Trait first)
- **Auto-Granted**: Some Perks automatically granted when acquiring their parent Trait

---

## Combat System

### Battle Format

- **Scale**: 1v1 to 20v20
- **Automation**: Pre-configured AI orders; combat executes automatically (idle/auto-battler)
- **Team Composition**: Variable team sizes for different match types
- **Player Interaction**: Players configure builds and AI behavior before combat; during combat, they spectate

### Spatial System

**Ranges**:
- **Short**: Same zone (melee range)
- **Medium**: Adjacent zones
- **Long**: Non-adjacent zones

**Zone Maps**:
- **Default**: 5 zones (North, South, East, West, Center)
- **Variants**: Custom layouts scaled to combatant count
- **Gimmicks**: Traps, timed hazards (leave zone or take damage), movement modifiers, obstacles
- **Zone Capacity**: No strict limits by default, but can be specified per-zone per-battlefield

**Movement**:
- Basic Move action (costs the character's turn — see Action Economy)
- Forced movement effects (push to random adjacent zone, push away from character, pull toward, etc.)

### Action Economy

When a character's Initiative reaches the action threshold, they take a turn. A standard turn consists of:
- **One Action**: Attack, use an Action Perk, Defend, or Move

Some Perks may grant bonus actions, free movement, or allow combining movement with attacks. These are exceptions granted by specific Perks, not baseline behavior. The default is one action per turn.

**Design Intent**: Making Move cost your full turn creates meaningful positioning decisions and gives ranged characters a genuine advantage — melee fighters pay a real cost to close distance. Perks that grant "attack + move" or "free movement on kill" become high-value abilities.

> ❓ **OPEN**: Exact action economy rules (free actions, bonus actions, reaction-type triggers) need full specification during Phase 1 prototyping.

### Initiative System

**Speed-Based Meter with Diminishing Returns**:
- Each tick, characters add `sqrt(Speed)` to their Initiative meter (diminishing returns prevents degenerate speed stacking)
- When Initiative ≥ 100, character acts
- After action, Initiative is reduced (base -100, modified by Action Speed of the ability used)
- Speed derived from: Agility + Equipment + Perks + Statuses

**Diminishing Returns Model**: Square root scaling means doubling your Speed from 100 to 200 only increases Initiative gain from 10.0 to ~14.1 per tick (41% increase for 100% more Speed). This keeps speed valuable without letting it become the only stat that matters.

> ❓ **OPEN**: May switch to logarithmic scaling or a bracket system if square root proves hard to tune. The principle — strong diminishing returns on speed stacking — is locked in; the specific formula is not.

**Initiative Tie-Breaking Order**:
1. Highest total Initiative (current + this tick's addition)
2. Highest Initiative added this tick
3. Highest Agility attribute
4. Highest Awareness attribute
5. Highest sum of all Attributes
6. Random coin flip

### Actions

**Basic Actions** (available to all characters):
- **Attack**: Standard damage attempt using equipped weapon
- **Move**: Change zones (costs full turn by default)
- **Defend**: Increase defensive stats temporarily until next turn

**Active Actions** (from Perks):
- Custom abilities with varying effects
- **Action Speed**: Determines how much Initiative is subtracted after use (fast actions let you act again sooner)
- **Cooldowns**: Variable types:
  - Standard: X turns after use
  - Toggle: Using A enables B, using B enables A
  - Stack-gated: Only usable at X stacks of status Y
  - Trigger-based: Available after specific combat events (e.g., "usable after taking a critical hit")
- **Resource Costs**: Mana, Stamina, Faith, etc. (see Resources)

### Resources

**Default Resources** (all characters have these):
- **Health**: Damage capacity. Fully recovers between battles. No natural in-combat regeneration — all healing comes from Perks, equipment, or consumables.
- **Stamina**: General-purpose action resource for physical abilities.

**Trait/Perk-Defined Resources** (unlocked by specific Traits):
- **Mana**: Shared by elemental magic Traits (Pyromancer, Cryomancer, Spatiomancer, Chronomancy)
- **Faith**: Divine Traits (Priest of Tharzul, Temple of the Forge God, etc.)
- Other resources defined per Trait/Perk family as needed

**Resource Scale**: Hundreds to thousands for granular tuning (e.g., a heal might restore 150 HP out of 2000 max, not 15 out of 200).

**Healing Design Principle**: There is no passive in-combat healing. All healing is an active choice — a Perk, a consumable, or an equipment proc. This means healing is a build investment. Teams in 3v3+ formats may want a dedicated healer, and healing Perks/consumables are common enough to be accessible but costly enough to be a real tradeoff.

### Damage Types

**Physical**:
- **Blunt** (hammers, maces, clubs, unarmed)
- **Piercing** (spears, arrows, daggers thrust)
- **Slashing** (swords, axes, claws)

**Elemental**:
- **Fire** (burning, explosions, heat)
- **Cold** (freezing, ice, chill)
- **Lightning** (shock, electricity, thunder)

**Magical**:
- **Poison** (toxins, venom, disease)
- **Shadow** (darkness, necrotic, life drain)
- **Light** (holy, radiant, purifying)
- **Psychic** (mental, illusion, mind damage)

**Type Philosophy**: No inherent rock-paper-scissors advantages between types. Differences come from affiliated status effects and Perk/equipment interactions. Fire tends to inflict Burning (DoT), Cold tends to inflict Slow, Blunt tends to Stun, Piercing tends to cause Bleeding — these are thematic patterns, not hard-coded rules. Any damage type can theoretically do anything via the right Perk or affix.

### Combat Resolution Flow

Combat uses a **roll-vs-static-defense** model where the attacker rolls and the defender's stats provide a flat threshold.

**Step 1: Attack Roll vs. Defense**
- Attacker rolls 1 to [Attack Value]
- Defender's Defense value acts as a flat subtraction
- **Result < 0**: Miss — no damage
- **Result ≥ 0**: Hit — the excess becomes damage potential bonus

*Example*: 120 Attack vs. 50 Defense → roll 1–120, subtract 50. Results range from -49 (miss) to +70 (strong hit). Hit chance ≈ 58%.

**Step 2: Damage Roll vs. Soak**
- Base weapon damage + excess from Step 1, rolled against Soak (armor/resistance)
- Same model: roll 1 to [Damage Value], subtract Soak
- **Result < 0**: Fully absorbed — no Health loss
- **Result ≥ 0**: Excess is damage dealt to Health

**Step 3: Rider Effects** (modular hooks from equipment/Perks):
- Applied after a successful hit, each with its own resolution
- **Stunning Blow**: Roll vs. Stun Resistance, excess = Stun stacks applied
- **Flaming Sword Proc**: Roll vs. Fire Resistance, excess = Burning DoT stacks
- **Armor Penetration**: Reduces target's effective Soak before the damage roll
- Rider effects are defined per-Perk and per-equipment-affix; the system supports arbitrary rider hooks

**Balance Implications of Roll-vs-Static**:
- Defense stacking is very powerful (flat subtraction). Defense should scale more slowly than Attack to prevent unkillable tanks.
- Low Attack values become completely ineffective against high Defense (hard tier gating). A character with 30 Attack cannot hit a 50 Defense target. This is intentional — mismatched fights should be non-competitive.
- All variance is on the attacker's side; defenders have predictable, plannable durability.

> ❓ **OPEN**: Does the damage roll (Step 2) use the same roll-vs-static model, or a different one? Double flat subtraction means heavy armor builds may be nearly immune to chip damage. This might be desirable (rewarding armor investment) or frustrating (low-damage builds feel useless). Needs playtesting.

### Status Effect System

**Stack-Based Mechanics**:
- **Max Stacks**: Effectively unlimited (9999 cap, mathematically unreachable in practice)
- **Decay**: Variable per status type (per-tick reduction, conditional reduction, etc.)
- **Application**: From rider effects, triggers, environmental hazards, Perks

**Status Examples** (illustrative — specific numbers are tuning targets, not final):

| Status | Effect | Decay Model |
|--------|--------|-------------|
| Stun | Reduces or prevents Initiative gain | Decays by Willpower per tick |
| Burning | Fire DoT per tick | Fixed per-tick reduction |
| Bleeding | Physical DoT | Exotic: reduces by 1 per Health restored |
| Slow | Reduces Speed (and thus Initiative gain) | Fixed duration or per-tick |
| Sneaking | Cannot be targeted (see Stealth) | Broken by attacking or failed check |
| Buff/Debuff | Stat modifications | Various decay rates |

**Design Note**: Status effects use the same stack-based system universally. Buffs, debuffs, DoTs, and crowd control all work on stacks with varying decay models. This keeps the system consistent and extensible — adding a new status effect just means defining its per-stack effect and its decay rule.

### Stealth & Detection

Some characters have Perks that allow them to enter a **Sneaking** status:
- While Sneaking, a character cannot be targeted by enemies
- Enemies must pass a **contested Awareness check** to detect a Sneaking character
- Failed detection: cannot target that character with anything (single-target or selected-target abilities)
- Sneaking is implemented as a status within the standard stack-based system

> ❓ **OPEN**: Does Sneaking break on attack (classic stealth — strike from stealth, then you're revealed)? Or can characters attack and remain hidden? This has massive implications for Trickster builds. "Attack from stealth, re-stealth" loops would be extremely powerful and need careful Perk gating if allowed.

> ❓ **OPEN**: How does AoE interact with Sneaking? Can an AoE hit a Sneaking character in the same zone even if they can't be individually targeted?

### Targeting & Multi-Target

**Target Style Flags** (defined per-Action):
- Single selected
- Single random
- Multiple selected
- Multiple random
- Hybrid (e.g., single high damage + multiple random low damage)
- **Friendly fire flag**: Defined per-Action (some AoEs hit allies, others don't)

### Morale System

Fighters have a Morale value that can be affected by combat events:
- Watching allies fall, facing overwhelming odds, or taking heavy damage may reduce Morale
- Low Morale could cause debuffs, fleeing, or refusal to use certain abilities
- High Morale (from kills, rallying Perks, etc.) could grant bonuses

> ❓ **OPEN**: Full morale mechanics need design. Key questions: Is Morale a combat-only stat or does it persist between fights? What triggers affect it? What are the consequences of low/high Morale? Is it a stack-based status or a separate system?

### Combat AI

Players configure AI behavior before combat. Combat then executes automatically.

**AI Configuration Options** (basic version target for Phase 2, full implementation Phase 4):
- **Target Priority**: Closest, lowest HP, highest threat, specific role targeting
- **Range Preference**: Prefer Short/Medium/Long range positioning
- **Action Priority Lists**: Ordered preference per range bracket (e.g., at Short range: use Flame Aura → Attack → Defend)
- **Consumable Triggers**: Use healing potion below X% HP, use buff potion on first turn, etc.
- **Personality Modifiers**: Character personality Traits may override or modify AI behavior (e.g., a Berserker might ignore defensive orders when low HP)

**Phase 1 Minimum**: Basic AI with simple priority rules (attack nearest enemy, move toward nearest enemy if out of range, defend if low HP). Enough to test combat mechanics.

---

## Damage Calculation & Balance Philosophy

### Tunable Category-Based Weighting

**Design Principle**: No fixed formulas upfront. Balance is engineered through iteration and data tracking, not theorycrafting.

**Major Factor Categories** (each with a tunable weight multiplier, default 1.00):
1. Character Attributes
2. Base equipment stats
3. Equipment affix modifiers
4. Status modifiers
5. Passive Perk bonuses
6. Active Action modifiers
7. Global multipliers (e.g., Initiative tick rate at 1.25× for faster combat globally)

**Tuning Process**:
1. Balance within each category (are all Attributes roughly equally valuable?)
2. Balance categories against each other (is equipment too dominant vs. Attributes?)
3. Track win/loss data per Perk/equipment/build to identify outliers
4. Adjust global multipliers for sweeping changes (speed up all combat, increase all damage, etc.)

**Mechanical Hooks Available**:
- Armor penetration (reduce effective Soak)
- Resistance penetration (reduce effective status resistance)
- Type conversion (physical ↔ elemental via Perks/equipment)
- Damage type multiplication (bonus damage against specific types)
- All implemented via Perks, equipment affixes, and statuses — no hardcoded interactions

---

## Equipment System

### Equipment Model

**Core Rule**: A character can equip up to **5 items**. Each item occupies one or more **anatomical slots**. All equipped items are active and provide their full benefits. The 5-item limit constrains loadout breadth; anatomical slots prevent illogical stacking (e.g., two helmets, three pairs of boots).

### Anatomical Slots

| Slot | Count | Notes |
|------|-------|-------|
| Hand | 2 | Main hand, off-hand |
| Head | 1 | |
| Torso | 1 | |
| Arms | 1 | |
| Legs | 1 | |
| Feet | 1 | |
| Finger | 2 | |
| Neck | 1 | |
| Back | 1 | |
| Belt | 1 | |

**Multi-Slot Items**: A single item can occupy multiple anatomical slots but counts as only 1 of the 5 equipment slots.
- Example: Plate Armor occupies Head + Torso + Arms + Legs = 1 equipment slot
- **Advantage**: Powerful consolidated bonuses, fewer pieces to manage
- **Disadvantage**: Fewer total affix sources (you get 5 affixes from 5 items vs. potentially more from separate pieces)
- **Balance**: Multi-slot items tend to have higher base stats but the same affix budget as single-slot items of equivalent quality

**Biological Variations**:
- Missing limbs (from injury or racial anatomy)
- Non-humanoid anatomy (quadrupeds, serpentine bodies, etc.)
- Extra limbs (four-armed species = 4 hand slots)
- No head (oozes, constructs, etc.)
- Anatomical slot availability is defined per-character based on their Core Traits (ancestry)

### Quality System

Quality is a 0–500+ scale that acts as a percentage modifier to an item's effectiveness:

| Quality Range | Star | Color | Modifier |
|---------------|------|-------|----------|
| 0–99 | 0★ | Gray | Below baseline |
| 100 | 1★ | White | 1.0× (baseline) |
| 200 | 2★ | Green | 2.0× |
| 300 | 3★ | Blue | 3.0× |
| 400 | 4★ | Purple | 4.0× |
| 500+ | 5★ | Orange | 5.0×+ |

**Quality Effects**:
- Directly multiplies weapon damage and armor protection
- Example: 60 base damage weapon at 150 quality = 90 effective damage (60 × 1.5)
- Enhances positive item traits, reduces negative ones
- Example: High-quality warhammer has less Speed penalty than a low-quality one

> ❓ **OPEN**: Linear quality scaling (5× at 500 vs 1× at 100) creates an enormous gear gap. A Legendary weapon does 5× the damage of a Common one before affixes. Consider diminishing returns (e.g., square root scaling: quality 400 = 2× instead of 4×) to prevent gear from completely overshadowing character builds. Needs playtesting to determine if this is a problem or an intentional progression lever.

### Durability & Degradation

- **Current Quality**: Drops during combat use (wear and tear)
- **Max Durability**: Equals initial quality rating
- **Repair**: Restores current quality to (max − 1). Each repair permanently reduces max quality by 1.
- **Breaking**: Small chance based on wear ratio (current/max quality). Lower ratio = higher break chance.
- **Quality Forging**: Expensive vendor service to increase max quality (counteracts degradation but is net-negative over long timeframes)

**Design Intent**: Gear is a consumable resource on a long timeline. Even the best equipment eventually degrades and must be replaced. This creates a permanent gold sink and keeps the economy flowing.

> ❓ **OPEN**: For an idle game, relentless degradation may feel punishing rather than interesting. Consider whether certain quality thresholds provide a floor (e.g., gear can't degrade below its star tier baseline), or whether degradation rate should be slow enough that it's a long-term concern, not a per-session anxiety.

### Affix System (Diablo-Style)

**Affix Budget by Quality Tier**:

| Tier | Max Affixes | Per-Affix Power |
|------|------------|----------------|
| 1★ White | 1 | 1.4× |
| 2★ Green | 2 | 1.0× each |
| 3★ Blue | 3 | 0.7× each |
| 4★ Purple | 4 | 0.6× each |
| 5★ Orange | 5 | 0.5× each |

**Design Intent**: Lower-tier items with fewer affixes have individually stronger affixes. A White item with one affix at 1.4× power can be better for a specific build than an Orange item with five affixes at 0.5× each if only one affix matters. This creates genuine itemization decisions at every tier.

**Affix Types**:
- Attribute bonuses (+Strength, +Agility, etc.)
- Damage type bonuses (+Fire damage, +Cold resistance)
- Special effects (life steal, chance to stun, proc effects)
- Perk bonuses (improves specific abilities — requires corresponding Perk to be meaningful)
- Affix potency is also modified by item quality

**Rare Affixes** (Orange 5★ items only):
- "Mythic", "Legendary", "Divine" tier affixes
- Extremely rare, uniquely powerful effects
- Only available on 500+ quality items

### Equipment Tags

Tags are a shared mechanical vocabulary used by weapons, armor, consumables, and Perks to create synergies. Tags can stack on a single item (e.g., a glaive can be both Reach and Two-Handed).

**Weapon Tags**:

| Tag | Examples | Effect |
|-----|----------|--------|
| Reach | Glaives, spears, polearms, staves | Can attack Medium range from Short range; required for Polearm Master Perks |
| Two-Handed | Greatswords, warhammers, longbows, staves | Occupies both hand slots; higher base damage; prevents shield use |
| Light | Daggers, shortswords, hand axes | Faster attack speed; reduced Strength requirements; dodge/parry bonus |
| Heavy | Greataxes, mauls, warhammers | High damage + armor penetration; Speed penalty; Strength requirement |
| Finesse | Rapiers, daggers, whips | Uses Agility instead of Strength for damage; crit bonus; required for Trickster/Duelist Perks |
| Ranged | Bows, crossbows, slings | Can attack Long range; Agility-based accuracy; Short range penalty or inability |
| Thrown | Javelins, throwing axes, throwing knives | Single-use or limited ammo; counts as both melee and ranged |
| Versatile | Longswords, spears, axes | One-handed or two-handed; better stats when two-handed |
| Arcane Focus | Wands, orbs, staves | Required for certain spells; spell power bonus |
| Channeling | Staves, rods, tomes | Reduces casting time; enhances sustained spell effects |

**Armor Tags**:

| Tag | Examples | Effect |
|-----|----------|--------|
| Light Armor | Leather, cloth, hide | No Agility penalty; low protection; preferred by rogues, casters |
| Medium Armor | Chainmail, scale, brigandine | Moderate protection; small Agility penalty; good all-rounder |
| Heavy Armor | Plate, full mail | High protection; significant Agility/dodge penalty; Strength requirement |
| Shield | Bucklers, tower shields, kite shields | Occupies off-hand; block chance or flat damage reduction; may enable defensive Perks |

### Equipment Requirements

Items can require:
- **Attributes**: Minimum Strength, Agility, etc.
- **Traits**: Must possess a specific Trait (e.g., "Requires Arcane Affinity")
- **Perks**: Must know a specific Perk

This prevents characters from equipping gear that doesn't match their build and creates meaningful gating for powerful items.

### Consumables (5 Consumable Slots)

Each character can bring up to 5 consumables into battle. Consumables are generally small-impact unless a build specifically invests in them via specialist Perks.

**Potions**:
- Fast use (minimal Initiative penalty)
- Guaranteed success (no check required)
- Moderate power cap
- Recipes: Blueprints with 1+ effects (e.g., Minor Healing Potion, Major Healing Potion, Vigor Potion)

**Bombs**:
- Fast use like potions
- Offensive effect set (damage, status application, AoE)
- Recipes with 1+ effects
- Tags: Damage types, AoE, etc.

**Scrolls**:
- Slow use (higher Initiative penalty)
- Spell failure chance (requires a check; can fail)
- Much cheaper than potions for equivalent effect
- No power ceiling (high-level effects MUST use scrolls since potion costs scale prohibitively)
- Creation: Characters with Scribe Perk can create Scrolls of any Perk they possess

**Vendor Services** (pre-combat purchases, not carried into battle):
- Even cheaper than scrolls
- No Perk requirements
- One-time service purchases (pre-fight healing, temporary buffs, etc.)

**Consumable Balance**:
- Small baseline impact UNLESS build specializes
- Specialist Perks (e.g., "Bomber" drastically increases bomb effectiveness, "Apothecary" improves potion potency)
- Tags hook into Traits/Perks for synergies

**Acquisition**: Craftable (requires specific Traits/Perks), vendor purchases, loot drops. No expiration.

> ❓ **OPEN**: Consumable quality levels are scaffolded in the design but not implemented initially. Quality may affect potency, success chance, or both.

---

## Time, Economy & Progression

### Time Model

The game advances in discrete **ticks** — periodic update cycles where passive systems resolve.

**Single-Player**: Ticks advance on player command (button press). Play at your own pace.

**Multiplayer**: Ticks are real-time on a server schedule (configurable — could be hourly, every 15 minutes, etc.).

**Each Tick Resolves**:
- Passive resource generation (gold income from support staff, etc.)
- Vendor inventory turnover (new items, characters available for hire)
- Natural healing (injuries recover over time)
- Upkeep deduction (character salaries, facility costs)
- Tournament scheduling (see Tournaments)

**Between Ticks**, players can:
- Spend gold for immediate healing at temples (to keep grinding within the same "day")
- Train characters (spend XP on Attributes and Perks)
- Buy/sell equipment and consumables
- Manage roster (hire, fire, reassign)
- Run PvE encounters (on-demand combat for loot, XP, and gold)
- Register characters for upcoming tournaments

**Design Intent**: This creates a natural idle loop. Check in → review tournament results → make roster/build adjustments → queue for next tournament → optionally grind some PvE → go back to work. The complexity is in decisions between ticks, not during combat.

### PvE Encounters

On-demand combat encounters available between ticks for grinding:
- Set difficulty tiers with scaling rewards
- Training/exhibition style — no injury risk (or reduced risk)
- Loot drops, XP, and gold rewards
- Can be used to test builds before committing to tournaments
- Predefined stages with themed opponents and loot tables

### Tournaments

Structured competitive events at various frequencies and tiers:

**Exhibition Tournaments**:
- Frequent (every tick or on-demand)
- No injury risk (or greatly reduced)
- Smaller rewards
- Good for testing builds, grinding XP

**Championship Tournaments**:
- Less frequent (daily, weekly in multiplayer)
- Full injury risk
- Significant rewards (gold, loot, metacurrency, prestige)
- Elimination or Swiss-bracket format

**Tournament Structure**:
- Multi-round elimination (loses = out)
- When the tournament fires, any unfilled slots are backfilled with NPC-generated teams
- Different tournaments have different average participant levels and entry requirements
- Characters registered for a tournament are committed until it resolves

> ❓ **OPEN**: Can a character participate in multiple tournaments simultaneously? What happens if a character is injured mid-tournament — do they fight injured in the next round, or are they withdrawn? Should there be a "forfeit" option?

### Character Acquisition

**Vendor Types**:

**Arena Recruitment Center**:
- Cheap, low-tier baseline hiring
- Mostly "blank slate" characters (few or no starting Traits/Perks)
- Weighted toward lower star ratings
- No starting equipment

**Guild Vendors** (Pyromancer Guild, Warrior's Academy, etc.):
- Themed character builds
- Guaranteed ≥1 Trait from themed table
- May have additional themed Traits randomly
- Higher cost, better starting builds
- Often paired with themed equipment vendor

**Tavern**:
- Wide variety, pre-existing Traits/Perks/Equipment
- Diverse random generation
- Variable costs based on total character value

**Specialist Vendors**:
- Temple recruits (divine-themed)
- Criminal underground (shadow/trickster-themed)
- Military outposts (warrior/defender-themed)

**Hiring Cost Factors**: Vendor type (baseline) + total character value (Attributes, Traits, Perks, equipment).

### Equipment Acquisition

- **Loot Drops**: From victories (tables based on opponent tier). Quality and affixes randomly generated.
- **Crafting/Forging**: Create new gear (quality varies with crafter skill/Perks), enhance existing gear (quality forging), repair services.
- **Market**: Buy/sell gear. Themed vendors (Pyromancer Guild sells fire-themed equipment). Pricing based on quality + affixes.

### Services & Vendors

| Service | Provider | Function |
|---------|----------|----------|
| Perk Training | Trainers | Spend XP to purchase Perks from owned Traits |
| Attribute Training | Trainers | Spend XP to increase Attributes toward Potential |
| Healing | Temples, Medics | Recover Health, cure injuries |
| Raise Dead | Temples | Resurrect dead characters (scaling cost/penalties) |
| Repair | Blacksmiths | Restore equipment quality (max − 1) |
| Quality Forging | Blacksmiths | Increase max quality (expensive) |
| Affix Manipulation | Enchanters | Re-roll, add, or remove affixes |
| Consumable Crafting | Alchemists | Create potions, bombs from recipes |
| Scroll Copying | Scribes | Create scrolls from known Perks |
| Respeccing | Specialist Vendors | Remove Traits (expensive — encourages roster turnover) |

### Support Staff (Household Hires)

Characters with support Role Traits can provide passive bonuses to the household even while also fighting:

| Role | Benefit |
|------|---------|
| Blacksmith | Reduced repair/forging costs, equipment maintenance |
| Enchanter | Affix manipulation services, quality bonuses on crafted items |
| Trainer | Reduced XP costs for specific Trait families, unlock advanced Perks |
| Accountant/Merchant | Gold generation bonuses, market discounts |
| Quartermaster | Equipment upkeep reduction, inventory management |
| Medic/Priest | Faster injury recovery, reduced healing costs |

**Note**: Same characters can fight AND provide support if their Perks allow both. A Blacksmith who fights during tournaments still provides crafting bonuses between fights.

### Injury & Death System

**Defeat → Injury Check**:
1. Character falls in combat (Health reaches 0)
2. End-of-fight injury resistance check
3. **Pass**: No injury
4. **Fail**: Injury severity based on degree of failure (random severity table)
5. Second check (difficulty based on injury severity)
6. **Pass**: Character survives with injury
7. **Fail**: Character dies

**Injury Types**:
- Persistent negative statuses (until healed)
- Lost limbs (permanent anatomical slot loss unless restored)
- Stat penalties (temporary or permanent)
- Time-limited injuries (heal naturally over several ticks)

**Recovery Options**:

| Method | Cost | Outcome |
|--------|------|---------|
| Natural Healing | Free (over time) | Time-limited injuries only |
| Medical Vendor | Gold | Consistent recovery for common injuries |
| Healing Perks | Character has Perk | Heal specific injury types |
| Severe Injury Specialists | High gold or Bond access | Requires Bond Trait connections or extreme markup |
| Raise Dead (Cheapest) | Great gold cost | Halve all Current Attributes, −1 to all Potentials |
| Raise Dead (Mid-tier) | Very high gold cost | Moderate attribute penalties |
| Raise Dead (Premium) | Extreme gold cost | No permanent penalties |

### Roster Management

- **Upkeep**: Every character has an ongoing upkeep cost deducted each tick
- **Over-hiring → Bankruptcy Risk**: If gold runs out, most expensive contracts leave first
- **Released Characters**: Become free agents (available to other players in multiplayer)
- **Roster Size**: Base limit, expandable via base building

**Base Building** (future system):
- Facilities: Barracks, training grounds, infirmary, forge, etc.
- Upgrades provide bonuses (roster size, training efficiency, healing speed, etc.)
- Lifestyle levels: Adjustable per-character (affects morale, recovery, upkeep cost)

### Metacurrency & Backgrounds

**Metacurrency** (persistent account-level progression):

**Earned Through**:
- Achievements and milestones (primary source)
- Tournament victories
- Persistent play milestones
- Graceful retirement of veteran characters (one source among many, not the primary one)

**Spent On**:
- **Backgrounds**: Starter templates with pre-configured Trait packages
  - "Orc Warrior" → Orc ancestry Trait + combat Traits
  - "Noble Lineage" → Starting gold, reputation bonuses
  - "Battle-Scarred Veteran" → Combat Traits, injury resistances
- Unlocks premium character generation options
- Controlled power creep for recruit quality over time

> ❓ **OPEN**: If the best way to earn metacurrency is to invest heavily in a character and then retire them, players may treat characters as disposable metacurrency farms rather than developing attachment. Consider making retirement one of several metacurrency sources, with achievements and milestones being the primary source, so players don't feel forced to sacrifice characters they like.

---

## Multiplayer Structure

- **Server Types**: Persistent world state servers, scheduled tournaments, asynchronous team management
- **PvP**: Live matches when both players available
- **Async PvP**: Fight against saved team snapshots (opponent doesn't need to be online)
- **NPC Farming**: PvE encounters for XP/loot grinding
- **Tournaments**: Bracketed competitions with rewards, NPC backfill for empty slots
- **Cross-Player Economy**: Trading, free agent hiring from released characters

> ❓ **OPEN**: Matchmaking system (ELO, tier-based, tournament brackets?), leaderboard metrics, season/reset structure, scouting mechanics (can you see opponent builds before a match?), and cross-player economy rules all need design.

---

## Development Phases

### Phase 1 — Core Combat Engine

**Goal**: Playable 3v3 fights that validate the combat math and feel.

- 3v3 fights on 5-zone map
- Basic character generation (Attributes + starting Traits)
- Simple hiring market (gold-based, Arena Recruitment Center only)
- Combat: Attack, Move, Defend + basic Action Perks from starting Traits
- Speed-based initiative system with diminishing returns
- Roll-vs-static-defense combat resolution
- Basic AI (attack nearest, move toward nearest, defend when low)
- Victory rewards: Gold + XP
- Implement minimum Trait set for damage type coverage:
  - Pyromancer (fire), Cryomancer (cold), Storm Caller (lightning)
  - Swordmaster (slashing), Lancer (piercing), Brawler (blunt)
  - Shadowmancer (shadow), Priest (light), Mentalist (psychic), Poisoner (poison)
- **No equipment yet** — pure baseline combat testing
- **Combat spectating**: Turn-by-turn log output in TUI (minimum viable "watching the fight")

### Phase 2 — Build Depth (Skills + Equipment Together)

**Goal**: Full build optimization loop — characters feel meaningfully different based on choices.

**CRITICAL**: Perks and equipment MUST be developed together. Many affixes enhance specific Perks and are meaningless without them.

- **XP Spending**: Train Attributes toward Potential; purchase Perks from Trainers; acquire new Traits
- **Equipment System**: Quality tiers, weapon/armor types with base trade-offs, tag system, prefix/suffix affixes, equipment requirements, multi-slot items
- **Durability/Degradation**: Current quality drops, repair reduces max
- **Consumables**: Potions, bombs, scrolls (5 consumable slots)
- **Loot drops** from victories
- **Basic equipment market**
- **AI Configuration (Basic)**: Target priority and action priority at minimum — players need to influence fights to test builds meaningfully
- **Stealth/Detection**: Sneaking status, Awareness checks

### Phase 3 — Economic Depth

**Goal**: Sustainable economy with meaningful resource tension.

- Support Staff system (Blacksmith, Enchanter, Trainer, Accountant, Quartermaster, Medic)
- Forging/Enhancement services (quality upgrading, affix re-rolling, repair efficiency)
- Injury/Death/Resurrection system
- Metacurrency + Backgrounds (retirement, achievements)
- Vendor variety (Guild vendors, Tavern, temples, forges, enchanters)
- Time model implementation (ticks, passive generation, vendor turnover)
- Tournament structure (exhibition vs. championship, scheduling, NPC backfill)
- PvE encounter system (on-demand grinding with difficulty tiers)
- Reputation system (optional)

### Phase 4 — Tactical Polish

**Goal**: Deep strategic expression and replayability.

- Full AI order customization (range preferences, action priorities per range, consumable triggers)
- Personality system (character-specific behavior modifiers for AI)
- Expanded Trait/Perk library
- Advanced battlefield gimmicks (traps, timed hazards, special zones, variant maps)
- Morale system
- Combat replay/spectating improvements

### Phase 5 — Multiplayer

**Goal**: Shared world with competitive play.

- Server/client architecture split (headless server, TUI client, API for future clients)
- Scheduled tournaments with real-time ticks
- PvP matchmaking (live and asynchronous)
- NPC farming encounters
- Leaderboards and rankings
- Cross-player economy (trading, free agent hiring)
- Seasons/resets (optional)

---

## Outstanding Design Questions

### Combat Resolution (Partially Resolved)
- ✅ Initiative tie-breaking defined
- ✅ Roll-vs-static-defense model defined
- ✅ Movement targeting and forced movement
- ✅ Status effect stacking mechanics
- ✅ Multi-target resolution flags
- ✅ Friendly fire defined per-action
- ✅ No natural in-combat healing (all healing from Perks/consumables/equipment)
- ✅ Action economy: One action per turn baseline, Perks grant exceptions
- ❓ Initiative recovery after action: Always −100, or modified by Action Speed? (Partially defined — Action Speed exists but formula TBD)
- ❓ Status application timing: Before or after damage?
- ❓ Resistance stacking: Additive or multiplicative?
- ❓ Critical hits: Do they exist? How do they work?
- ❓ Overkill/underkill mechanics: Does excess damage matter?
- ❓ Zone-based damage falloff: Range penalties for non-optimal range?
- ❓ Defense scaling: How to prevent Defense from becoming too dominant given flat-subtraction model?
- ❓ Double flat subtraction (attack vs defense, then damage vs soak): Is this too punishing for low-damage builds? Does it create "unkillable" edge cases?
- ❓ Expected match length in turns/ticks: This shapes every balance decision (DoTs, cooldowns, resources, healing). Define a target range.
- ❓ Mirror match handling: How do identical builds resolve in multiplayer?

### Stealth & Detection
- ✅ Sneaking as a contested Awareness check status
- ❓ Does Sneaking break on attack?
- ❓ Can "attack from stealth, re-stealth" loops exist? (If yes, needs heavy Perk gating)
- ❓ AoE interaction with Sneaking characters
- ❓ Detection range: Zone-based or map-wide?

### Morale
- ❓ Is Morale a combat-only stat or does it persist between fights?
- ❓ What triggers affect Morale? (Ally death, taking heavy damage, kills, rallying Perks)
- ❓ Consequences of low Morale: Debuffs? Fleeing? Refusal to act?
- ❓ Is it stack-based (fits existing status system) or a separate system?

### Initiative System
- ✅ Square root Speed scaling as starting model
- ❓ Exact formula: `sqrt(Speed)` per tick, or alternative?
- ❓ Speed floor/ceiling for Initiative calculations
- ❓ Action Speed: How much does it modify the −100 Initiative cost?
- ❓ Haste/Slow effects: Modify Speed directly, or modify Initiative gain separately?

### Perk Action Mechanics
- ✅ Action Speed affects Initiative recovery
- ✅ Cooldown types (turn-based, toggle, conditional)
- ✅ Resource system (Health/Stamina default, Mana/Faith/etc. from Traits)
- ❓ Detailed resource costs per action type
- ❓ Resource regeneration rates
- ❓ Action interrupt mechanics
- ❓ Channeled abilities (multi-turn actions?)

### Equipment Balance
- ✅ 5-item limit with anatomical slot constraints
- ✅ Multi-slot items count as 1 equipment slot
- ✅ Affix budget by quality tier
- ✅ Equipment requirements system
- ✅ Tag stacking allowed
- ❓ Quality scaling: Linear (current) or diminishing returns? Critical for gear-vs-build balance.
- ❓ Multi-slot item power balance: Proportionally stronger base stats, or just simpler?
- ❓ Loot drop rate curves
- ❓ Quality forging cost scaling
- ❓ Degradation rate: Per-fight, per-hit, per-action?

### Consumable Economy
- ✅ Acquisition methods (crafting, vendors, loot)
- ✅ AI usage based on triggers
- ✅ No expiration
- ✅ Scroll creation via Scribe Perk
- ❓ Consumable quality level implementation
- ❓ Specific AI consumable trigger configuration
- ❓ Crafting recipe acquisition (loot? vendor? research?)

### Economy & Pacing
- ❓ Primary gold sink: Which is the main tension — upkeep, repair, training, or hiring? (Other sinks should be tuned around the primary one)
- ❓ XP cost scaling: Linear or exponential?
- ❓ Gold income relative to costs
- ❓ Loot drop rate tuning
- ❓ Leveling speed: Battles per meaningful upgrade?
- ❓ Degradation feel: Does it feel punishing for idle play, or manageable?

### Progression & Endgame
- ❓ Power ceiling: Is there a max character power, or infinite scaling?
- ❓ Endgame content after arena tiers are completed
- ❓ Prestige systems beyond metacurrency
- ❓ Horizontal vs. vertical late-game progression
- ❓ Low-star character viability: Any reason to field 1–2★ characters beyond cost savings?
- ❓ Damage type diversity: Can a "stack one damage type" strategy be countered, or does it always overwhelm single-type resistance?
- ❓ Cross-category Trait synergies: Emergent only, or explicit bonus interactions?

### Tournament Structure
- ❓ Elimination format details (single elimination, double elimination, Swiss?)
- ❓ Injury risk in tournaments: Per-round or final-result only?
- ❓ Can a character be in multiple tournaments simultaneously?
- ❓ What happens to injured characters mid-tournament?
- ❓ Exhibition vs. Championship tier definitions

### Time Model
- ❓ Tick frequency in multiplayer (hourly? every 15 minutes? configurable per server?)
- ❓ PvE encounter availability: Unlimited between ticks, or rationed?
- ❓ Vendor turnover rate: How often do vendor inventories refresh?

### Multiplayer
- ❓ Matchmaking system (ELO, tier-based, tournament brackets?)
- ❓ Team persistence in tournaments
- ❓ Opponent visibility (scouting allowed?)
- ❓ Async vs. synchronous matches
- ❓ Leaderboard metrics
- ❓ Seasons/resets
- ❓ Cross-player economy (trading rules, free agent hiring)

### Content Volume for MVP
- ❓ Minimum Traits per category (Core/Role/Bond)
- ❓ Minimum Perks per Trait
- ❓ Minimum weapon/armor types
- ❓ All 10 damage types in Phase 1, or subset?
- ❓ Zone/battlefield variants in Phase 1?

### Technical Architecture
- ❓ Data format (JSON, SQLite, PostgreSQL?)
- ❓ Procedural generation seeding
- ❓ Save system (single, multiple, cloud for multiplayer?)
- ❓ Server architecture (persistent state storage, match history?)
- ❓ Moddability (design for easy content addition via data files?)
- ❓ Combat log format (for spectating/replay)

---

## Key Design Principles

1. **Modularity**: Systems hook together via tags, requirements, and mechanical interfaces — not hardcoded interactions
2. **Tunable Balance**: Category-based weighting with data-driven tracking, not fixed formulas
3. **Interdependency**: Perks and equipment designed together (Phase 2 priority)
4. **Build Diversity**: Three Trait categories (Core/Role/Bond) create character identity; tags and requirements create build constraints
5. **Economic Pressure**: Upkeep, degradation, injury recovery, and consumable costs create resource tension
6. **Progression Through Turnover**: Expensive respec encourages roster rotation over endlessly perfecting one character
7. **Vendor Access as Progression**: Bond Traits unlock better services, markets, and exclusive content
8. **Dual-Purpose Characters**: Role Traits enable combat AND support capabilities
9. **Stack-Based Status System**: Unified mechanics for buffs, debuffs, DoTs, crowd control — all use stacks with flexible decay
10. **Quality as Universal Modifier**: 0–5★ rating applies to characters, Traits, Perks, equipment
11. **High Number Scaling**: ~50 baseline stats, hundreds/thousands for resources — granular tuning without floating point
12. **Tag-Driven Mechanics**: Shared tag system across weapons, armor, consumables for Perk hooks
13. **Idle-Friendly Design**: Complexity in between-fight decisions, not during combat. Check in, decide, queue, leave.
14. **No Natural Healing**: All in-combat healing is a build choice, not a given

---

## Example Build Archetype: Pyromancer Tank

**Character**: 3★ (3 slots per category)

**Core Traits**:
1. Dragonblood Ancestry (2★) — Fire resistance, +Willpower
2. Stubborn (1★) — Resist knockback, +mental saves

**Role Traits**:
1. Pyromancer (3★) — Fire magic Perks, uses Mana resource
2. Defender (2★) — Defensive Perks, threat generation
3. Blacksmith (1★) — Craft equipment, Strength bonus (support capability)

**Bond Traits**:
1. Pyromancer's Guild (2★) — Fire-themed vendor access, scroll discounts
2. Temple of the Forge God (1★) — Blacksmith service discounts

**Key Perks**:
- Flame Aura (Pyromancer) — Action + trigger (melee hits grant stacks)
- Inferno (Pyromancer) — High-cost AoE fire nuke
- Taunt (Defender) — Force enemies to target you
- Shield Wall (Defender) — Massive soak bonus
- Smith's Strike (Blacksmith) — Armor-ignoring Strength attack

**Equipment (5 items)**:
1. "Molten Warhammer" (Two-Handed, Heavy tags, 300 quality, +Fire damage affix, proc: Burning) — occupies both Hand slots
2. "Dragonscale Plate" (Heavy Armor, 250 quality, +Fire resistance, +Willpower) — occupies Head + Torso + Arms + Legs
3. "Ring of the Forge" (+Strength, reduces equipment degradation) — occupies 1 Finger slot
4. *(2 equipment slots remaining for boots, cloak, second ring, etc.)*

**Consumables (5 slots)**:
1. Major Healing Potion ×2
2. Scroll of Greater Inferno (emergency AoE)
3. Fire Resistance Potion (vs. enemy fire mages)
4. Ironbark Bomb (+armor to allies in zone)

**Playstyle**: Stand in center zone, activate Flame Aura, Taunt enemies into melee range. Gain Flame Aura stacks when hit, use Shield Wall for survivability. Drop Inferno when enemies cluster. Smith's Strike for single-target armor penetration. Out of combat, crafts weapons/armor for team.

**AI Configuration**: Stay in center zone. Priority: Taunt (if enemies not taunted) → Shield Wall (if HP < 50%) → Inferno (if 3+ enemies in zone) → Attack. Use Healing Potion below 30% HP.

---

*This document represents the complete design state. All systems are interconnected via tags, requirements, and mechanical hooks. The document serves as the single source of truth for development and will be iteratively updated as design questions are resolved through prototyping and playtesting.*

