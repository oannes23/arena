# Decisions Summary

All 8 original open questions resolved plus 95 additional structural decisions (31 rounds total):

| # | Decision | Resolution |
|---|----------|-----------|
| 1 | Quality scaling | **Linear confirmed** — gear multiplies base, but builds provide the base. Squire with Excalibur ≈ Musashi with an oar numerically. |
| 2 | Multi-slot power balance | **Less than proportional** — diminishing returns per slot (~1.0/1.7/2.2/2.6/3.0x). |
| 3 | Loot drop curves | **Event loot tables + enemy drops** — hybrid controlled/thematic system. |
| 4 | Quality forging cost | **Exponential**, Group-gated (e.g., Blacksmith's Guild). MVP 1. |
| 5 | Degradation trigger | **Flat per fight × usage weight** (weapons: Actions taken, armor: damage absorbed). |
| 6 | Quality floor | **No floor, full decay** — items degrade to 0 and are destroyed. Maximum economic pressure. |
| 7 | MVP weapon/armor types | **Moderate** — 12 weapon types, ~18 armor pieces (6 slots × 3 categories) + 3 multi-slot + 3 shields + 3 accessories. |
| 8 | Affix rerolling | **Vendor-dependent** — both single-affix and full-affix rerolling, offered by different Groups. |
| 9 | Block mechanic | **Post-hit flat reduction** — shield items provide block chance % + block value (quality-scaled). |
| 10 | Accessories | **Yes, MVP 1** — Rings, Amulet, Cloak. Pure affix carriers. |
| 11 | Weapon type model | **Weapon type defines all** — base damage, formula weights, tags, range, speed. Items vary by quality + affixes. |
| 12 | Affix formula overrides | **Rare affixes can include Formula Modifier Tags** — item-local attribute substitution (parallels Perk system). |
| 13 | Repair model | **Three tiers** — Cheap (max−2), Standard (max−1), Premium (no loss, very expensive). |
| 14 | Armor stat model | **Hybrid** — flat base Soak (quality-scaled) + attribute-scaling component per armor type. |
| 15 | Resource pool bonuses | **Yes, via affixes** — pre-multiplier pool capacity, same as Perk Stat Adjustments. |
| 16 | Affix generation | **Source-themed tables** — each loot source biases toward thematic affixes. |
| 17 | Affix pools | **Tiered progressive unlock** — 1-2★ basic, 3★ intermediate, 4★ advanced, 5★ rare/unique. |
| 18 | Damage type affixes | **Adds damage component** — +Fire affix adds Fire Damage effect component to Attack action. Per-component Soak resolution. |
| 19 | Item death | **Destroyed** at max durability 0. Clean end-of-life. |
| 20 | Equipment Actions | **Yes** — items can grant Actions (same data model as Perks). Available at 4-5★ affix pools. |
| 21 | Dual-wield | **Two attack rolls** — main-hand and off-hand each make independent Attack rolls. Small penalty on main-hand, moderate penalty on off-hand (Attack and Damage). Perks mitigate. |
| 22 | Versatile mode | **Automatic by off-hand** — two-hand if empty, one-hand if occupied. Each Versatile weapon type defines **two stat profiles** (one-hand and two-hand modes). |
| 23 | Thrown weapons | **Deferred** — will be Consumables when added. |
| 24 | Cursed items | **Yes** — negative affixes alongside positive ones. Identified, no hidden negatives. |
| 25 | Spear range | **Short + Reach tag** — Reaching attacks incur Accuracy penalty. Better than nothing, optimal to close distance. |
| 26 | Armor Speed penalties | **Flat per piece** — each Medium/Heavy piece imposes a flat Speed reduction (Bonus Modifier). Stacks additively. |
| 27 | Armor Soak families | **All armor boosts all three** — different ratios per category (Heavy: Physical-weighted, Light: Magical-weighted, Medium: balanced). |
| 28 | Armor mixing | **Free, no interaction** — each piece applies independently. No set bonuses, no heaviest-dominates. |
| 29 | Inventory | **Household stash (shared)** — all unequipped items in shared pool. No per-character inventory. |
| 30 | Death & gear | **Preserved on corpse** — resurrect = keep gear. Permanent death = gear returns to stash. |
| 31 | Stash decay | **Combat only** — items in stash don't degrade. No time-based passive decay. |
| 32 | Proc effects | **Trigger-based** — proc affixes are equipment-granted Triggers using the combat Trigger system. No new mechanic. |
| 33 | Ranged penalty | **Accuracy penalty at Short** — Ranged weapons take Accuracy penalty at Short range. Mirrors Reach penalty. |
| 34 | Affix data model | **Unified with Perk components** — affixes use exact same data structures as Perks (Stat Adjustments, Triggers, Actions, Damage). |
| 35 | Salvage materials | **Tiered + typed (deferred)** — will have quality tiers and material types. Detailed system deferred; placeholder for now. |
| 36 | Unique items | **Templates** — fixed name/lore/some affixes + 1-2 random slots. Chase items from specific sources. |
| 37 | Selling | **Sell or salvage** — sell for gold (% of value) or salvage for materials (forging/rerolling). |
| 38 | Tournament gear | **Locked at registration** — no equipment changes between tournament rounds. |
| 39 | Unarmed choice | **Always available** — empty Hand slots = Unarmed (Combatant system Trait). Valid build path with Perks. |
| 40 | Affix quality scaling | **Continuous** — `affix_power = base × tier_multiplier × (quality/100)`. Every quality point matters for affix effectiveness, not just tier thresholds. |
| 41 | Block vs multi-component | **Proportional split** — block value is split across damage components proportionally by magnitude. |
| 42 | Dual-wield Action model | **One Action, bonus attack** — main-hand is primary Action (full Initiative cost), off-hand fires a generic bonus Attack. Not two separate Actions. |
| 43 | Weapon Triggers in DW | **Per-weapon** — each implement's proc Triggers fire only on that implement's hit. Independent proc sources. |
| 44 | Cursed item removal | **Free unequip** — no sticky curses. Player always has agency. |
| 45 | Hand slot combos | **Any 2 hand items** — weapon+focus, focus+focus, weapon+weapon, shield+anything. No restrictions on combinations. |
| 46 | Off-hand weapon restrict | **Light only (default)** — Perk override (e.g., Ambidextrous) allows any one-hand item in off-hand. |
| 47 | Forge + affixes | **Two-step: forge + enchant** — forging past tier threshold opens affix slots; separate Enchanting service fills them. |
| 48 | Affix count per item | **Random 1 to max** — affix count is random within 1 to tier max. Fewer affixes = stronger each (inverse budget). |
| 49 | Cursed affix rerolling | **Rerollable** — negative affixes participate in normal rerolling. Curses fixable with gold. |
| 50 | Weapon vs Focus categories | **Parallel categories** — Weapons (physical) and Focuses (magical) are two distinct Hand-slot item types. Staff/Wand/Tome are Focuses, not weapons. |
| 51 | Equipment combat lock | **All fights locked** — equipment snapshotted at combat start for all fight types. Degradation post-combat. |
| 52 | Item identification | **Always visible** — no identification mechanic. All affixes visible on acquisition. |
| 53 | Focus parallel to weapons | **Same mechanical role** — focuses define base magic power and formulas for magic Actions, just as weapons define base damage and formulas for physical Actions. |
| 54 | Unfocused magic default | **Weak default** — magic without a focus = "Unfocused" (slightly stronger than physical Unarmed). System Trait provides both defaults. |
| 55 | Focus MVP types | **7 types: Wand, Tome, Staff, Orb, Holy Symbol, Artifact, Rod** — Staff/Tome two-handed; Wand/Orb/Holy Symbol/Artifact/Rod one-hand Light. (Revised Round 24 — was Staff/Wand/Tome/Orb/Relic.) |
| 56 | Enchanting fills slots | **Yes** — enchanting adds affixes to any empty slot (creation-empty or forge-opened). Sub-max items upgradeable. |
| 57 | Implement resolution | **Fallthrough: main→off→default** — physical Actions check for weapon, magic Actions check for focus. Main-hand priority when both hands match. |
| 58 | Focus tag | **Generic "Focus"** — replaces "Arcane Focus". All focuses carry it. Perk synergies use other tags for specificity. |
| 59 | Off-hand bonus attack | **Always generic Attack** — off-hand fires generic Attack (physical or magic per implement type) even when main-hand uses special Perk Action. |
| 60 | Focus type redesign | **7 types with combat roles** — Wand (Long, sniper, crit bonus), Tome (Long, AoE multi-target), Staff (Medium, high damage, defense bonuses, two-handed), Orb (Medium, multi-target), Holy Symbol (Short, multi-hit, magic defense), Artifact (Long, wild card with per-item overrides), Rod (Short, caster tank, defense bonuses, one-handed). Relic split into Holy Symbol + Artifact. |
| 61 | Focus as damage basis | **Focus = magic weapon for damage** — equipped focus provides default damage basis for magic Actions. Perk Actions commonly override range/tags but use focus base magic power. More frequent overrides than physical Actions. |
| 62 | Default Attack adaptation | **One adaptive Attack** — single default Attack Action adapts to main-hand implement type (physical for weapons, magical for focuses). Multihit tag changes default Attack to AoE. |
| 63 | Implement defense/crit tags | **Defend, Deflect, Deadly tags** — implement tags granting Defense bonuses or crit bonus. Defend = +~10% Attack Value → Physical Defense. Deflect = +~10% Attack Value → Magic Defense. Deadly = crit chance bonus. Apply to both weapons and focuses. |
| 64 | Multihit unified | **Same mechanic** — multi-target and multi-hit are one mechanic called "Multihit." Hits 1–5 enemies in the zone. |
| 65 | Artifact template overrides | **Per-item overrides** — individual Artifact item templates can override base combat role fields (range, tags, targeting, bonuses). Artifact is a category of unique magical implements. |
| 66 | Tag assignments (initial) | **Staff** (Defend+Deflect), **Holy Symbol** (Deflect+Multihit), **Rod** (Defend), **Wand** (Deadly), **Tome** (Multihit), **Orb** (Multihit), **Axe** (Deadly), **Dagger** (Deadly). *(Mace initially Defend here; changed to Sunder in #74.)* |
| 67 | Finesse loses crit | **Deadly is the crit tag** — Finesse tag now means Speed-weighted formulas only. Crit bonus separated into Deadly tag. Weapons wanting both carry both tags (e.g., Dagger: Light, Finesse, Deadly). |
| 68 | Weapon tag expansion | **Dagger** gets Deadly (crits fit rogue fantasy). *(Mace initially got Defend here; changed to Sunder in #74.)* Implement tags apply to both weapons and focuses. |
| 69 | Spear gets Defend | **Reach + Versatile + Defend** — phalanx fighter. Defensive reach weapon. Three tags is fine for a versatile type. |
| 70 | DW defense stacking | **Both contribute** — dual-wielding two Defend/Deflect implements stacks both bonuses. But Attack Value basis already includes DW penalties, naturally limiting stacking. Ambidextrous Perks unlock full-defense dual-wield builds. |
| 71 | Defense is passive | **Always while equipped** — Defend/Deflect bonuses are passive like armor Soak. Apply regardless of hand slot or offensive use. Just holding a Staff grants Defense. |
| 72 | Deadly is flat | **Flat crit bonus** — does not scale with quality. Crit scaling comes from underlying damage already being quality-scaled. |
| 73 | Sword gets Defend | **Versatile + Defend** — sword can parry (has a hilt). Balanced defensive weapon. |
| 74 | Mace → Sunder | **Sunder replaces Defend** on Mace — blunt weapons bypass armor (Penetration), not parry (no hilt). |
| 75 | Sunder tag | **New implement tag** — grants flat Penetration bonus. Effective against armored targets. On blunt weapons (Mace, Warhammer). Does not scale with quality. |
| 76 | Two-handed archetypes | **Greatsword** (Defend), **Greataxe** (NEW, Deadly), **Warhammer** (Sunder). Three distinct two-handed melee identities: defensive, crit-focused, armor-piercing. |
| 77 | New ranged weapons | **Crossbow** (Two-Handed, Ranged), **Hand Crossbow** (Light, Ranged, one-handed ranged), **Sling** (Light, Ranged, blunt at range). 12 weapon types + Unarmed. |
| 78 | Tag-granting affixes | **Any implement tag via affix** — affixes can add Defend, Deflect, Deadly, Sunder, Multihit to any weapon or focus. Common affix pattern. Unique Item templates can also override tags. |
| 79 | *(Duplicate of #67)* | — |
| 80 | Tag assignments expanded | **Sword** (Defend), **Greatsword** (Defend), **Spear** (Defend), **Mace** (Sunder), **Warhammer** (Sunder), **Greataxe** (Deadly), **Dagger** (Deadly), **Axe** (Deadly). |
| 81 | All ranged Long range | **All ranged weapons are Long range** with no Action Speed penalties. Crossbow's higher damage is its identity, not slow speed. |
| 82 | Bow Deadly, Crossbow Sunder | **Bow** gets Deadly (precision crits). **Crossbow** gets Sunder (armor-piercing bolts). Parallels melee differentiation. **Hand Crossbow and Sling** have no implement tags — identity from Light + Ranged. |
| 83 | Hand Crossbow no tags | **Light + Ranged** is sufficient identity. One-handed ranged for off-hand dual-wield is unique enough. |
| 84 | Channeling tag removed | **Channeling removed** from all focus types and tag lists. Was a placeholder with no defined mechanics. Can be re-added if a specific purpose is identified. |
| 85 | Multihit mechanics | **Random 1–5 targets** with triangular distribution (peak 3, weights 1/2/3/2/1). First target chosen normally; additional targets are other hostiles in zone. Additional hits take flat Accuracy penalty, full damage on hit. Capped by valid targets. Hostiles only, no friendly fire. |
| 86 | Focus base power hierarchy | **Parallel to weapons** — Staff/Tome highest (Two-Handed premium). Rod/Orb moderate (one-handed baseline). Wand lower (offset by Deadly). Holy Symbol lower (offset by Deflect+Multihit). Artifact moderate default (overridable per template). |
| 87 | Ranged weapon formulas | **Specific weights locked** — Crossbow: 65% Accuracy + 20% Might + 15% Awareness. Hand Crossbow: 50% Accuracy + 50% Speed. Sling: 50% Speed + 35% Accuracy + 15% Might. |
| 88 | Equipment Slot limit removed | **Anatomical slots only** — removed the 5-item Equipment Slot limit. Characters equip items into Anatomical Slots directly. Loadout size is naturally constrained by available anatomy. |
| 89 | Deadly/Sunder scope | **Per-implement only** — crit bonus (Deadly) and Penetration bonus (Sunder) apply to attacks made with that specific implement. Defend/Deflect remain passive (character-wide while equipped). Clean split: defensive tags = always on, offensive tags = per-weapon/focus. |
| 90 | Action Speed source | **Tag-derived only** — Light (+bonus), Heavy (−penalty), Two-Handed (−small penalty). No per-type individual Action Speed values. Type identity from damage, formulas, and implement tags. |
| 91 | Enchanting theme | **Source-themed** — Enchanting uses the same source-themed affix tables as loot generation. Group vendor theme biases available affixes. Differentiates Group enchanters. |
| 92 | Versatile focuses | **None** — focuses are strictly Light or Two-Handed. Versatile is weapons-only (Sword, Axe, Spear). |
| 93 | Accessory degradation | **Combat-length proportional** — accessories track ticks survived for usage weight. Longer fights = more accessory wear. |
| 94 | DW penalty range | **Main −15%, Off −35%** Attack and Damage. Raw DW ≈ 75% total output (85% + 65%). Perk-gated investment model. |
| 95 | Armor Speed penalties | **Medium −2, Heavy −5 per piece**. Light = 0. Full Heavy (6 pieces) = −30 Speed. Harsh — forces mixed loadouts or Speed investment. |
| 96 | Degradation base rate | **~2 quality per fight** at 1.0× usage. A 100-quality item lasts ~50 fights without repair. Gentle economic pressure. |
| 97 | Range penalty format | **~20% percentage** of Attack Value for Reach at Medium range and Ranged at Short range. Scales with character power — doesn't become irrelevant at high Accuracy. |
| 98 | Multihit penalty format | **~15–20% percentage** of Attack Value per additional target. Consistent with range penalty format. |
| 99 | Economic tuning values | **Fully deferred** to economy spec / implementation — repair costs, forging cost curve, affix rerolling, sell value, enemy drop rates. |
| 100 | Heavy Armor scaling | **55% Endurance + 30% Might + 15% Speed** — tanky, strong, slightly athletic. |
| 101 | Medium Armor scaling | **40% Endurance + 30% Speed + 30% Awareness** — balanced three-way split for jack-of-all-trades. |
| 102 | Light Armor scaling | **60% Speed + 30% Awareness + 10% Endurance** — agile and perceptive with slight toughness. |
| 103 | Characters spec cleanup | **Flag** "5 Equipment Slots" reference for removal — Anatomical Slots only. |
