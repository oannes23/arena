# Systems

### Stealth & Detection

**Sneaking State**:
- While Sneaking, character cannot be targeted by single-target or selected-target enemy abilities
- **AoE hits Sneaking characters**: Zone-based AoE effects hit all characters in the zone, including Sneaking ones
- **Break conditions**:
  - Using a non-stealth-tagged Action breaks Sneaking (attacking, casting, etc.)
  - Stealth-tagged Actions can be used without breaking Sneaking (ambush attacks, stealth-specific abilities)
  - Taking damage has a **chance** to break Sneaking: `break_chance = damage_taken / (stealth_stacks × K)` where K is a tuning constant. Linear and transparent — high stealth stacks resist breaking, high damage makes it more likely
  - Being detected by an enemy does NOT automatically break Sneaking — it only allows that enemy to target the Sneaking character

**Detection System** (dual-mode):
- **Passive Detection**: Automatic, same-zone only. Before each character's turn, passive detection re-rolls against Sneaking enemies in their zone using Awareness. No action cost. **Per-turn only** — there is no persistent "this team has detected character X" state between turns. A character who detects a stealth enemy can target them during their turn only; teammates who act later in the same tick must make their own detection checks.
- **Active Detection (Search)**: A deliberate Action that searches the entire map. Detection chance decreases with range: same zone (high), adjacent zones (moderate), distant zones (low). Costs the character's turn but has Action Speed +50 (only −50 Initiative cost). Same per-turn detection persistence — Search results apply to the searching character's turn only.
- **Detection result**: Successfully detecting a Sneaking character allows the detector to target that character during their turn. Detection does not remove the Sneaking status — the Sneaking character retains other stealth benefits (damage break chance, etc.).

### Targeting

Actions use a **tag-based targeting system** (defined in [traits-and-perks](../traits-and-perks/index.md)).

**Core Target Tags** (MVP set — extensible):

| Tag | Resolution |
|-----|-----------|
| `[Self]` | The acting character only |
| `[Enemy, Single]` | One enemy, selected by AI |
| `[Ally, Single]` | One ally, selected by AI |
| `[Enemy, Random]` | One random enemy |
| `[Ally, Random]` | One random ally |
| `[Enemy, AoE-Zone]` | All enemies in a target zone |
| `[Ally, AoE-Zone]` | All allies in a target zone |
| `[All, AoE-Zone]` | Everyone in a target zone (friendly fire) |
| `[Enemy, AoE-Map]` | All enemies on the map |
| `[All, AoE-Map]` | Everyone on the map |
| `[Fallen, Ally, Single]` | One Fallen ally, selected by AI (Revive-only targeting) |

**`[Fallen]` target tag**: Used by Revive-type Actions that exclusively target Fallen allies. Only Actions with the `[Fallen]` tag restrict themselves to Fallen targets.

**Revive on any ally**: The Revive effect component itself works on **any ally target** — if the target is Fallen, the Revive component resurrects them; if the target is not Fallen, the Revive component does nothing but other effect components in the Action still resolve normally. This enables hybrid heal+revive Actions (e.g., `[Ally, Single]` with both Heal and Revive components — heals living allies, revives Fallen ones). Actions that should **only** target Fallen allies use the `[Fallen, Ally, Single]` tag to restrict targeting.

New targeting modes are new tags — no schema changes required. Target tag resolution respects range constraints (each Action specifies its valid range band).

### AoE Resolution

When an AoE Action hits multiple targets, resolution uses a **batch-then-triggers** model:

1. **Batch resolution**: All targets in the AoE resolve through the full combat pipeline (hit roll, crit check, effect resolution, per-component damage) **simultaneously**. A target dying from the AoE does not affect other targets' resolution within the same batch.
2. **Trigger collection**: All resulting Triggers from the batch (OnHit, OnKill, OnDamageDealt, etc.) are collected but not yet fired.
3. **Trigger resolution**: All collected Triggers fire together after the batch completes. Trigger effects from multiple kills resolve simultaneously.

This means an AoE that kills 3 enemies would: (1) resolve damage against all 3 simultaneously, (2) collect three OnKill triggers, (3) fire all OnKill triggers together. A Perk like "Blood Frenzy" (OnKill: gain +10% damage) would gain its stacks from all kills at once.

### Per-Damage-Type Soak

Soak is **per-damage-type**, using three family-level base formulas. Types within a family share the same base Soak value; per-type differentiation comes from Perks and equipment only.

**Family Soak Formulas** (attribute blend weights × per-stat scaling multiplier):
- **Physical Soak** (Blunt, Piercing, Slashing): 60% Endurance + 25% Might + 15% Speed
- **Elemental Soak** (Fire, Cold, Lightning): 45% Endurance + 35% Awareness + 20% Luck
- **Magical Soak** (Poison, Shadow, Light, Psychic): 55% Willpower + 25% Intellect + 20% Luck

**Soak formula**: `Reduction % = Soak[family] / (Soak[family] + K)` where K is a global tuning constant. Higher Soak is always valuable but never grants immunity.

**Penetration**: Universal — one Penetration stat with **diminishing returns** applied against whatever family Soak is being checked. `Effective Soak = Soak[family] - Effective_Pen` (minimum 0), where `Effective_Pen = Pen × K_pen / (Pen + K_pen)` (K_pen is a tuning constant). Same harmonic diminishing returns shape as Soak — high Penetration is always valuable but can never fully zero out Soak. Penetration does NOT affect Shields (only reduces Soak).

*Example with K_pen=100*: Pen 20 → Effective 17. Pen 50 → 33. Pen 100 → 50. Pen 200 → 67. Penetration asymptotically approaches K_pen.

**Per-type bonuses**: Tag-scoped Stat Adjustments (e.g., "+5 Soak vs [Fire]") add to the family base for specific damage types. A character with 50 Elemental Soak and "+10 Soak vs [Fire]" has effective 60 Soak against Fire but 50 against Cold and Lightning.

### Dual Defense System

Two Defense values, each a derived stat:

- **Physical Defense**: 40% Speed + 35% Accuracy + 25% Awareness (× scaling multiplier). Used for physical attacks (Blunt, Piercing, Slashing). Represents dodge, evasion, and reading physical attacks.
- **Magic Defense**: 60% Willpower + 25% Awareness + 15% Luck (× scaling multiplier). Used for elemental and magical attacks (Fire, Cold, Lightning, Poison, Shadow, Light, Psychic). Represents mental fortitude and magical resistance.

The Action's **primary damage type** determines which Defense is checked (see [Combat Resolution Pipeline](resolution.md#combat-resolution-pipeline)). Actions with both physical and magical damage components use the primary type for the single hit/miss roll.

### Combat Event Stream & Deterministic Simulation

**Deterministic simulation**: Combat uses a **seeded PRNG**. Same seed + same inputs = identical replay. The seed is stored alongside the event log, enabling:
- Exact fight replays
- Deterministic testing and bug reproduction
- Verifiable results (anti-cheat, tournament integrity)

**Combat event stream**: The engine emits a stream of **structured typed events** during simulation:
- Each significant combat action produces one or more typed events (e.g., AttackEvent, DamageEvent, FallenEvent, MovementEvent, HealEvent, StatusEvent, DyingBlowEvent)
- Events contain all relevant data (actor, target, values, results)
- The full event stream is stored as the authoritative combat log

**Presentation model**: Combat simulates the entire fight to completion before presenting results. The TUI offers:
- **Dramatized summary**: A condensed narrative of key moments (generated from the event stream, potentially LLM-enhanced)
- **Detailed event logs**: Full tick-by-tick event stream review for players who want depth
- **Combat Scoreboard**: Per-character stat summary (see below)

The specific event type catalog is an implementation detail — the spec establishes that a structured event stream exists and what it enables.

### Resources

**Universal** (all characters):
- **Health**: Fully recovers between battles. **No in-combat regeneration** — all healing comes from Perks, consumables, or equipment procs.
- **Stamina**: General-purpose physical action resource. **Percentage-based passive regen**: X% of max Stamina pool per tick (tuning value) + larger recovery burst from the Defend action (Y% of max pool). When Stamina reaches 0, physical Actions cost Health at a **1:1 ratio** — no penalty multiplier (exhaustion mechanic from [characters](../characters.md)).

**Trait-granted** (via Perk content — see [traits-and-perks](../traits-and-perks/index.md)):
- Mana, Faith, Spirit, Focus, and content-author-defined resource types
- **No default in-combat regen** for Trait-granted resources. Regeneration comes from Perks and Triggers — Perks that define a resource pool often bundle a regen Trigger as part of the same Perk or a companion Perk in the same Trait tree.
- Resource pools start at full capacity at the beginning of combat
- **Resource scale**: Hundreds to thousands for granular tuning (governed by per-stat scaling multipliers)

**Resource Depletion Rules**: Only Stamina has the Health substitution mechanic (Exhaustion — see [characters](../characters.md)). When any other resource pool (Mana, Faith, Spirit, Focus, etc.) is depleted, Actions that require that resource are simply **unavailable** — the `resource_gate` vetoes them. There is no fallback or substitution. This makes resource management a genuine strategic concern for non-Stamina builds.

### Judgment — AI Quality Stat

Judgment is a derived combat stat that controls how effectively a character's AI evaluates and selects Actions during combat. See [combat-ai](../combat-ai.md) for the full Utility AI system.

**Formula**: 40% Awareness + 35% Willpower + 25% Intellect

**Function** — controls three AI parameters:
- **Tactical-vs-Personality blend**: How much the character weighs objective effectiveness vs personality impulses. Range ~0.3 (low Judgment) to ~0.95 (high Judgment).
- **Selection sharpness**: How reliably the character picks the top-scoring Action vs making "suboptimal" choices. Range ~1 (low Judgment) to ~10 (high Judgment).
- **Lookahead depth**: How many ticks ahead the character can project deterministic game-state events (Effects Phase + Initiative timing). Range 1 tick (low Judgment) to 10 ticks (high Judgment). See [combat-ai](../combat-ai.md) for the full lookahead system.

**Modifiers** beyond the base formula:
- Character Star Rating (higher stars = modest baseline boost)
- Specific Perks and equipment affixes
- Status effects (confusion, fear could lower it temporarily)

Judgment uses the same per-stat scaling multiplier model as other derived stats. The exact multiplier and the functions mapping Judgment to blend/sharpness/lookahead values are tuning parameters.

### Combat Context Flags

At the start of each fight, the combat system assembles an explicit **context flags** object and passes it to the AI system. This metadata describes the fight circumstances — tournament round, exhibition status, PvE tier, team sizes, consumable replenishment rules, attrition ramp parameters, etc. Any AI Scorer can query these flags to adjust behavior based on context.

**Canonical schema** (owned by combat.md — AI system consumes read-only):

```
combat_context: {
  is_tournament: bool,
  round: int,                    // current round (1-indexed)
  total_rounds: int,             // total rounds in this event
  consumables_replenish: bool,   // do consumables reset between rounds?
  is_exhibition: bool,
  pve_tier: int | null,          // null if not PvE
  team_sizes: [int],             // list of team sizes, own team first (e.g., [3, 5, 2])
  attrition_ramp_onset: int,     // tick at which attrition ramp activates
  attrition_ramp_rate: float,    // % damage bonus per tick after onset
  ...                            // extensible for future flags
}
```

Context flags are assembled by the combat system from the event configuration. The AI system consumes them read-only. See [combat-ai](../combat-ai.md) for how Scorers use these flags.

### Resource Pool Scaling Multipliers

Each derived stat (including resource pools) has a per-stat **scaling multiplier** that converts the weighted attribute blend into game-scale values.

**Formula-based defaults**: Multipliers are derived from desired gameplay ranges rather than hand-tuned individually. Starting approach:
- Determine desired range for each stat at target attribute levels (e.g., "Health should be 400–800 for a typical 3★ character")
- Back-calculate the multiplier from the blend formula and target range
- Health Pool starts at ×10 as baseline (from [characters](../characters.md))
- All multipliers are tuning values — the formula-based approach sets initial values; playtesting refines them

### Default Actions — Combatant System Trait

Every character has a hidden **"Combatant" system Trait** at level 1. This Trait is:
- **Invisible**: Not displayed in the Trait list UI. Does not occupy a Trait Slot.
- **Immutable**: Cannot be leveled, respecced, or modified.
- **Universal**: Automatically present on all characters (player-owned, Named NPCs, and Ephemeral Combatants).

The Combatant Trait contains a single Perk with the default Actions (Attack, Defend, Move, Search) as standard Perk Action components. This means:
- Default Actions are real Perk components that go through the same resolution pipeline as all other Actions
- No special-case code for "basic attacks" — everything is a Perk-granted Action
- The Combatant Trait's Perk can be extended in future phases to grant additional default capabilities

### Fallen State and Revival

- **Fallen**: When a character's HP drops below 1, they enter the Fallen sub-state (from [characters](../characters.md)). They are out of the fight for its remainder (unless revived).
- **Fallen Resolution is a pure HP check**: At the end of each tick (step 4), the system checks whether HP < 1. If a character was reduced below 0 HP during the tick but healing (from allies, self-heal via Dying Blow, or Triggers) brought them back above 0 HP before step 4 (by an ally's Action, a self-heal on a Dying Blow turn, or a Trigger), they are **not** Fallen. Healing saves from Fallen.
- **Self-save via Dying Blow**: A character at ≤0 HP who takes their turn (Dying Blow) can use a self-heal Action. If the heal brings them above 0 HP before Fallen Resolution, they survive. Consistent rules — no special cases.
- **Revival**: Fallen characters **can** be revived mid-combat via Perks, consumables, or rare equipment effects. A revived character returns at a fraction of their maximum Health (exact fraction is a tuning value).
- **Revival: true clean slate**: **ALL** status effects are cleared on revival — both beneficial and harmful. There is no "buff vs debuff" classification; revival is a complete reset. Buffs applied before falling are lost. Hybrid revive+buff Actions work by applying fresh buffs AFTER the clean slate (via the type-ordered resolution: Revive Step 0 → Status Step 1). The revived character starts fresh — only the fractional HP remains from the pre-Fallen state.
- **Fallen Resolution order**: Characters enter Fallen in **reverse Initiative order** (lowest Initiative first). `OnFallen` Triggers fire sequentially in this order. This means higher-Initiative characters' Fallen Triggers resolve last, which can matter for chain reactions.
- **Status decay frozen while Fallen**: Status effects do not tick down while a character is in the Fallen state. If revived, statuses resume from where they were (but see "clean slate" above — revival clears all statuses, so this only matters for effects that occur between Falling and potential Revival within the same tick).
- **Overkill**: The amount of excess damage beyond 0 HP is tracked. Overkill = `abs(final_HP)` regardless of damage source — DoTs, direct damage, and environmental effects all contribute. Overkill magnitude affects injury severity in the post-combat injury roll — massive overkill increases the chance and severity of injuries.

### Attrition Ramp

An anti-stalemate mechanic that ensures all fights converge to a conclusion:

- **Onset**: After a configurable onset tick (default ~100), the ramp activates. Onset tick is a per-event-type value (specified in [Combat Context Flags](#combat-context-flags)).
- **Effect**: A **stacking global damage bonus** (+0.5–1% per tick, tuning value) applies to ALL combatants once the ramp is active. The bonus increases every tick after onset.
- **Scope**: Damage only — healing is unaffected. This means healing becomes proportionally less effective as the ramp escalates, naturally pushing fights toward resolution.
- **No hard tick limit**: There is no maximum tick count. The exponentially increasing damage bonus makes extended fights increasingly lethal, providing a soft cap on fight length.
- **Per-event-type configuration**: Both the onset tick and escalation rate are specified per event type via Context Flags. PvE fodder encounters might set onset at ~30 ticks with a faster ramp; championship finals might set onset at ~100 with a slower ramp.

**Design Intent**: The attrition ramp replaces the need for a hard time limit. Fights that naturally resolve quickly (mismatched teams, burst-heavy compositions) never trigger the ramp. Stalemate-prone compositions (double tank, pure healer builds) are gradually pushed toward a conclusion. The ramp onset tick serves as the "expected fight length" for a given event type.

### Fight Phase Signal

Combat exposes a **fight phase signal** to the AI system and Triggers:

```
fight_phase = current_tick / estimated_max_ticks
```

Where `estimated_max_ticks` = the attrition ramp onset tick (from Context Flags). This produces a 0.0–1.0+ ratio:
- **< 1.0**: Normal combat phase (before attrition ramp)
- **≥ 1.0**: Attrition phase (ramp is active, fight is running long)

The `resource_efficiency` Scorer in [combat-ai](../combat-ai.md) uses this signal for fight phase detection: conserve resources early (fight_phase < 0.5), balanced mid-fight (0.5–0.8), spend freely late (> 0.8). Triggers can also reference fight_phase for phase-dependent effects.

### Post-Combat Handoff

When victory or defeat is declared (last team standing), combat ends and hands off to the [post-combat](../post-combat.md) system. Combat provides:
- The **Combat Scoreboard** (see below) with per-character stats
- The list of **Fallen characters** with overkill values
- The set of **Traits that were active** during combat (for Perk Discovery rolls)
- The **Combat Context Flags** (exhibition status, event type, etc.)

See [post-combat](../post-combat.md) for the full post-combat flow: injury/death checks, Perk discovery, recruitment, and loot distribution.

### Combat Scoreboard

A structured per-character stat block tracked throughout combat and available to the [post-combat](../post-combat.md) spec and [meta-balance](../../architecture/meta-balance.md) system:

| Stat | Description |
|------|-------------|
| Damage Dealt | Total damage dealt (post-Soak, post-Shield) |
| Damage Taken | Total damage taken (post-Soak, post-Shield) |
| Damage Healed | Total healing done to self and allies |
| Kills | Number of enemies reduced to Fallen |
| Actions Used (by type) | Count of each Action used (e.g., "Flame Burst: 3, Attack: 7") |
| Fallen Events | Number of times this character entered Fallen state (relevant with revival) |
| Revival Events | Number of times this character was revived |
| Dying Blows | Number of Actions taken while at ≤0 HP (before end-of-tick Fallen resolution) |
| Ticks Survived | Number of ticks the character was active (not Fallen) |

The scoreboard is purely observational — it does not affect combat mechanics. It feeds into post-combat Perk Discovery (determining which Traits were "active") and the meta-balance system (tracking win/loss per Trait).

### Zone Effects (Phase 4)

Zone-targeted effects (e.g., "create a fire zone that damages everyone inside") are explicitly deferred to Phase 4. MVP combat uses zones for positioning and range only — no persistent zone-based effects or terrain modifiers.

### Morale (Phase 4)

Fighters have a Morale value affected by combat events. Low Morale may cause debuffs, fleeing, or refusal to act. High Morale may grant bonuses. Full design TBD.

**Crowd Appeal**: The derived stat (60% Charisma + 25% Luck + 15% Awareness, from [characters](../characters.md)) is part of the Morale system, deferred to Phase 4. For MVP, Crowd Appeal exists as a derived stat with no mechanical combat effect.
