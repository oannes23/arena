# Vocabularies

## Canonical Vocabularies

These vocabularies define the core set of system-recognized keywords. All are extensible — new entries can be added as content without schema changes.

### Target Tags

See [Targeting](systems.md#targeting) above for the MVP target tag set and resolution rules.

### Trigger Event Types

The authoritative list of events that can fire Perk Triggers.

**Trigger Recursion Guard**: Triggers can chain — a Trigger's effect can cause other Triggers to fire, which can chain further. However, each individual Trigger instance can fire **at most once per originating event**. Implementation: maintain a "fired set" per originating event; skip Triggers already in the set. This prevents infinite loops while allowing complex chains (e.g., OnHit → ApplyBurning → OnStatusApplied → bonus damage → OnDamageDealt → ..., but each Trigger fires at most once).

**Combat Flow Events:**
- `OnCombatStart` — at the beginning of combat (before first tick)
- `OnCombatEnd` — at the end of combat (after last tick)
- `OnTickStart` — at the start of each tick (during Effects Phase)

**Turn Events:**
- `OnTurnStart` — at the start of this character's turn
- `OnTurnEnd` — at the end of this character's turn

**Attack Events:**
- `OnHit` — when this character successfully hits an enemy (Step 1 result ≥ 0)
- `OnCrit` — when this character lands a critical hit
- `OnMiss` — when this character's attack misses
- `OnHitBy` — when this character is hit by an enemy
- `OnCritBy` — when this character is hit by a critical hit

**Damage Events:**
- `OnDamageDealt` — when this character deals damage (after Soak reduction). Fires immediately during the Action.
- `OnDamageTaken` — when this character takes damage. Fires immediately during the Action.

**Kill/Fall Events** (deferred — see [Deferred Kill Trigger Model](#deferred-kill-trigger-model)):
- `OnKill` — when this character drops an enemy to Fallen (fires in Step 5, not immediately). Event payload includes overkill value data.
- `OnAllyFallen` — when a friendly character falls in combat (fires in Step 5)
- `OnFallen` — when this character enters the Fallen state (fires in Step 5)
- `OnDyingBlow` — when this character takes an Action while at ≤0 HP (before end-of-tick Fallen resolution). Fires immediately during the Action.

**Revival Events:**
- `OnRevive` — when this character is revived (fires during Step 0 of effect resolution, immediate trigger). Fires on the revived character.

**Defense Events:**
- `OnBlock` — when this character's shield item successfully blocks (fires at pipeline step 5, joins Action trigger collection)
- `OnShieldBreak` — when a Shield HP pool on this character reaches 0 (fires at pipeline step 7, joins Action trigger collection)
- `OnDodge` — when this character successfully dodges an enemy attack (defender's perspective on a miss, roll < 0). Symmetric with OnMiss. Fires as part of the Action's trigger collection.

**Healing Events:**
- `OnHeal` — when this character heals an ally (healer's perspective, symmetric with OnDamageDealt). Fires immediately during the Action.
- `OnHealedBy` — when this character receives healing from any source (symmetric with OnDamageTaken). Fires immediately during the Action.

**Summon Events:**
- `OnSummon` — when this character creates a summon via the Summon effect component. Fires immediately during the Action.

**Status Events:**
- `OnStatusApplied` — when a status effect is applied to this character
- `OnStatusExpired` — when a status effect fully decays from this character

**Resource Events:**
- `OnResourceDepleted` — when a resource pool reaches zero
- `OnStaminaExhausted` — when Stamina hits 0 (exhaustion trigger)

**Movement Events:**
- `OnMove` — when this character changes zones
- `OnForcedMove` — when this character is pushed/pulled to a different zone

**Stealth Events:**
- `OnStealthBreak` — when this character's Sneaking status is broken
- `OnDetect` — when this character detects a Sneaking enemy

### Trigger Timing Rules

Triggers fire at different points depending on their type:
- **Immediate triggers** (fire during the Action): `OnHit`, `OnCrit`, `OnMiss`, `OnHitBy`, `OnCritBy`, `OnDamageDealt`, `OnDamageTaken`, `OnDyingBlow`, `OnBlock`, `OnShieldBreak`, `OnDodge`, `OnHeal`, `OnHealedBy`, `OnSummon`, `OnStatusApplied`, `OnStatusExpired`, `OnResourceDepleted`, `OnStaminaExhausted`, `OnMove`, `OnForcedMove`, `OnStealthBreak`, `OnDetect`. These fire as part of the Action's trigger collection (batch-then-triggers model for AoE).
- **Revive trigger**: `OnRevive` fires during Step 0 of effect resolution as an immediate trigger on the revived character.
- **Kill triggers** (deferred): `OnKill`, `OnFallen`, `OnAllyFallen` fire in Step 5 of tick resolution (after Fallen Resolution). See [Deferred Kill Trigger Model](#deferred-kill-trigger-model).
- **Flow triggers**: `OnCombatStart`, `OnCombatEnd`, `OnTickStart`, `OnTurnStart`, `OnTurnEnd` fire at their named points in the combat flow.

### Deferred Kill Trigger Model

Kill-related triggers (`OnKill`, `OnFallen`, `OnAllyFallen`) use a **deferred trigger queue** rather than firing immediately:

1. **During Actions**: When a target is reduced to HP < 0, the potential kill is recorded with the causing Action and character. The kill trigger does NOT fire yet.
2. **Step 4 (Fallen Resolution)**: Characters who actually have HP < 1 at tick end are confirmed as Fallen. Characters who were reduced below 0 HP but healed back above 0 before Step 4 do NOT generate kill triggers.
3. **Step 5 (Kill Triggers)**: Confirmed kill triggers fire as a batch. Each kill event tracks the causing Action for attribution. AoE kills from the same Action share a batch.
4. **Recursion guard**: Each kill in Step 5 is a separate originating event for trigger chain purposes (the standard per-event "fired set" applies independently per kill).

**Why deferred?** Immediate OnKill during Actions would fire for targets at HP < 0 who might later be healed back above 0 before Fallen Resolution — creating false kills. Deferring to Step 5 ensures kill triggers only fire for characters who actually enter Fallen.

**Interaction with other triggers**: `OnHit`, `OnDamageDealt`, and all other Action-phase triggers still fire immediately during the Action (unchanged). Only kill-related triggers are deferred.

### Effect Component Types

The authoritative catalog of effect component types used by Actions and Triggers:

| Type | Parameters | Example |
|------|-----------|---------|
| **Damage** | damage_type, formula, [penetration] | Deal 20 [Fire] damage |
| **Heal** | formula, [target] | Restore 30 HP |
| **ApplyStatus** | status_type, stacks, [duration] | Apply 3 stacks of Burning for 5 ticks |
| **RemoveStatus** | status_type, stacks | Remove 5 stacks of Poison |
| **ModifyStat** | stat_key, value, [duration] | Reduce target's Soak by 10 for 3 ticks |
| **Move** | target, direction/zone | Push target 1 zone away |
| **Shield** | formula, [duration], [damage_types] | Grant 50-point shield for 3 ticks |
| **Summon** | template, [zone] | Summon a Fire Elemental in current zone |
| **Revive** | health_fraction | Revive Fallen ally at 25% HP (does nothing on non-Fallen target; other components still resolve) |
| **ResourceDrain** | resource_type, amount | Drain 30 Mana from target |
| **ResourceGrant** | resource_type, amount | Restore 20 Stamina to self |

New types can be added without schema changes — the list format is inherently extensible.

**Resolution within Actions**: Effect components resolve in type order (see [Effect Resolution Order Within Actions](resolution.md#effect-resolution-order-within-actions)).

**Level Scaling**: All numeric output values scale with Perk/Trait level multipliers. Resource costs, cooldowns, and requirements stay flat.

---

## Equipment Combat Mechanics (Cross-Reference)

Several combat-time mechanics are defined in [equipment](../equipment/index.md) and affect combat resolution. This section lists them for implementers; see the equipment spec for full details.

| Mechanic | Summary | See Equipment Spec |
|----------|---------|-------------------|
| **Implement Resolution** | Actions specify physical or magical implement type. Resolution order: main-hand → off-hand → default (Unarmed/Unfocused). | Hand Slots & Implements |
| **Off-Hand Bonus Attack** | Enemy-targeted Actions (damage, debuff) trigger a generic Attack from the off-hand implement as a sequential second pipeline pass. Does NOT fire for heals, self-buffs, Move, Defend, Search. DW penalties: main-hand −15%, off-hand −35% Attack/Damage. | Off-Hand Bonus Attack |
| **Dual-Wield Penalties** | Main-hand −15% Attack/Damage, off-hand −35%. Raw DW ≈ 75% output. Perks (Ambidextrous, Two-Weapon Fighting) mitigate. | Dual-Wield |
| **Armor Speed Penalties** | Medium armor: −2 Speed per piece. Heavy armor: −5 Speed per piece. Full Heavy = −30 Speed. **Mitigated by Might** (see below). | Armor Types |
| **Might Armor Speed Mitigation** | `effective_penalty = base_penalty × (1 - Might/(Might+K))`. High Might reduces armor Speed penalties. See below. | (Defined here) |
| **Versatile Mode** | Versatile weapons auto-switch between one-hand and two-hand stat profiles based on off-hand occupancy. | Wielding Modes |
| **Equipment Combat Lock** | Equipment snapshotted at combat start for all fight types. No mid-combat gear swaps. Degradation calculated post-combat. | Equipment Combat Lock |
| **Default Attack Adaptation** | Single Attack Action adapts to main-hand implement: weapon → physical, focus → magical, empty → Unarmed. | Default Attack Adaptation |
| **Multihit Targeting** | Implements with the Multihit tag change the default Attack to hit 1–5 random enemies in the target zone (triangular distribution). One attack roll, per-target Defense check. | Implement Tags |
| **Reach Range Extension** | Reach-tagged weapons can attack at Medium range (extending from Short) with an Accuracy penalty (~20% Attack Value reduction at Medium). | Weapon Types |
| **Ranged Short Penalty** | Ranged-tagged weapons attack at Long range by default. ~20% Attack Value Accuracy penalty when used at Short range. | Weapon Types |
| **Block Chance** | Shield items (Buckler/Kite/Tower) provide Block Chance % and Block Value in the damage pipeline. See [Block Mechanics](resolution.md#block-mechanics). | Shields |
| **Implement Tags** | Deadly (+crit, per-implement), Sunder (+Penetration, per-implement), Defend (+Physical Defense, passive), Deflect (+Magic Defense, passive), Multihit (AoE default Attack). | Implement Tags |

### Might Armor Speed Mitigation

Might reduces the Speed penalty from Medium and Heavy armor using the same diminishing returns formula used throughout the combat system:

```
effective_penalty = base_penalty × (1 - Might / (Might + K_armor))
```

Where `K_armor` is a tuning constant. This creates a natural Might + Heavy Armor synergy:
- A high-Might warrior in full Heavy armor (base −30 Speed) with Might 150 and K=100 reduces the penalty to about −12
- A low-Might mage in the same armor would suffer nearly the full −30 penalty
- Thematic: strong fighters wear heavy gear efficiently; mages are pushed toward Light armor

This gives Might a unique combat niche beyond weapon damage — it's the "I can wear heavy armor effectively" stat.
