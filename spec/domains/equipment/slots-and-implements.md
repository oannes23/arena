# Slots & Implements

## Slots & Equipping

### Anatomical Slot Constraints

- **Decision**: Each item occupies one or more Anatomical Slots. Anatomical Slots prevent illogical stacking and naturally limit loadout size.
- **Rationale**: Anatomical slots alone create sufficient loadout constraints. A humanoid character can fill ~12 anatomical positions. Multi-slot items (like Full Plate covering Head+Torso+Arms+Legs) reduce total item count as a natural consequence.
- **Implications**: Equipment UI shows Anatomical Slot coverage.

### Anatomical Slots (Default Humanoid)

- **Decision**: Default humanoid characters have 11 Anatomical Slots: Head, Torso, Arms, Legs, Hands, Feet, 2x Hand (weapon/shield), 2x Finger, Neck, Back.
- **Rationale**: Full coverage for interesting equipment puzzles. Separating Hands/Feet from Arms/Legs enables granular armor choices. Accessories (Finger, Neck, Back) provide pure affix slots without competing with armor.
- **Implications**: Non-humanoid ancestry Core Traits can modify this set (extra limbs, missing slots, merged slots). Characters spec should define this as the default anatomy.

| Slot | Count | Equipment Types |
|------|-------|----------------|
| Head | 1 | Helmets, Hoods, Coifs |
| Torso | 1 | Chest armor, Robes |
| Arms | 1 | Bracers, Pauldrons, Wraps |
| Legs | 1 | Greaves, Leggings |
| Hands | 1 | Gauntlets, Gloves |
| Feet | 1 | Boots, Sandals, Sabatons |
| Hand | 2 | Weapons, Focuses, Shields |
| Finger | 2 | Rings |
| Neck | 1 | Amulets |
| Back | 1 | Cloaks |

### Equipment Requirements

- **Decision**: Acquisition-only gates. Requirements checked at equip time; removing the requirement source (e.g., respec removing a Trait) does NOT unequip the item.
- **Rationale**: Consistent with [traits-and-perks](../traits-and-perks/index.md) acquisition-only philosophy. Prevents disruptive auto-unequip cascades.

Requirements can include:
- **Attributes**: Minimum values (e.g., Might 60 for Heavy Armor)
- **Traits**: Must possess a specific Trait
- **Perks**: Must know a specific Perk

### Equipment Combat Lock

- **Decision**: Equipment is locked at combat start for **all fight types** (not just tournaments). No mid-combat gear swaps. Degradation is calculated post-combat, not during.
- **Rationale**: Simple, consistent. Equipment decisions happen between fights. No mid-fight affix deactivation from degradation. Equipment is snapshotted at combat start.
- **Implications**: Tournament lock at registration is a separate, stricter constraint (locks between rounds too).

### Item Identification

- **Decision**: All affixes are always fully visible immediately on acquisition. No identification/appraisal mechanic.
- **Rationale**: Respects player time. Players can evaluate loot instantly. Evaluation skill is build synergy recognition, not stat discovery.

---

## Hand Slots & Implements

### Wielding Modes

- **Decision**: 2 Hand anatomical slots with flexible assignment. Two parallel implement categories occupy Hand slots: **Weapons** (physical combat — Sword, Dagger, Mace, Axe, Spear, Greatsword, Greataxe, Warhammer, Bow, Crossbow, Hand Crossbow, Sling) and **Focuses** (magical combat — Wand, Tome, Staff, Orb, Holy Symbol, Artifact, Rod). Shields also occupy a Hand slot. Any combination of implements is valid.
- **Rationale**: Weapons are to physical attacks what Focuses are to magical attacks. Clean parallel system. Maximum build flexibility.

**Implement categories:**
- **Weapons**: Define base damage and Attack/Damage formulas for physical Actions.
- **Focuses**: Define base magic power and formulas for magical Actions. Work identically to weapons mechanically — each focus type defines base magic power, formula weights, damage type, tags, range, and combat role. The magical parallel to physical weapons.
- **Shields**: Defensive items. Block chance + block value. Can pair with either a weapon or a focus.

**Wielding configurations** (any combination of 2 Hand-slot items is valid):
- **Weapon + Shield**: 1 one-hand weapon + 1 shield. Physical combat with defense.
- **Focus + Shield**: 1 one-hand focus + 1 shield. Magical combat with defense.
- **Weapon + Focus**: 1 weapon + 1 focus. **Hybrid build** — physical Actions use the weapon's base stats, magical Actions use the focus's base stats. Off-hand item fires a generic bonus Attack of its type after each Action (see Off-Hand Bonus Attack below).
- **Two-Handed Weapon**: 1 two-hand weapon (Greatsword, Greataxe, Warhammer, Bow, Crossbow). Both Hand slots. Higher base damage.
- **Two-Handed Focus**: 1 two-hand focus (Staff, Tome). Both Hand slots. Higher base magic power.
- **Dual-Wield Weapons**: 2 one-hand weapons. See Dual-Wield rules below.
- **Dual-Wield Focuses**: 2 one-hand focuses. Same dual-wield rules as weapons, applied to magic.
- **Dual Shield**: 2 shields. Valid but unusual. No offensive implement — uses Unarmed/Unfocused defaults.
- **Empty Hands**: Unarmed physical + Unfocused magic. Valid build path with Perks.

**Off-hand restrictions (default):** Off-hand is restricted to **Light-tagged** items by default (applies to both weapons and focuses). Light weapons: Dagger, Hand Crossbow, Sling. Light focuses: Wand, Orb, Holy Symbol, Artifact, Rod. Two-Handed items (Greatsword, Greataxe, Warhammer, Bow, Crossbow, Staff, Tome) use both slots. A Perk (e.g., Ambidextrous) can override this restriction, allowing any one-hand item in the off-hand. Shields are exempt from the Light restriction.

**Versatile**: Applies to both weapons and focuses. Automatically uses two-hand mode if off-hand is empty; one-hand mode if off-hand has an item. Each Versatile type defines **two stat profiles**: one-hand mode (lower base, faster) and two-hand mode (higher base, slower or stronger). The type template carries both profiles.

### Off-Hand Bonus Attack

- **Decision**: When two Hand-slot items are equipped (any combination), each Action triggers a **bonus generic Attack** from the off-hand implement. The bonus attack is always a basic Attack Action — even when the main-hand uses a special Perk Action.
- **Rationale**: Creates satisfying hybrid combat. A warrior with a Sword+Orb uses Power Strike (physical, main-hand) and the Orb automatically fires a generic magic Attack afterward. Reversal: Fireball (magic, off-hand focus) and the Sword fires a generic physical Attack.

**Mechanics:**
- Main-hand Action resolves first with full effects (whether generic Attack or special Perk Action)
- Off-hand fires a **generic Attack** using the off-hand implement's base stats and type (physical for weapons, magic for focuses)
- **Dual-wield penalties apply**: main-hand takes a **−15%** Attack/Damage penalty; off-hand takes a **−35%** Attack/Damage penalty
- Perks can mitigate penalties (Ambidextrous, Two-Weapon Fighting — apply to both weapon and focus dual-wielding)
- **Shield off-hand**: No bonus attack (shields don't have offensive stats)
- **Weapon Triggers fire per-weapon/focus**: Each implement's proc Triggers (OnHit, etc.) fire only on that implement's hit, not on both

**Examples:**
- Sword (main) + Orb (off): Attack → Sword strike (small penalty) + Orb generic magic attack (moderate penalty). Fireball → Orb magic (moderate off-hand penalty) + Sword generic physical attack (small main-hand penalty).
- Wand (main) + Dagger (off): Magic Missile → Wand magic (small penalty) + Dagger generic physical attack (moderate penalty).
- Dagger (main) + Dagger (off): Attack → main Dagger (small penalty) + off Dagger (moderate penalty). Classic weapon dual-wield.

### Implement Resolution

- **Decision**: Actions specify an **implement type** (physical or magical). The system resolves which equipped item provides base stats using a fallthrough: main-hand → off-hand → default.
- **Rationale**: Clean parallel system. Hybrid builds naturally use the right implement for each Action type.

**Resolution order:**
1. Check **main-hand** for a matching implement type (weapon for physical, focus for magical)
2. If main-hand doesn't match, check **off-hand**
3. If neither hand has a matching implement, use the **default**: Unarmed (physical) or Unfocused (magical)

**Examples:**
- Sword (main) + Shield (off): Physical Actions → Sword. Magic Actions → Unfocused default.
- Sword (main) + Orb (off): Physical Actions → Sword. Magic Actions → Orb.
- Wand (main) + Shield (off): Magic Actions → Wand. Physical Actions → Unarmed default.
- Wand (main) + Orb (off): Magic Actions → Wand (main-hand priority). Off-hand Orb provides affixes/procs only.

### Default Attack Adaptation

- **Decision**: A single default **Attack** Action (from the Combatant system Trait) adapts to the main-hand implement type. Not two separate "physical Attack" and "magic Attack" defaults.
- **Rationale**: Simple. One button. The implement determines behavior. Clean parallel to how implement resolution already determines which base stats to use.

**Adaptation by implement type:**
- **Weapon main-hand** → physical Attack (uses weapon base damage, weapon's damage type, weapon's range)
- **Focus main-hand** → magical Attack (uses focus base magic power, focus's damage type, focus's range, focus's targeting pattern)
- **No implement** → Unarmed (physical, Short range, single target)

**The Multihit tag changes default Attack targeting:**
- **Without Multihit** (Wand, Artifact, Rod, all weapons): Attack targets `[Enemy, Single]`
- **With Multihit** (Tome, Orb, Holy Symbol): Attack hits 1–5 enemies in the target zone (see Implement Tags for Multihit mechanics)

**Focus as damage basis:** The equipped focus provides the default damage basis for magical Actions, just like weapons for physical Actions. Individual magical Perk Actions commonly override the focus's default range and/or targeting tags (more frequently than physical Perk Actions override weapon defaults), but still use the focus's base magic power in the damage calculation.

### Unarmed & Unfocused Defaults

- **Decision**: Characters can intentionally fight with empty Hand slots. The Combatant system Trait provides both **Unarmed** (physical default) and **Unfocused** (magic default).
- **Rationale**: Unarmed is a build path, not just a fallback. A Monk character with empty Hand slots and Unarmed-enhancing Perks is a valid and potentially strong build.

**Unarmed**: Physical combat default. Blunt damage, Short range, single target. Might-scaled. Functional but weaker than weapons. Perks (e.g., Monk builds) can enhance to competitive levels.

**Unfocused**: Magical combat default. Weak base magic power. Slightly stronger/less punishing than physical Unarmed. A warrior with no focus can still use a magic Perk, just weakly.
