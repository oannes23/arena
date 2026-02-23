# Decisions

## Decisions

### Roll-vs-Static Defense Model

- **Decision**: Attacker rolls, defender provides flat threshold. Two-step: Attack vs. Defense (flat subtraction), then Damage vs. Soak (percentage reduction).
- **Rationale**: All variance on attacker's side makes defenders predictably durable. Flat Defense creates a binary hit/miss gate. Percentage Soak prevents the "armor wall" problem.
- **Implications**: Defense stacking creates hard tier gating (intentional). Soak stacking has diminishing returns. Penetration exists as a counter-stat.

### Percentage Soak with Diminishing Returns

- **Decision**: Soak uses `Reduction % = Soak / (Soak + K)` instead of flat subtraction. Penetration reduces effective Soak before the formula.
- **Rationale**: Flat Soak (the original model) made high-armor characters nearly immune to chip damage. Percentage with diminishing returns means Soak is always valuable but never makes a character invulnerable. Penetration creates meaningful build choices for attackers.
- **Implications**: K is a global tuning constant controlling how quickly Soak reaches diminishing returns. Equipment spec must define Penetration as a valid affix stat. Balance must tune K alongside damage ranges.
- **Alternatives considered**: Flat Soak (original model — rejected due to armor wall problem), threshold-based tiers (rejected — less granular than continuous formula).

### Critical Hit: Extra Damage Roll

- **Decision**: After a successful hit, a separate crit chance roll may add bonus damage. Uses Physical Crit or Magic Crit derived stats.
- **Rationale**: Separate roll after the hit keeps crits exciting without adding variance to the miss/hit determination. Extra damage roll (not a multiplier) means crits are impactful but bounded.
- **Implications**: Awareness, Luck, and Accuracy/Intellect all contribute to crit chance via the derived stat formulas. Crit-focused builds are viable but not dominant.

### Zone-Based Spatial System

- **Decision**: Discrete zones with three range bands (Short/Medium/Long), not grid-based or hex-based. Range is binary — in-range or not, with no damage falloff.
- **Rationale**: Simple enough for auto-battler AI, deep enough for positioning to matter. Zones are easy to represent in TUI. Binary range avoids complex distance math.
- **Implications**: All spatial abilities must work in terms of zones and ranges, not absolute positions or distances.

### Movement Costs Full Turn

- **Decision**: Move is a full action — no free movement by default.
- **Rationale**: Creates positioning tension. Ranged characters have a genuine advantage. "Move + attack" Perks become high-value.
- **Implications**: Melee-only characters need gap-closing Perks or forced movement to compete with ranged. AI must factor movement cost into decisions.

### Initiative: Sqrt Speed × Global Multiplier

- **Decision**: Initiative gain per tick = `sqrt(Speed) × global_initiative_multiplier`, acting at threshold ≥ 100. Global multiplier defaults to ~3.0.
- **Rationale**: Sqrt provides diminishing returns on Speed stacking. Global multiplier is a match-pacing knob independent of character balance — adjusting it speeds up or slows down combat without affecting relative character power.
- **Implications**: At default multiplier 3.0, a character with Speed 100 gains ~30 Initiative per tick → acts roughly every 3–4 ticks. Match length target of 25–150 ticks means most characters act 7–50 times per fight depending on fight type. Speed investment has clear but limited returns. The global Initiative multiplier can be varied per event type to control fight length.
- **Alternatives considered**: Pure `sqrt(Speed)` without multiplier (rejected — too few actions per character per fight), logarithmic scaling (rejected — sqrt provides sufficient diminishing returns).

### Action Speed as Flat Modifier

- **Decision**: Action Speed is a flat modifier to the base −100 Initiative cost after acting. Positive = faster recovery, negative = longer delay.
- **Rationale**: Simple and transparent. Players can easily reason about "this ability costs 70 Initiative instead of 100." Flat modifier avoids percentage-on-percentage complexity.
- **Implications**: Very fast actions (e.g., +50 Action Speed = only −50 Initiative) enable rapid-fire playstyles. Very slow actions (e.g., −50 Action Speed = −150 Initiative) can be balanced with high power.

### Tick Resolution: Effects → Initiative → Turns

- **Decision**: Each tick resolves in fixed order: per-tick effects first, then Initiative accumulation, then turns.
- **Rationale**: Effects-first means DoTs and status decay happen predictably before anyone acts. Initiative-then-turns means all characters accumulate Initiative before any turns resolve, preventing "first mover" issues within a tick.
- **Implications**: A DoT can kill a character before they get their turn on that tick. Status decay happens at a predictable point in the tick cycle.

### Status Before Damage (Type-Ordered Resolution)

- **Decision**: Within a single Action's effect components, status/buff/debuff effects apply before damage effects, then movement effects last. This overrides the "simultaneous" model from traits-and-perks.
- **Rationale**: Enables powerful combo Actions (apply armor shred, then deal damage against reduced armor). Creates interesting design space for single-Action debuff+damage patterns.
- **Implications**: A single Action CAN have an ApplyStatus that reduces Soak followed by a Damage component that benefits from the reduction. Content authors can design "setup-and-exploit" Actions. Traits-and-perks spec updated to reference combat for authoritative resolution order.

### 10 Damage Types, No Inherent Advantages

- **Decision**: 10 damage types across 3 families (Physical, Elemental, Magical) with no inherent rock-paper-scissors. All 10 available in MVP.
- **Rationale**: Advantages come from Perk/equipment interactions and status affiliations, not from hardcoded type matchups. Enables emergent meta via content, not systemic rules.
- **Implications**: All 10 types need at least one Trait and representative Perks in MVP content. Resistance is per-type.

### Resistance: Automatic with Dual Reduction

- **Decision**: Status effects always apply (no binary resist roll). Resistance reduces both the number of stacks applied and damage of that type.
- **Rationale**: "Always applies" means status effects are reliable offensive tools — you never "waste" an ability because of a resist roll. Resistance provides gradual mitigation rather than all-or-nothing, making both attacker investment and defender resistance always meaningful.
- **Implications**: High resistance reduces but never eliminates status effects. Stacking resistance to a specific type is a meaningful defensive investment. Balance must tune resistance curves to prevent either "status immunity" or "resistance uselessness."

### Haste/Slow: Dual Channel

- **Decision**: Speed modification (permanent stat changes) and Initiative gain modification (temporary multiplier) are separate channels that stack.
- **Rationale**: Two channels provide more design space — a Perk can grant temporary haste (Initiative gain buff) without permanently changing Speed. Equipment can modify base Speed. The distinction matters for stacking and dispel interactions.
- **Implications**: "Slow" statuses might reduce Initiative gain (temporary, dispellable) while injuries might reduce Speed (permanent until healed). Both pathways exist for content design.

### Stealth: AoE Hits, Damage Breaks, Stealth-Tagged Actions

- **Decision**: AoE hits Sneaking characters. Taking damage has a chance to break Sneaking. Non-stealth Actions break Sneaking; stealth-tagged Actions do not.
- **Rationale**: AoE as a stealth counter creates tactical counterplay without making stealth useless. Damage-break-chance means even indirect hits threaten stealth. Stealth-tagged Actions enable "assassin" builds that can strike from stealth at the cost of limiting their Action pool.
- **Implications**: Stealth builds need investment in damage avoidance (positioning, allies as shields). AoE abilities have anti-stealth utility beyond raw damage. Stealth-tagged Actions are a distinct tag that content authors can apply to specific Perks.

### Detection: Passive + Active

- **Decision**: Passive detection is automatic and same-zone only. Active detection (Search) is a full-turn Action that searches the entire map with range-based penalties.
- **Rationale**: Passive detection means Sneaking characters in the same zone as high-Awareness enemies are at risk. Active Search provides a dedicated counter-stealth Action at the cost of a full turn — meaningful trade-off.
- **Implications**: AI must decide when to spend a turn on Search vs. other actions. Awareness stat gains utility as the passive detection stat. Search is available via the Combatant system Trait (default Action).

### Unified Stack-Based Status System

- **Decision**: All temporary effects use stacks with per-status decay rules. No separate subsystems for buffs vs. debuffs vs. DoTs vs. CC.
- **Rationale**: Consistent, extensible. One implementation, infinite content variety.
- **Implications**: Every new status effect is just a data definition (per-stack effect + decay model), not new code.

### No Passive In-Combat Healing

- **Decision**: No natural Health regeneration during combat. Stamina has slow passive regen.
- **Rationale**: Makes healing a build investment. Teams must commit resources (Perk slots, consumable slots, equipment affixes) to sustain.
- **Implications**: Matches trend toward attrition without healing investment. Match length tuning (25–150 ticks) accounts for no passive healing — longer fights especially reward healing investment.

### No Default Regen for Trait-Granted Resources

- **Decision**: Trait-granted resource pools (Mana, Faith, Spirit, Focus, custom) have no default in-combat regeneration. Regen comes from Perks and Triggers.
- **Rationale**: Makes resource management a build investment. A Pyromancer who invests in Mana regen Perks has a genuine advantage over one who doesn't. Perks that grant a resource pool often bundle a regen Trigger as part of the same Perk or its Trait tree.
- **Implications**: Characters without regen Perks will deplete their resource pools during combat — this is intended. Resource conservation (using cheaper Actions) becomes a meaningful AI/build decision. Trait tree design should include regen options for the resources they introduce.

### Fallen Revival via Perks/Consumables

- **Decision**: Fallen characters can be revived mid-combat via Perks, consumables, or rare equipment effects.
- **Rationale**: Revival creates strategic depth — investing in revival capabilities is a meaningful team composition choice. Creates dramatic comeback moments. Consistent with Perks-grant-all-capabilities design philosophy.
- **Implications**: Revive is an effect component type in the system. AI must evaluate revival priority. Revival at fractional HP means revived characters are vulnerable.

### Overkill Affects Injury Severity

- **Decision**: Excess damage beyond 0 HP (overkill) is tracked and increases injury severity in the post-combat injury roll.
- **Rationale**: Creates meaningful stakes for getting hit hard. A character barely knocked out vs. one obliterated have different injury expectations. Encourages protecting characters from massive single hits.
- **Implications**: Equipment and Perks that reduce spike damage (shields, damage caps) have indirect injury-prevention value. Tournament strategy must consider opponent's burst damage potential.

### Tiered Injury Checks

- **Decision**: Post-combat injury uses two rolls: first determines if injury occurs, second determines severity. Both are modified by overkill, Endurance, Luck, and Perks.
- **Rationale**: Two-tier system creates more nuanced outcomes than a single roll. Most Falls result in no injury (minor events). When injuries do occur, the severity spectrum (minor → major → critical → death) provides dramatic range.
- **Implications**: Characters built for durability (high Endurance, Luck, injury-resistance Perks) have a meta-advantage beyond just surviving combat.

### Phased Post-Combat Presentation

- **Decision**: Post-combat flow is a sequential dramatic presentation: combat resolution → injury/death → Perk discovery → recruitment → loot.
- **Rationale**: Sequential presentation creates narrative moments and gives each phase appropriate attention. Bulk summaries feel rushed and don't generate player attachment.
- **Implications**: UI must support phased presentation. Each phase can be expanded with detail as the game matures.

### Hidden Combatant System Trait

- **Decision**: Default Actions (Attack, Defend, Move, Search) are provided by a hidden, immutable "Combatant" system Trait at level 1 on all characters.
- **Rationale**: Keeps all Actions in the Perk-Action framework — no special-case "basic attack" code. Hidden Trait doesn't occupy a Trait Slot or appear in UI. Extensible: future phases can add more default capabilities to this Trait.
- **Implications**: Data model must support a hidden Trait type. The Combatant Perk's Actions go through the same resolution pipeline as all Perk Actions.

### Binary Range (No Falloff)

- **Decision**: Abilities are either in-range (full effect) or out-of-range (cannot use). No damage falloff for non-optimal range.
- **Rationale**: Binary range is simplest for both implementation and AI reasoning. Zone-based spatial system already provides meaningful range dynamics through movement costs.
- **Implications**: An archer at Long range deals the same damage as one at Short range (if the Action allows both ranges). Range advantages come from not needing to move, not from damage bonuses.

### Target Match Length: 25–150 Ticks

- **Decision**: Individual fights should last 25–150 ticks. The range reflects fight variety: short PvE fodder encounters (~25 ticks), standard tournament matches (~50–80 ticks), and epic championship finals (~100–150 ticks). Adjusted via the global Initiative multiplier per event type.
- **Rationale**: The original 15–25 tick target was too narrow for the game's fight variety. Short PvE encounters need to feel fast; championship finals need enough ticks for strategic depth, resource management, and dramatic reversals. The wider range gives each fight type its own pacing identity.
- **Implications**: Balance must account for the full range. Cooldowns, resource pools, and status durations should be tuned for the middle of the range (~60 ticks) with acceptable behavior at both extremes. A 5-tick cooldown fires ~5–30 times per fight; a 20-tick cooldown fires 1–7 times. Tournament pacing assumptions must account for longer individual fights.

### MVP Scope: All Types + Default Map

- **Decision**: MVP includes all 10 damage types and the default 5-zone map. Zone variants and gimmicks are Phase 4.
- **Rationale**: All damage types need to exist for the Trait/Perk system to function properly — Traits are themed around damage types. The default 5-zone map is sufficient for validating spatial mechanics.
- **Implications**: MVP content must include at least one Trait per damage type. Zone variants are deferred content, not system work.

### Channeled Abilities and Interrupts: Deferred

- **Decision**: Channeled abilities (multi-turn Actions) and interrupt mechanics are deferred to Phase 2+.
- **Rationale**: The core Action model (one Action per turn, variable Action Speed) provides enough depth for MVP. Channeled abilities add significant system complexity (partial completion, interrupt handling, AI evaluation).
- **Implications**: No multi-turn Actions in MVP. The Action Speed system provides "fast" and "slow" ability pacing without multi-turn commitment.

### Formula-Based Scaling Multiplier Defaults

- **Decision**: Per-stat scaling multipliers are derived from desired gameplay ranges rather than hand-tuned individually.
- **Rationale**: Starting from target ranges (e.g., "Health 400–800 for a typical 3★") and back-calculating multipliers ensures consistent balance foundations. Individual hand-tuning can refine from there.
- **Implications**: Balance work starts with defining target ranges for all derived stats, then computing multipliers. Multipliers remain tuning values that can be adjusted during playtesting.

### Attrition Ramp — Anti-Stalemate Mechanic

- **Decision**: After a configurable onset tick (~100), a stacking global damage bonus (+0.5–1%/tick) applies to all combatants. Healing is unaffected. Onset tick and escalation rate are per-event-type (part of Context Flags). No hard tick limit.
- **Rationale**: Eliminates stalemates without a hard time limit. Fights that resolve naturally never trigger the ramp. Healing-heavy / double-tank compositions are gradually pushed toward resolution. The ramp onset tick doubles as the "expected fight length" signal for AI fight phase detection.
- **Implications**: Attrition ramp parameters must be defined per event type. AI `resource_efficiency` Scorer uses ramp onset as `estimated_max_ticks` for fight phase detection. Balance must ensure the ramp rate is aggressive enough to prevent true stalemates but gentle enough to allow come-from-behind moments in the attrition phase.

### Shield Mechanics — Second Defense Layer

- **Decision**: Shields absorb damage after Soak reduction (second defense layer). Penetration doesn't affect shields. Multiple shields stack; damage drains the oldest first (FIFO). Each shield has its own HP pool and optional damage type filter.
- **Rationale**: Provides a proactive defensive tool distinct from passive Soak. Oldest-first drain means refreshing shields adds HP rather than overwriting. Type filters enable specialized shields (fire ward, poison ward) without making generic shields obsolete.
- **Implications**: Combat pipeline gains a shield absorption step between Soak reduction and Health damage. Shield effect component already exists in the effect type catalog. AI must factor shield HP into target vulnerability assessments.
- **Alternatives considered**: Shields reducing before Soak (rejected — would make Penetration affect shields, blurring the two defense layers), newest-first drain (rejected — would incentivize spamming small shields to protect large ones, creating degenerate play patterns).

### Combat Scoreboard

- **Decision**: Structured per-character stat tracking: damage dealt/taken/healed, kills, Actions used by type, Fallen events, revival events, ticks survived. Available to post-combat spec and meta-balance system.
- **Rationale**: Provides the data needed for post-combat Perk Discovery (determining which Traits were "active"), meta-balance tracking, and player-facing fight statistics. Tracked transparently without affecting combat mechanics.
- **Implications**: Data model must support per-character stat accumulation during combat. Post-combat spec uses scoreboard data for discovery eligibility. Meta-balance spec uses it for win/loss tracking.

### Fight Phase Signal

- **Decision**: Combat exposes `current_tick / estimated_max_ticks` where `estimated_max_ticks` = attrition ramp onset tick. This provides a 0.0–1.0+ ratio for fight phase detection.
- **Rationale**: Gives the AI and Trigger system a clean, normalized signal for fight phase without hardcoding tick thresholds. Values < 1.0 are normal combat; ≥ 1.0 means attrition phase is active.
- **Implications**: AI `resource_efficiency` Scorer uses this for conservation strategy. Triggers can reference fight_phase for phase-dependent effects. The signal is derived from Context Flags (attrition ramp onset), not a separate configuration.

### Win Condition — Elimination Only

- **Decision**: Combat ends when one team is eliminated (last team standing). No timer, no point system, no draw conditions.
- **Rationale**: Simplest model. The attrition ramp ensures fights always converge to a conclusion, eliminating the need for a timer or draw condition. Elimination is the most dramatic and satisfying conclusion for spectators.
- **Implications**: No draw handling needed. Fight length is controlled by the attrition ramp onset and escalation rate, not by a hard timer.

### Global Initiative Order (No Team Alternating)

- **Decision**: Turn order within a tick follows global Initiative order, interleaved between teams. Characters from both teams act in Initiative order — there is no team-alternating structure.
- **Rationale**: Global ordering is simpler and more granular. Speed investment directly determines when you act relative to all combatants, not just your team. Team-alternating would add complexity without strategic depth in an auto-battler.
- **Implications**: A team with multiple high-Speed characters can act consecutively within a tick. Speed stacking has clear individual benefits (act sooner) but no team-turn-order manipulation.

### Percentage-Based Stamina Regen

- **Decision**: Stamina regenerates at X% of max pool per tick (tuning value), not a flat amount.
- **Rationale**: Percentage-based scaling means characters with larger Stamina pools (from Endurance/Willpower investment) regenerate proportionally more, making Stamina investment doubly valuable (more capacity AND more regen). Flat regen would make pool size investment feel wasted.
- **Implications**: Balance must tune the percentage alongside Stamina pool scaling multipliers. The Defend action's Stamina burst is also percentage-based (Y% of max pool).

### Defend — Fixed % Boost + Stamina Burst

- **Decision**: The Defend action provides a fixed percentage boost to Defense and Soak until the character's next turn, plus a Stamina recovery burst equal to Y% of max Stamina pool. Both percentages are tuning values.
- **Rationale**: Percentage-based Defense/Soak boost scales naturally with the character's existing defenses — a tank gets more absolute benefit from Defend than a glass cannon, which is intuitive. Percentage Stamina burst ensures meaningful recovery regardless of pool size.
- **Implications**: AI `self_hp_critical` and `resource_efficiency` Scorers factor Defend's defensive and Stamina benefits. The Combatant system Trait's Defend Perk uses these percentages as base values.

### Exhaustion — 1:1 Health Cost

- **Decision**: When Stamina = 0, physical Actions cost Health at a 1:1 ratio (no penalty multiplier). A 30-Stamina Action costs 30 Health instead.
- **Rationale**: Simple and transparent. No multiplier avoids the complexity of "is it worth paying 1.5× or 2× Health?" — the decision is clear: "do I spend Health to keep fighting?" The 1:1 ratio makes Health the true last-stand resource.
- **Implications**: Characters with high Health pools have more exhaustion runway. AI must factor Health cost into Action evaluation when Stamina is depleted. No special multiplier tracking needed.

### Forced Movement — Unrestricted by Default

- **Decision**: Forced movement is unrestricted (no resistance roll) by default. Fires `OnForcedMove` Triggers. Resistance Perks can reduce or prevent forced movement.
- **Rationale**: Unrestricted default makes forced movement a reliable tactical tool. Resistance as a Perk investment creates build choices for characters who want to hold position (tanks, zone controllers). If forced movement were resisted by default, it would be unreliable and undervalued.
- **Implications**: Content can create resistance Perks ("Immovable Stance", "Rooted") as meaningful investments. `OnForcedMove` Triggers enable reactive abilities (counter-push, damage on forced move). AI must account for forced movement reliability when evaluating push/pull Actions.

### Zone Effects — Deferred to Phase 4

- **Decision**: Zone-targeted persistent effects (fire zones, healing zones, etc.) are explicitly deferred to Phase 4. MVP combat uses zones for positioning and range only.
- **Rationale**: Zone effects add significant system complexity (duration tracking, zone state, interaction with movement) without being essential to the core combat loop. The spatial system provides enough tactical depth from positioning and range alone.
- **Implications**: No persistent zone state in MVP. Zone gimmicks and zone-targeted Actions are Phase 4 content.

### Context Flags Schema Owned by Combat

- **Decision**: The canonical Context Flags field list is defined in combat.md. The AI system (combat-ai.md) references combat.md for the schema and consumes flags read-only.
- **Rationale**: The combat system assembles the context flags from event configuration — it is the natural owner of the schema. The AI system is a consumer, not a definer. Centralizing the schema in one place prevents divergence.
- **Implications**: New context flags are added to combat.md's schema definition. Combat-ai.md's Context Flags section references combat.md. The schema includes attrition ramp parameters (`attrition_ramp_onset`, `attrition_ramp_rate`).

### Post-Combat Flow Moved to Separate Spec

- **Decision**: All post-combat resolution (injuries, Perk discovery, recruitment, loot) is moved to [post-combat](../post-combat.md). Combat.md ends at victory/defeat declared and hands off scoreboard data, Fallen list, active Traits, and context flags.
- **Rationale**: Post-combat is a distinct gameplay phase with its own mechanics, UI flow, and spec dependencies (economy, roster-management, groups). Separating it reduces combat.md's scope and allows post-combat to evolve independently.
- **Implications**: Combat.md no longer owns injury checks, Perk discovery timing, recruitment, or loot. These are now in post-combat.md. Cross-references updated throughout.

### Perk Discovery — Single End-of-Combat Check

- **Decision**: Perk Discovery uses a single end-of-combat check per qualifying Trait, not per-Action/Trigger-use during combat. One discovery roll per Trait. Only Traits that had at least one Action used or Trigger fire qualify ("active Traits").
- **Rationale**: Per-use rolls during combat were implementation-complex (tracking every Action use for discovery) and created a bias toward high-Action-count builds. A single per-Trait roll at end-of-combat is simpler, fairer, and creates a cleaner handoff to the post-combat phase.
- **Implications**: Discovery probability per Trait per fight is higher than the old per-use base rate (retuned to produce similar per-fight discovery rates). The Combat Scoreboard tracks which Actions/Triggers were used, enabling the "active Traits" determination. Traits-and-perks.md updated to reflect the per-Trait model. Full resolution mechanics in [post-combat](../post-combat.md).

### Multi-Team Support with Ranked Placement

- **Decision**: Combat supports 2+ teams of any sizes. "Enemy" = anyone not on your team. Teams are ranked by elimination order (last standing = 1st, last eliminated = 2nd, etc.). Teams eliminated on the same tick share placement rank (next rank skipped).
- **Rationale**: Enables free-for-all, multi-faction, and asymmetric scenarios. Per-team elimination order supports ranked multi-team tournament brackets. Tied placement handles the simultaneous elimination edge case cleanly.
- **Implications**: AI targeting must handle "enemy = not my team" with potentially multiple enemy teams. Tournament formats can use multi-team brackets. Context Flags should include team count. The data model needs per-team elimination tracking.

### Event-Type-Defined Starting Positions

- **Decision**: Starting positions are configured per event type via per-team zone lists (e.g., Team A: [North], Team B: [South]). Characters are distributed within their team's allowed zones by the system.
- **Rationale**: Different event types need different starting configurations — tournaments use opposing zones, PvE encounters might use ambush positioning, free-for-all spreads teams across all zones.
- **Implications**: Event template data model includes starting zone assignments per team. System handles character distribution within allowed zones.

### One Turn Per Tick

- **Decision**: Each character acts at most once per tick. Excess Initiative carries over to the next tick. Tick is the atomic time unit of the combat simulation.
- **Rationale**: Prevents runaway multi-action chains from extreme Speed investment. Very fast characters act in consecutive ticks before others get a turn — appearing to chain actions — but each action occurs in a separate tick. Speed investment is rewarded with frequency, not burst.
- **Implications**: No need for per-tick action limits or anti-infinite-loop safeguards. Initiative overflow naturally balances itself. Balance can focus on per-action effectiveness without worrying about action multiplication.

### Full-Tick Death Deferral and Dying Blows

- **Decision**: When a character's HP drops below 0 (from any source — DoTs, Actions, etc.), they are NOT immediately removed from play. They finish the full tick: remaining effects resolve, they accumulate Initiative, and they can take their turn. Fallen state applies at end of tick. Actions taken while at ≤0 HP are **Dying Blows** — explicitly tracked and available as a Trigger condition (`OnDyingBlow`).
- **Rationale**: Full-tick deferral creates dramatic moments — a character reduced to 0 HP by an attack can still take their turn and potentially eliminate their killer with a Dying Blow. The Dying Blow mechanic rewards Perks that trigger on the edge of death ("Last Stand", "Dying Breath").
- **Implications**: Dying Blows tracked in Combat Scoreboard. `OnDyingBlow` added to Trigger event vocabulary. Content can create "Last Stand" Perks (+100% damage on Dying Blows). AI does not need special death-checking mid-tick — all characters act normally within a tick.

### Starting Initiative: Default Zero with Offset Modifiers

- **Decision**: All characters begin combat with Initiative = 0 by default. Surprise conditions (event-configured) and Perks/affixes can grant an initial offset — either flat (+10) or scaled (+sqrt(Speed) × multiplier = one tick's head start).
- **Rationale**: Default 0 is deterministic and predictable. Offset modifiers create meaningful investment opportunities (a "Quick Draw" Perk that grants starting Initiative) and event-type flavor (ambush events give one team a head start).
- **Implications**: Initiative offset is a new Stat Adjustment target for Perks and equipment. Event templates can specify per-team Initiative offsets alongside starting zone assignments.

### Unified Combat Pipeline with Component-Driven Branching

- **Decision**: All Actions go through the same pipeline. The presence and types of effect components determine which steps apply — no explicit "action type" flag. Actions with enemy-targeted Damage components go through the full attack pipeline; heal/buff Actions skip the attack roll.
- **Rationale**: One code path reduces implementation complexity and bug surface. Component inspection is more flexible than explicit action typing — a mixed Action (damage + heal) naturally resolves the damage through the attack pipeline and the heal directly.
- **Implications**: No "action type" field in the Action data model. Pipeline code inspects effect components to determine flow. A single Action can have components that follow different pipeline paths.

### Per-Component Damage Resolution

- **Decision**: Each Damage effect component resolves through its own complete Soak + Shield + Resistance calculation independently, as if it were a separate hit. A "Flame Blade" with Slashing + Fire resolves each against its respective Soak family.
- **Rationale**: Rewards Actions with diverse damage types — each component hits a potentially different Soak family. Creates interesting build choices for multi-type attacks. Consistent with the per-type Soak system.
- **Implications**: Multi-damage Actions are mechanically stronger than equivalent single-type Actions of the same total damage (they test multiple Soak values). Content balance should account for this. Per-component resolution is more computationally expensive but bounded by component count per Action (typically 1-3).

### Per-Damage-Type Soak with Family Formulas

- **Decision**: Soak is per-damage-type with three family-level base formulas. Physical Soak (60% Endurance + 25% Might + 15% Speed), Elemental Soak (45% Endurance + 35% Awareness + 20% Luck), Magical Soak (55% Willpower + 25% Intellect + 20% Luck). Per-type bonuses via tag-scoped Stat Adjustments.
- **Rationale**: Per-type Soak creates deeper defensive build choices — a character can specialize in physical toughness vs magical resilience. Family formulas keep the formula count manageable (3 vs 10) while per-type bonuses from Perks/equipment add granularity. Each family has a distinct attribute identity.
- **Implications**: Characters have 3 base Soak values tracked. Tag-scoped bonuses add per-type refinement. The K constant in the Soak formula applies universally. Equipment and Perks that grant "+X Soak vs [Fire]" add to the Elemental Soak base for Fire damage specifically.

### Dual Defense: Physical vs Magic

- **Decision**: Two Defense values: Physical Defense (40% Speed + 35% Accuracy + 25% Awareness) and Magic Defense (60% Willpower + 25% Awareness + 15% Luck). Physical family attacks check Physical Defense; Elemental and Magical family attacks check Magic Defense.
- **Rationale**: Creates meaningful differentiation between martial and magical attackers. A character with high Speed (good Physical Defense) may be vulnerable to spells, and vice versa. Each Defense has a distinct attribute identity — Physical Defense is about evasion, Magic Defense is about mental fortitude.
- **Implications**: Physical Defense is a new derived stat (adds to the existing 16). The Action's primary damage type determines which Defense is checked. A "Flame Blade" with Slashing + Fire uses its primary type for the single hit/miss roll — both damage components benefit from the same result.
- **Alternatives considered**: Single universal Defense (original model — rejected because it didn't differentiate martial vs magical attackers), per-type Defense (rejected — 10 Defense values is too complex).

### Primary Damage Type on Actions

- **Decision**: Each Action with Damage components designates a primary damage type. This determines which Defense (Physical vs Magic) is checked in Step 1. All Damage components in the Action share the same hit/miss result.
- **Rationale**: One hit roll per Action keeps combat resolution fast. The primary type reflects the Action's "flavor" — a Flame Blade is primarily a physical attack enhanced with fire, so it checks Physical Defense. Content authors choose the primary type to match the Action's theme.
- **Implications**: Action data model gains a `primary_damage_type` field (required when Action has Damage components). Content authoring must specify primary type for mixed-damage Actions.

### Universal Penetration with Diminishing Returns

- **Decision**: Penetration is a single universal stat with **diminishing returns**: `Effective_Pen = Pen × K_pen / (Pen + K_pen)`. The effective value is then subtracted from Soak: `Effective Soak = Soak[family] - Effective_Pen` (minimum 0). Same harmonic formula shape as Soak and Resistance.
- **Rationale**: Flat Penetration without diminishing returns could completely zero out Soak at high values, creating a balance problem. Diminishing returns on Penetration match the DR on Soak, creating a symmetrical offensive-defensive investment curve. One Penetration value is simpler to track than per-type or per-family.
- **Implications**: Equipment affixes granting Penetration are valuable but with diminishing returns — the first 50 points of Penetration are much more impactful than going from 150 to 200. K_pen is a tuning constant that caps the maximum effective Penetration.
- **Alternatives considered**: Flat subtraction (rejected — allowed complete Soak bypass at high values), percentage-based Penetration (rejected — different semantic from the existing flat model).

### Universal Critical Hits

- **Decision**: Crits apply to ALL effect component types, not just Damage. A crit heal heals more, a crit shield is stronger, a crit buff applies more stacks. Uses the same Physical/Magic Crit stats.
- **Rationale**: Universal crit rewards crit investment for all build archetypes — healers, support, and tanks benefit from crit chance, not just damage dealers. Creates exciting moments for all roles.
- **Implications**: Crit stats (Awareness, Luck, Accuracy/Intellect) are valuable for every character archetype. Content balance must account for crit scaling on heals and shields. AI Scorers should factor crit chance into Action value assessment for all component types.

### Allies Skip Resistance

- **Decision**: Resistance applies only to enemy-sourced effects. Friendly buffs, heals, and beneficial status effects always apply at full strength.
- **Rationale**: Intuitive — your team's abilities help you, enemy abilities are resisted. Avoids the counterintuitive situation where a Fire-resistant character receives reduced benefit from a friendly fire buff.
- **Implications**: Effect source tracking needed (friendly vs enemy). No complex "beneficial vs harmful" tagging on status effects — the system checks source allegiance instead.

### Action-Defined Damage Formulas

- **Decision**: Each Damage effect component has its own `formula` field defining base damage. Weapon attacks reference the equipped weapon's damage; spell Actions define their own formula (e.g., "30 base + 40% Intellect"). This is already part of the effect component spec but now explicitly confirmed.
- **Rationale**: Action-defined formulas give content authors full control over damage scaling. A fire spell scales with Intellect; a physical attack scales with Might. No implicit assumptions about damage sources.
- **Implications**: The `formula` field on Damage effect components is the canonical source of base damage. Weapon damage is one possible input to a formula, not the only one.

### Deterministic Simulation with Seeded RNG

- **Decision**: Combat uses a seeded PRNG. Same seed + same inputs = identical replay. The seed is stored alongside the combat event log.
- **Rationale**: Enables exact fight replays, deterministic testing, bug reproduction, and verifiable results (tournament integrity, anti-cheat). Essential for the simulation-first presentation model.
- **Implications**: All random calls in combat must use the seeded PRNG (no system random). Seed is part of the combat record. Replays are byte-identical.

### Structured Combat Event Stream

- **Decision**: The combat engine emits structured typed events during simulation. Combat simulates the entire fight to completion, then presents results via dramatized summary and/or detailed event logs.
- **Rationale**: Simulation-first enables the MUD-style TUI to present combat in whatever format suits the player — quick summary, LLM-dramatized narrative, or tick-by-tick review. Structured events are the authoritative combat record, enabling replays, statistics, and presentation flexibility.
- **Implications**: Event type catalog is an implementation detail (not locked in spec). Combat engine produces events as output, not real-time rendering. TUI consumes events for presentation. Event logs + seed enable exact replay. LLM summarization is a presentation layer feature.

### Batch-Then-Triggers AoE Resolution

- **Decision**: AoE Actions resolve all targets through the pipeline simultaneously (batch), then all resulting Triggers fire together after the batch completes.
- **Rationale**: Clean separation between "what happens" (batch resolution) and "what it causes" (triggers). A target dying mid-AoE doesn't affect other targets. All OnKill triggers from a multi-kill AoE fire together, enabling consistent trigger stacking.
- **Implications**: AoE implementation resolves damage against all targets before checking triggers. "Blood Frenzy" (OnKill: +damage) would stack from all kills simultaneously. No ordering dependency between AoE targets.

### Weapon-Type-Defined Attack & Damage Formulas

- **Decision**: Each weapon type defines its own attribute blend formula for Attack Value and Damage Value. There are no fixed "Physical Attack" or "Magic Attack" derived stats. Spell Actions define their own formulas independently in their Damage component's `formula` field.
- **Rationale**: Content-driven formulas give maximum flexibility. A dagger uses Speed, a warhammer uses Might, a staff uses Intellect. Each weapon type has its own gameplay identity without hardcoded derived stats. Spells are fully independent of weapons.
- **Implications**: The combat system needs a formula evaluation engine that resolves weapon/spell formulas dynamically. Equipment spec must define per-weapon-type attribute formulas as content data. No fixed Attack derived stat in the characters spec.

### Zone Adjacency: Center + Ring Topology

- **Decision**: Default 5-zone map uses center + ring adjacency. Center is adjacent to all four cardinals. Cardinals are adjacent to ring neighbors (N↔E, E↔S, S↔W, W↔N). Opposite cardinals (N↔S, E↔W) are NOT adjacent = Long range.
- **Rationale**: Creates meaningful Long range on the default map — ranged characters can target across opposite sides. Center is a contested hub accessible to all. Ring adjacency provides multiple movement paths between any two zones.
- **Implications**: Long range Actions ARE useful on the default map (across N↔S or E↔W). Center is the most connected zone (adjacent to all 4), making it valuable territory. Movement from N to S requires 2 moves minimum (through C, E, or W).

### Attribute-Scaled Unarmed Combat

- **Decision**: Characters with no weapon equipped have a baseline unarmed attack (Blunt damage, Might-scaled). Functional but naturally weaker than any weapon. Perks can enhance unarmed combat (Monk-style builds).
- **Rationale**: Ensures every character can participate in combat even without equipment. Might-scaling makes physical prowess the natural unarmed damage source. Perk-enhanced unarmed creates a viable build archetype.
- **Implications**: The Combatant system Trait's Attack action must support both weapon-based and unarmed resolution. Unarmed Perks could grant new Attack actions with better formulas, tags, and effects.

### Context Flags: Team List for Multi-Team

- **Decision**: Replace `team_size`/`opponent_team_size` with `team_sizes: [int]` — a list of team sizes with own team first. AI derives all needed information from the list.
- **Rationale**: A list naturally supports 2+ teams. Own team first is a clean convention. AI can calculate total enemies, team count, outnumbered status, etc. from the list.
- **Implications**: Context Flags schema updated. AI Scorers that reference team size must use the list format.

### Formula Modifier Tags (Attribute Substitution)

- **Decision**: Perks can apply formula modifier tags — a Stat Adjustment subtype that substitutes one attribute for another in formula evaluation. When multiple modifiers target the same slot, the **highest attribute value wins** (player always benefits from their best option).
- **Rationale**: Formula modifiers enable "Intelligent Strikes" (Intellect for Accuracy) and "Charismatic Hitter" (Charisma for Might) without creating new Action variants. Highest-value-wins conflict resolution is simple and always benefits the player.
- **Implications**: Formula evaluation must check for active modifier tags and resolve conflicts. This is a new Stat Adjustment subtype alongside flat bonuses and tag-scoped bonuses. The data model needs a formula modifier representation.

### DoTs Bypass Soak

- **Decision**: DoT damage (from status effect ticks during the Effects Phase) bypasses Soak entirely and is applied directly to Health. Shields still absorb DoT damage. Resistance reduces stack count at application time.
- **Rationale**: DoTs already passed through Resistance when initially applied — double-gating through both Resistance and Soak would make them too weak. Shields proactively protecting against DoTs is a valid strategy and creates build choices.
- **Implications**: The Effects Phase damage path skips Soak calculation for status-sourced damage. Shield absorption still applies (Shields sit between Soak and Health). This makes Resistance the primary defense against DoTs, and Shields the proactive defense.
- **Alternatives considered**: DoTs through Soak (rejected — too much defensive gating, DoTs become irrelevant), DoTs bypass both Soak and Shields (rejected — no proactive defense against DoTs).

### Passive Detection Before Each Turn

- **Decision**: Passive stealth detection happens immediately before each character's turn in the Turns Phase, not once per tick. Characters act with the most current detection information.
- **Rationale**: More granular detection means an ally's turn that reveals a stealth enemy immediately benefits the next ally to act. Creates natural team coordination for stealth counterplay.
- **Implications**: Passive detection is not part of the Effects Phase — it's per-character in the Turns Phase. This adds a detection check before each turn but the check is simple (same-zone Awareness comparison).

### Summons as Ephemeral Combatants

- **Decision**: The Summon effect component creates Ephemeral Combatants with their own Initiative, preset AI, and team membership matching the summoner. They disappear when killed or when the summoner enters Fallen. They do not trigger post-combat phases.
- **Rationale**: Using the existing Ephemeral Combatant model (from characters spec) avoids special-case code for summons. Independent Initiative means summons act in global turn order like everyone else. Summoner-death-kills-summons prevents orphaned entities.
- **Implications**: Summon templates need to define stat blocks, AI configurations, and duration. Fallen resolution must check for summoner-linked entities. Post-combat explicitly excludes summons from injury/discovery/recruitment.

### Stun Prevents Initiative Gain

- **Decision**: Stun stacks prevent Initiative gain entirely (0 gain while stacks > 0). Stunned characters never reach Initiative ≥ 100, so they never get turns. Stun stacks decay proportionally to Willpower per tick.
- **Rationale**: Hard stun is the most impactful CC — losing turns entirely is devastating. Willpower-based decay provides a clear counter-stat and prevents permanent stun-lock. High Willpower characters shrug off stun quickly; low Willpower characters are vulnerable.
- **Implications**: Stun is the most powerful status effect — content must price it accordingly (high resistance, short duration, limited access). AI must factor stun vulnerability into target selection. Stun immunity Perks would be high-value defensive investments.

### Revival: Clean Slate

- **Decision**: All status effects are cleared on revival. A revived character starts fresh — no lingering debuffs, DoTs, or buffs from before they fell. Only the fractional HP remains.
- **Rationale**: Clean slate is simpler (no "which statuses survive?"), avoids punishing revived characters with pre-death debuffs, and creates a clear mental model: revival = fresh start at low HP.
- **Implications**: Revival timing matters less from a debuff perspective (no benefit to waiting for debuffs to expire before reviving). Buff-then-revive combos are not viable — the buff would be cleared. Content design should not rely on pre-Fallen status persisting through revival.

### Summon Control Capacity Budget

- **Decision**: Summoner Traits grant a Resource Pool acting as a control capacity budget (e.g., "Necro Control" = 0.8×Willpower + 0.2×Charisma). Each summon type has a control cost. Summoners maintain summons up to capacity; dying summons free capacity for new summons.
- **Rationale**: Budget-based control is more flexible than per-Perk caps. A summoner can choose many weak summons or few strong ones. Tying capacity to a Resource Pool means it scales with attributes and can be modified by Perks/equipment. Dying summons naturally free capacity, creating dynamic summoning decisions.
- **Implications**: Summoner Traits need a control capacity Resource Pool definition. Summon templates need a control cost field. AI must factor remaining capacity into summoning decisions. Balance must ensure capacity pools and summon costs produce reasonable summon counts (e.g., 2-5 active summons for a dedicated summoner).

### Mutual Elimination is a Draw

- **Decision**: When all remaining teams are eliminated on the same tick, all share 1st place (draw). Consistent with the existing same-tick shared rank rule.
- **Rationale**: The shared rank rule already handles same-tick elimination cleanly. Mutual elimination is just the extreme case where all teams eliminate each other simultaneously. No special handling needed — the general rule covers it.
- **Implications**: Tournament formats must handle draw outcomes (shared 1st). Post-combat flow must handle the case where no team "won" outright.

### Non-Stamina Resource Depletion: Action Unavailable

- **Decision**: Only Stamina has the Health substitution mechanic (Exhaustion). When any other resource pool (Mana, Faith, Spirit, Focus, etc.) is depleted, Actions requiring that resource are simply unavailable — the `resource_gate` vetoes them. No fallback or substitution.
- **Rationale**: Stamina's Health substitution creates dramatic last-stand moments for physical fighters. Extending this to all resources would eliminate resource management as a strategic concern. Mana depletion means a mage must fall back on default Actions or cheaper alternatives — that's a meaningful build constraint.
- **Implications**: AI `resource_efficiency` Scorer must conserve non-Stamina resources more aggressively (no safety net). Builds relying on a single non-Stamina resource need regen Perks or will hit a hard wall. Default Actions (Attack, Move, Defend, Search) remain available since they don't cost non-Stamina resources.

### Crit Bonus: Percentage of Base

- **Decision**: Crit bonus formula: `output = base × (1 + crit_multiplier)`. The `crit_multiplier` is a tuning value (e.g., 0.5 for +50% bonus on crit). This applies universally to all effect component types.
- **Rationale**: Percentage of base scales naturally with ability power — a strong ability crits for more than a weak one. Single multiplier tuning value is simple to balance. Avoids the variance of a separate die roll and the complexity of flat-per-component bonuses.
- **Implications**: Crit multiplier is a new tuning value. Perks/equipment that increase crit multiplier are powerful (scale with ability strength). Balance must ensure that crit builds with high-base-damage abilities don't become dominant.

### Revive Targeting: Fallen Target Tag

- **Decision**: A new `[Fallen]` target tag allows Actions to exclusively target Fallen allies (e.g., `[Fallen, Ally, Single]`). The Revive effect component itself works on any ally — on non-Fallen targets, the Revive component does nothing but other components still resolve. This enables hybrid heal+revive Actions.
- **Rationale**: Two complementary mechanisms: the `[Fallen]` tag for Revive-only Actions, and Revive-on-any-ally for hybrid Actions. A "Healing Light" Action with both Heal and Revive can target any ally — it heals the living and revives the Fallen. A "Resurrect" Action with only Revive uses `[Fallen, Ally, Single]` to restrict to Fallen targets.
- **Implications**: Target tag vocabulary gains `[Fallen]`. AI `revival_priority` Scorer evaluates both Fallen-only and hybrid heal+revive Actions. Content authors choose between dedicated and hybrid Revive Actions based on design intent.

### Revived Summoner: Summons Stay Dead

- **Decision**: If a summoner is revived mid-combat, their previously active summons remain dead. The summoner must re-summon using their restored control capacity.
- **Rationale**: Summons are removed during Fallen resolution (when the summoner falls). Revival is a fresh start — the summoner returns with clean slate (no statuses, no summons). Automatically restoring summons would be complex and potentially too powerful.
- **Implications**: Reviving a summoner is less immediately impactful than reviving a direct combatant (the summoner needs turns to rebuild their summon army). AI must factor re-summon cost into revival priority for summoner characters.

### Fallen Status Decay: Frozen

- **Decision**: Status effects do not tick down while a character is in the Fallen state. If a Fallen character were to be affected by any mechanism, their statuses remain at the stacks they had when they fell.
- **Rationale**: Frozen decay is simpler than continuing to process statuses on inactive characters. Since revival clears all statuses (clean slate), frozen decay's primary purpose is consistency — the system doesn't need to track or process status decay for Fallen characters.
- **Implications**: Minimal implementation impact — Fallen characters are simply skipped during the Effects Phase. Clean slate on revival makes frozen decay moot in practice, but the rule exists for completeness.

### Healing Saves from Fallen

- **Decision**: Fallen Resolution (step 4 of tick resolution) is a pure HP check. If a character was reduced below 0 HP during the tick but healing brought them back above 0 HP before step 4, they do not enter Fallen.
- **Rationale**: Pure HP check is the simplest rule — no tracking of "was this character ever at 0 HP?" Just check current HP at tick end. This means healing is a genuine life-saving tool, not just damage mitigation.
- **Implications**: Fast healers (high Initiative, acting before Fallen Resolution) can save allies who were hit earlier in the tick. Creates dramatic save moments. Self-healing on Dying Blow turns can prevent the character's own death.

### Resistance Formula: Percentage with Diminishing Returns

- **Decision**: Both stack reduction and damage reduction use the same formula: `value × (1 - resist_ratio)` where `resist_ratio = Resistance / (Resistance + K)`. Same K per type for both. Parallels the Soak formula shape.
- **Rationale**: Using the same diminishing returns curve as Soak creates system consistency — players learn one formula shape and understand both defense mechanisms. Per-type K allows tuning individual resistances without affecting others.
- **Implications**: K is a per-type tuning constant (or possibly a single global constant like Soak). Balance must tune K alongside stat ranges. Very high Resistance approaches but never reaches immunity. Minimum 1 stack always applies regardless of Resistance.

### Stealth Break: Damage/Stealth Ratio

- **Decision**: Stealth break chance on taking damage uses `break_chance = damage_taken / (stealth_stacks × K)` where K is a tuning constant.
- **Rationale**: Linear formula is transparent and easy to reason about. More stealth stacks = harder to break. More damage = more likely to break. K controls the overall sensitivity.
- **Implications**: Stealth break K is a new tuning value. High stealth investment (many stacks) provides genuine protection against chip damage. AoE that deals low damage per target is less likely to break stealth than a big single hit.

### Resistance Damage Reduction: Same Formula, Same K

- **Decision**: Damage reduction from Resistance uses the same `damage × (1 - resist_ratio)` formula and the same K as stack reduction for that type.
- **Rationale**: One K per type simplifies tuning. If Resistance 40 reduces applied stacks by 44%, it also reduces incoming damage of that type by 44%. Players can reason about resistance strength from a single number.
- **Implications**: No separate damage-reduction tuning per type — stack and damage reduction are always in lockstep. This means high Resistance provides comprehensive defense against a damage/status type.

### Minimum Stacks: Floor of 1

- **Decision**: Status effects always apply at least 1 stack, even after Resistance reduction. Resistance can reduce 10 stacks to 1 but never to 0.
- **Rationale**: Minimum 1 stack ensures status effects are reliable offensive tools. An attacker who lands a status-applying ability always gets *something* through, even against high-Resistance targets. This prevents Resistance stacking from creating effective immunity to statuses.
- **Implications**: Even maximally Resistance-stacked characters are affected by statuses (at reduced potency). Content balance can rely on statuses always having some effect. The 1-stack floor is a hard rule, not a tuning parameter.

### Attack Costs Stamina

- **Decision**: The default Attack action has a small Stamina cost (tuning value). Move, Defend, and Search are free (no resource cost).
- **Rationale**: A small Stamina cost on Attack creates Stamina pressure even for characters relying on the basic attack — they can't attack indefinitely without eventually triggering Exhaustion. This makes Defend (which recovers Stamina) a meaningful choice even for simple builds. Move, Defend, and Search being free ensures characters always have zero-cost options.
- **Implications**: Default Attack Stamina cost is a new tuning value. AI must factor this cost into Attack evaluation via `resource_efficiency`. Characters with very low Stamina pools will feel Exhaustion pressure faster from basic attacks alone.

### Formula Representation: Weighted List + Optional Expression

- **Decision**: Formulas use a structured `{base, weights}` format for the common case (~95% of formulas) and an optional `formula_override` expression string for complex Perks/spells.
- **Rationale**: Structured format is machine-inspectable — AI Scorers can read weights, the balance system can analyze formulas, and formula modifier tags can substitute attributes. Expression override provides an escape hatch for complex mechanics without compromising the simple common case.
- **Implications**: Formula evaluation engine must support both formats. Content authoring tools should default to the structured format and only expose expression override for advanced use cases. Modifier tags only apply to the structured format (expression overrides handle their own attribute references).

### DoT Overkill Counts

- **Decision**: Overkill = `abs(final_HP)` regardless of damage source. DoT damage, direct damage, environmental effects — all contribute to the overkill value used for post-combat injury severity.
- **Rationale**: Consistent rule — damage is damage regardless of source. A character who burns to death from DoTs should have the same injury risk as one killed by a sword strike of equal excess damage.
- **Implications**: DoT-heavy builds can generate overkill just like burst damage builds. Characters under heavy DoTs who are about to fall should be healed or shielded to reduce overkill, creating tactical depth.

### Self-Save via Dying Blow

- **Decision**: A character at ≤0 HP who takes their turn (Dying Blow) can use a self-heal Action. If the heal brings them above 0 HP before Fallen Resolution (step 4), they survive. Consistent rules — no special cases.
- **Rationale**: If Fallen Resolution is a pure HP check, then any healing that brings HP above 0 should prevent Fallen — including self-healing on a Dying Blow turn. Special-casing self-saves would add complexity for no gameplay benefit.
- **Implications**: Characters with self-heal abilities have a natural survival advantage on Dying Blow turns. AI must evaluate self-heal as a high-priority option during Dying Blow turns. Content can create "Second Wind" style Perks that are specifically designed for self-save scenarios.

### Formula Modifier Scope: Per-Category

- **Decision**: Each formula modifier tag specifies a **formula category scope** — which categories of formulas it applies to (attack, damage, defense, healing, etc.). A modifier can target multiple categories.
- **Rationale**: Per-category scoping prevents a single modifier from universally replacing an attribute across all formulas. "Intelligent Strikes" should make Intellect replace Accuracy in *attack* formulas, not in defense or healing formulas too. This is important for balance — universal attribute replacement would be too powerful.
- **Implications**: Formula modifier data model gains a `scope` field (list of formula categories). Content authors must specify which categories each modifier affects. The formula evaluation engine checks scope before applying modifiers.

### Fallen Resolution: Initiative Order

- **Decision**: Characters enter Fallen in **reverse Initiative order** (lowest Initiative first). `OnFallen` Triggers fire sequentially in this order.
- **Rationale**: Initiative order provides a deterministic, meaningful ordering for Fallen resolution. Lowest-Initiative characters fall first, which means higher-Initiative characters' Triggers fire last — potentially benefiting from the state changes caused by earlier Triggers. Sequential processing (not simultaneous) ensures clear cause-and-effect chains.
- **Implications**: High-Initiative characters who fall on the same tick get their Triggers processed after lower-Initiative characters. This creates a subtle advantage for Speed investment. OnFallen Trigger chains are deterministic and ordered.

### Revive Works on Any Ally

- **Decision**: The Revive effect component works on any ally target. On a non-Fallen target, Revive does nothing — but other effect components in the Action still resolve normally. This enables hybrid heal+revive Actions.
- **Rationale**: Allowing Revive on any target (with no-op on living allies) enables versatile hybrid Actions without requiring separate "heal" and "revive" versions. A single "Healing Light" Action with Heal + Revive components can serve both purposes. The `[Fallen]` target tag still exists for dedicated Revive-only Actions that should only be used on Fallen allies.
- **Implications**: AI must evaluate hybrid heal+revive Actions for both living and Fallen ally targets. Content authors can create versatile support Actions that combine healing and revival. The `[Fallen]` tag remains for Actions that should exclusively target Fallen allies.

### Numeric System: Ceiling Integer Rounding

- **Decision**: All combat values are integers. All fractional results use ceiling rounding (round up). Minimum 1 damage per component on successful hits. Minimum 1 stack on all status applications.
- **Rationale**: Ceiling rounding favors attackers — even tiny damage deals at least 1. Consistent with minimum 1 stack philosophy. Integer-only values simplify implementation and player reasoning.
- **Implications**: All formula evaluations produce integers via ceiling. Damage, healing, Initiative gain, status decay, Soak reduction — everything is integers.

### Block Chance in Combat Pipeline

- **Decision**: Block Chance from equipped shield items (Buckler, Kite Shield, Tower Shield) is integrated into the per-component damage resolution pipeline between damage modifiers and Soak reduction (step 5 of 9). Block applies to all damage types, splits proportionally across multi-component damage.
- **Rationale**: Block as a pre-Soak flat reduction makes it especially effective against weaker hits (removing a larger percentage of damage before Soak's diminishing returns apply). Proportional split prevents gaming by component ordering. All-type blocking is thematic — the shield physically intercepts the attack.
- **Implications**: Two separate "shield" mechanics in the system: Block Chance (from equipment shield items, flat reduction) and Shield HP (from Perks/consumables, absorption pool). Equipment spec defines Block Chance/Value per shield type; combat spec defines pipeline position.

### Off-Hand Bonus Attack as Sequential Second Pass

- **Decision**: When dual-wielding or using weapon+focus, the main Action resolves through the full pipeline, then the off-hand fires a generic Attack as a completely separate pipeline pass (own hit roll, own crit check, own Block, own Soak). Each implement's proc Triggers fire only on its own hits.
- **Rationale**: Two separate Actions is clean and consistent — no special cases in the pipeline. Per-implement Trigger firing creates interesting build choices. DW penalties (-15%/-35%) ensure the bonus attack doesn't make dual-wielding dominant.
- **Implications**: Every dual-wield turn produces two separate damage events in the combat log. AI must evaluate off-hand bonus value. Shields are more effective against dual-wielders (two Block checks per turn).

### Friendly Fire: Full Normal Pipeline

- **Decision**: When Actions with `[All]` targeting tags hit allies (friendly fire), the full combat pipeline applies — Defense roll, Block, Soak, Resistance, Shields. The "allies skip resistance" rule applies ONLY to beneficial effects (heals, buffs, positive statuses), not to friendly fire damage.
- **Rationale**: Without this rule, a character's own AoE would deal MORE damage to allies than to enemies (unresisted damage). The full pipeline prevents this design bug while keeping beneficial effects reliable.
- **Implications**: Effect source tracking needs to distinguish beneficial vs harmful effects, not just ally vs enemy. AI `aoe_net_value` Scorer accounts for friendly fire pipeline correctly.

### Additive Status Stacking with Formula-Based Decay

- **Decision**: New status applications add stacks to existing stacks (additive). No separate duration tracking — stacks ARE the duration. Each status type defines a per-tick decay formula (percentage, flat, or attribute-derived). Character status tracking is minimal: `{status_type, current_stacks}`.
- **Rationale**: Eliminates the complexity of duration timers. Percentage decay creates natural exponential falloff. Attribute-derived decay (e.g., Stun → Willpower) ties defensive stats to status recovery. Ceiling rounding ensures statuses always decay (last stack always clears).
- **Implications**: Status type definitions gain a `decay_formula` field using the standard formula representation. Content defines per-status decay behavior. Multiple sources of the same status stack naturally.

### Penetration Diminishing Returns

- **Decision**: Penetration uses the same harmonic diminishing returns formula as Soak: `Effective_Pen = Pen × K_pen / (Pen + K_pen)`. The effective value subtracts from Soak. Same formula shape everywhere in the system.
- **Rationale**: Flat Penetration without DR could completely zero out Soak at high values. Harmonic DR creates system consistency — one formula shape for Soak, Resistance, and Penetration.
- **Implications**: K_pen is a new tuning constant. Penetration affixes need rebalancing for DR. First 50 points of Penetration are much more impactful than going from 150 to 200.

### Defend Boosts All Defenses, No Tick Limit

- **Decision**: Defend boosts both Physical and Magic Defense, plus all 3 Soak families (Physical, Elemental, Magical). No maximum tick duration — persists until the character's next turn. Implemented as the "Defending" status (1 stack, special decay: clears on turn start).
- **Rationale**: A general defensive brace should protect against all attack types. The stun+Defend interaction (indefinite defense while stunned) is accepted as thematic — a braced character who gets stunned is harder to kill while helpless. Content balances stun duration.
- **Implications**: Defend is a status effect using the standard stack-based system. No special mechanic needed.

### Trigger Chaining with Recursion Guard

- **Decision**: Triggers can chain — a Trigger's effect can fire other Triggers, which can chain further. Each individual Trigger instance fires at most once per originating event (recursion guard).
- **Rationale**: Unlimited chaining enables complex "combo chain" Perks (OnHit → apply Burning → OnStatusApplied → bonus damage). The recursion guard prevents infinite loops without limiting creative content design.
- **Implications**: Implementation maintains a "fired set" per originating event. Content authors can create chain-enabling Perks. Performance: chain depth is bounded by the total number of unique Triggers on the affected characters.

### Summon Starting Initiative: Zero

- **Decision**: Summoned Ephemeral Combatants start with Initiative = 0 and accumulate from scratch.
- **Rationale**: Summoning is a long-term investment, not instant payoff. High-Speed summons (wolves, elementals) reach their first turn faster than slow summons (golems). Simple rule consistent with default starting Initiative.
- **Implications**: Summoner build evaluation must account for summon ramp-up time. AI factors Initiative ramp into summon decision timing.

### No True Status Immunity

- **Decision**: There are no immunity Perks. Minimum 1 stack always applies with no exceptions. Defense against powerful statuses comes from high Resistance (reduces to 1 stack) + fast decay (attribute-derived formula clears quickly). Functionally zero impact from 1 stack that decays in the same tick.
- **Rationale**: Removing the immunity concept simplifies the system. The combination of minimum-1-stack + fast-decay provides the same gameplay outcome (negligible status impact) without a separate binary immunity mechanic. No special cases in status application.
- **Implications**: Removes mentions of "Stun immunity Perks" from the spec. High-Willpower characters are effectively stun-proof through fast decay, not through a separate immunity gate.

### Hit Bonus: Post-Modifier Flat Addition

- **Decision**: The hit margin (roll − Defense) is added to damage AFTER crit multiplier and damage modifiers but BEFORE defensive reductions (Block, Soak, Shield). Hit bonus is NOT amplified by crit or buffs — it's raw accuracy reward. Defenses still reduce it.
- **Rationale**: Attack Value intentionally double-dips (hit chance AND damage scaling), creating high per-hit variance. This is the intended combat feel — a great attack roll is both accurate and powerful. Applying hit bonus post-crit prevents crit from amplifying accuracy reward.
- **Implications**: High Attack investment is very powerful. The 9-step per-component damage pipeline explicitly orders hit bonus at step 4.

### AoE: One Roll, Per-Target Defense

- **Decision**: For AoE Actions, the attacker rolls once (1 to Attack Value). Each target subtracts their own Defense from that single roll independently. A high roll hits everyone; a low roll may miss high-Defense targets but hit low-Defense ones. Hit bonus varies per target.
- **Rationale**: "One swing" is thematic and faster to resolve. Per-target Defense comparison means the same attack can hit some targets and miss others based on individual defenses — creates interesting AoE outcomes.
- **Implications**: One roll per AoE Action, not per target. Each target gets an independent hit/miss result and hit bonus. Multihit (from implement tags) follows the same rule.

### Might Mitigates Armor Speed Penalties

- **Decision**: Might reduces Medium/Heavy armor Speed penalties using the harmonic diminishing returns formula: `effective_penalty = base_penalty × (1 - Might/(Might+K_armor))`. K_armor is a tuning constant.
- **Rationale**: Gives Might a unique combat niche beyond weapon damage — it's the "I can wear heavy armor effectively" stat. Creates natural Might + Heavy Armor synergy. Thematic: strong fighters bear heavy gear; mages are pushed toward Light armor.
- **Implications**: Might description in characters spec needs updating. Equipment spec's armor Speed penalty section gains a cross-reference. K_armor is a new tuning constant. A high-Might warrior in full Heavy armor (base −30 Speed) might reduce the penalty to about −12.

### Equipment Combat Mechanics Cross-Reference

- **Decision**: Combat spec gains an "Equipment Combat Mechanics" cross-reference section listing all equipment-originated combat effects (DW penalties, armor Speed penalties, implement resolution, Versatile mode, Multihit, equipment combat lock, Reach/Ranged, Block, implement tags) with links to equipment spec for details.
- **Rationale**: Implementation teams can find all combat-relevant mechanics from one document without duplicating content. Equipment spec remains the authoritative source for equipment details; combat spec provides navigability.
- **Implications**: Combat spec is the "starting point" for combat implementation; equipment spec provides equipment-specific detail.

### Status Decay: Ceiling Rounding Shortens Durations (#121)

- **Decision**: Ceiling rounding applies to decay removal, making percentage-decay statuses shorter-lived than they appear. Burning 25% decay: `ceil(10×0.25)=3` removed → 10→7→4→2→1→0 (5 ticks, not 8).
- **Rationale**: Consistent with the global ceiling rounding rule. Ceiling on decay removal favors status recovery — statuses clear faster than naive calculation suggests. This is intentional: it balances the minimum-1-stack rule on application.
- **Implications**: Content authors must use the ceiling-corrected decay sequence when designing status durations. A "25% decay" status clears in roughly `ceil(log(stacks) / log(1/0.75))` ticks.

### Default Action Speeds: Non-Offensive Actions Are Faster (#122)

- **Decision**: Default Action Speeds: Attack = 0 (−100 Initiative), Move = +25 (−75), Defend = +25 (−75), Search = +50 (−50). Non-offensive actions are faster than Attack.
- **Rationale**: Non-offensive default Actions should be competitive with attacking. Search is deliberately cheapest — map-wide detection at low Initiative cost makes it a viable tactical option. Move and Defend as "quick" actions reward tactical decisions over pure aggression.
- **Implications**: AI scoring must account for variable Initiative costs when comparing default Actions. A character who Searches recovers Initiative 50% faster than one who Attacks.

### Hit Bonus: Primary Component Only (#123)

- **Decision**: The hit bonus (roll − Defense) is added as flat addition to the **primary damage type component only**. Secondary damage components in multi-component Actions do NOT receive the hit bonus.
- **Rationale**: Prevents multi-component Actions from double-dipping on Attack investment. A "Flame Blade" with Slashing (primary) + Fire gets the hit bonus only on the Slashing component. This keeps multi-component Actions balanced — their advantage is hitting multiple Soak families, not amplifying hit bonus across all components.
- **Implications**: The per-component damage pipeline step 4 must identify which component matches the Action's primary damage type. Content balance should consider that secondary components receive no hit bonus.

### Off-Hand Fires on Enemy-Targeted Actions Only (#124)

- **Decision**: Off-hand bonus attack fires when the main Action targets enemies (any enemy-targeting component — Damage, debuff). Does NOT fire for heals, self-buffs, Move, Defend, Search. A debuff-only Action targeting enemies DOES trigger off-hand.
- **Rationale**: Thematically, the off-hand swing is part of an aggressive act. Healing an ally while also swinging your off-hand weapon is not a coherent combat action. Debuffs targeting enemies ARE aggressive (poisoning someone while stabbing with the main hand).
- **Implications**: AI must distinguish offensive from non-offensive Actions for off-hand evaluation. Dual-wielding healers do NOT get bonus attacks when healing — only when using enemy-targeted Actions.

### Revive at Step 0 in Effect Resolution (#125)

- **Decision**: Revive components resolve at Step 0 (before all other effects) in the type-ordered effect resolution. Hybrid heal+revive Actions: Revive first (clean slate), then buffs (Step 1), then heals (Step 2).
- **Rationale**: Ensures hybrid Actions work naturally without special-casing. A "Healing Light" with Revive+Buff+Heal revives the character (clearing statuses), applies fresh buffs, then heals — the revived character gets full benefit of all components.
- **Implications**: Content authors can create reliable hybrid revive+buff Actions. The clean slate from Revive doesn't destroy the Action's own buff/heal components because they resolve after Revive.

### Deferred Kill Trigger Queue (#126)

- **Decision**: OnKill, OnFallen, and OnAllyFallen use a deferred trigger queue. Potential kills are recorded during Actions; confirmed at Step 4 (Fallen Resolution); triggers fire at Step 5 (Kill Triggers). Characters healed back above 0 HP before Step 4 do not generate kill triggers.
- **Rationale**: Immediate OnKill during Actions would fire for targets who might be healed back above 0 HP before Fallen Resolution — creating false kills. Deferral ensures kill triggers are accurate. Also creates a cleaner separation between Action-phase triggers (immediate) and kill-phase triggers (deferred).
- **Implications**: AI cannot rely on immediate OnKill feedback during the Turns Phase. Step 5 adds a new tick resolution phase. Recursion guard applies per-kill-event in Step 5.

### Remove OnOverkill — Merge into OnKill (#127)

- **Decision**: `OnOverkill` is removed from the trigger vocabulary. Overkill value data is available in the `OnKill` event payload. One fewer trigger type to maintain.
- **Rationale**: OnOverkill was a niche trigger that could be fully served by OnKill with overkill data in its payload. Content that wants overkill-dependent behavior can check the payload value. Reduces vocabulary size without losing functionality.
- **Implications**: All references to OnOverkill replaced with OnKill. Content using overkill-dependent triggers reads from the OnKill event payload.

### New Trigger Events: Revival, Defense, Healing, Dodge, Summon (#128)

- **Decision**: Seven new trigger events added: `OnRevive` (revival), `OnBlock` and `OnShieldBreak` (defense), `OnHeal` and `OnHealedBy` (healing), `OnDodge` (attack/defense), `OnSummon` (summon). One removed: `OnOverkill`.
- **Rationale**: Fills gaps in the trigger vocabulary that would block Perk content design. OnBlock enables shield-focused builds. OnHeal/OnHealedBy creates symmetric healing triggers (matching OnDamageDealt/OnDamageTaken). OnDodge is the defender's counterpart to OnMiss. OnRevive enables post-revival Perks. OnSummon enables summoner synergies.
- **Implications**: Traits-and-perks has 7 new trigger events available for Perk content design. AI considerations may need updating for new trigger evaluations. Total trigger vocabulary: 30+ events.

### Detection: Per-Turn Only, No Persistent State (#129)

- **Decision**: Detection is per-turn only — there is no persistent "this team has detected character X" state between turns. Before each character's turn, passive detection re-rolls. Active Search also has per-turn persistence only. Teammates must make their own detection checks.
- **Rationale**: Per-turn detection prevents a single high-Awareness character from permanently revealing all stealth enemies for the whole team. Each character must earn their own detection. This makes stealth more robust and Awareness investment more individually meaningful.
- **Implications**: No detection state tracking needed between turns (simpler implementation). AI stealth evaluation must account for per-character detection probability, not team-level detection. Search's value is per-character, not team-wide.

### DoTs Bypass Block (#130)

- **Decision**: DoTs bypass both Soak AND Block. DoT tick damage occurs in the Effects Phase, outside the per-component damage pipeline where Block operates. DoT damage path: DoT tick → Shield absorption → Health.
- **Rationale**: Block is a per-Action mechanic — the shield physically interposes against an incoming attack. DoTs are ongoing status effects, not discrete attacks. Block conceptually doesn't apply. This also simplifies the Effects Phase — DoT damage just goes straight to Shields/Health.
- **Implications**: DoT-focused builds bypass two layers of defense (Soak and Block) but are still gated by Resistance at application and Shields during ticks. Makes Resistance and Shields the primary DoT defenses.

### Summon Deaths Fire Full Triggers (#131)

- **Decision**: When summons are removed (killed or summoner enters Fallen), they fire full trigger events — OnFallen on the summon, OnAllyFallen on teammates. A summoner with 5 summons falling generates 5× OnAllyFallen.
- **Rationale**: Summons are combatants that participate fully in the trigger system. Excluding them from death triggers would create special-case code and prevent interesting summoner Perks (e.g., "Necro Empowerment: OnAllyFallen: gain stacks of Empowered"). The cascade potential is a feature, not a bug — it rewards summoner investment.
- **Implications**: Content design should account for summon death cascades. A 5-summon necromancer falling is a major trigger event (6 total Fallen events). Balance should consider this when designing OnAllyFallen Perks.

### Defend + Stun: Accepted Emergent Strategy (#132)

- **Decision**: The Defend+Stun interaction (indefinite defense via self-stun) is an accepted emergent strategy. A team deliberately stunning their own Defended tank creates a meaningful trade-off: costs the stunner's Action, deals friendly fire damage from the stun-applying ability, and the stunned tank can't act.
- **Rationale**: This interaction emerges naturally from the rules (Defending clears on turn start; stunned characters never get turns). Rather than patch it with a special case, it's accepted as an interesting tactical option with real costs. Content provides counters (anti-stun Perks, dispel mechanics, Defend-removing abilities).
- **Implications**: No rule changes needed. Content design should ensure adequate anti-stun and dispel options exist as counters. AI should be capable of evaluating self-stun strategies (likely very niche).

### Crowd Appeal: Deferred to Phase 4 (#133)

- **Decision**: Crowd Appeal (derived stat from characters spec) has no mechanical combat effect in MVP. It exists as a derived stat for future use in the Morale system (Phase 4).
- **Rationale**: Crowd Appeal feeds into the Morale system, which is entirely deferred. Including it as a stat now ensures the foundation exists, but there's no point in giving it combat mechanics before Morale is designed.
- **Implications**: Crowd Appeal can be calculated and displayed but has no combat impact. Phase 4 Morale design will define its mechanical role.

### Revival Clean Slate: True Clean Slate (#134)

- **Decision**: Revival clean slate clears ALL statuses — both beneficial and harmful. No "buff vs debuff" classification. This is a complete reset.
- **Rationale**: Binary buff/debuff classification would add complexity (what about neutral statuses? Mixed statuses?). "Clear everything" is the simplest, most predictable rule. Hybrid revive+buff Actions work naturally via type-ordered resolution: Revive (Step 0, clears all) → Status (Step 1, applies fresh buffs).
- **Implications**: Pre-fall buffs are lost on revival. Content cannot rely on buffs persisting through fall+revival cycles. Revive+buff Actions are the intended way to buff revived characters.

### Kill Triggers in Step 5 of Tick Resolution (#135)

- **Decision**: Add Step 5 "Kill Triggers" to tick resolution order, after Fallen Resolution (Step 4). Tick now has 5 steps: (1) Effects Phase, (2) Initiative Phase, (3) Turns Phase, (4) Fallen Resolution, (5) Kill Triggers.
- **Rationale**: Kill triggers need their own resolution step because they are deferred (not fired during Actions). Placing them after Fallen Resolution ensures only confirmed kills generate triggers. This is the natural home for the deferred kill trigger queue.
- **Implications**: Tick resolution gains a fifth step. Data model needs a deferred kill trigger queue that accumulates during the tick and drains in Step 5.

### Effects Phase: Internal Sub-Ordering (#136)

- **Decision**: The Effects Phase resolves in fixed internal order: (1) Decay — all stacks on all statuses decay, (2) Effects — all per-stack effects apply, (3) Shield Expiry — expired shields removed last. Per-character ordering: highest stack count first (tiebreaker: alphabetical). All characters process simultaneously.
- **Rationale**: Decay-before-effects means statuses apply at their post-decay stack count (not pre-decay). Shield expiry last means shields protect against DoTs on their final tick. Simultaneous cross-character processing prevents ordering dependencies.
- **Implications**: A 3-stack DoT that decays by 1 this tick deals damage at 2 stacks, not 3. Shields set to expire this tick still absorb DoT damage. Implementation must process all characters' decay before any character's effects.

### Step 5 Kill Triggers: AoE Batch Sharing (#137)

- **Decision**: AoE kills from the same Action share a batch in Step 5. Each kill within the batch is a separate originating event for the recursion guard.
- **Rationale**: Consistent with the batch-then-triggers model for AoE resolution. Multiple kills from one AoE should fire their kill triggers together (as a batch), matching how AoE Action triggers work. Separate originating events per kill ensures the recursion guard tracks each kill chain independently.
- **Implications**: A "Blood Frenzy" Perk (OnKill: +damage stack) would gain one stack per confirmed kill in the batch. An AoE that kills 3 enemies gives 3 OnKill events, each as a separate originating event.

### OnRevive: Immediate During Step 0 (#138)

- **Decision**: `OnRevive` fires during Step 0 of effect resolution as an immediate trigger on the revived character. Not deferred like kill triggers.
- **Rationale**: OnRevive needs to fire immediately so that Revive-triggered effects (e.g., "gain Shield on revive") can apply before other Action components resolve. Deferring it would break the type-ordered resolution model.
- **Implications**: Content can create "Phoenix" style Perks: OnRevive → gain Shield, gain buff, etc. These effects apply in the Revive step before other Action components.

### OnBlock and OnShieldBreak: Pipeline Triggers (#139)

- **Decision**: `OnBlock` fires at pipeline step 5 (Block check) and `OnShieldBreak` fires at pipeline step 7 (Shield absorption). Both join the Action's trigger collection for batch resolution.
- **Rationale**: These are combat pipeline events that naturally fire during their respective pipeline steps. Joining the Action's trigger collection is consistent with how other pipeline triggers (OnHit, OnCrit) work.
- **Implications**: Shield-focused builds can trigger effects on blocks ("Riposte: OnBlock → counter-attack") and shield breaks ("Desperate Shield: OnShieldBreak → gain Defending status").

### OnHeal/OnHealedBy: Symmetric Healing Triggers (#140)

- **Decision**: `OnHeal` (healer's perspective) and `OnHealedBy` (recipient's perspective) added as symmetric counterparts to OnDamageDealt/OnDamageTaken. Fire immediately during the Action.
- **Rationale**: The trigger vocabulary had damage triggers from both perspectives but no healing equivalents. Healing-focused builds need trigger events. OnHeal enables "Holy Fervor" (gain stacks on healing), OnHealedBy enables "Grateful Blessing" (buff when healed).
- **Implications**: Content design gains healing-triggered Perk options. AI Considerations should evaluate healing triggers when assessing heal Action value.

### OnDodge: Defender's Perspective on Miss (#141)

- **Decision**: `OnDodge` fires when this character successfully dodges an enemy attack (defender's perspective, attack roll < 0). Symmetric with `OnMiss` (attacker's perspective).
- **Rationale**: The vocabulary had OnMiss (attacker side) but no defender equivalent. High-evasion builds need a trigger for "dodge-and-counter" Perks (e.g., "Riposte: OnDodge → bonus attack").
- **Implications**: Content gains dodge-reactive Perks. Speed/Accuracy investment gains trigger synergy value beyond just hit/miss probability.

### OnSummon: Summoner Trigger (#142)

- **Decision**: `OnSummon` fires when this character creates a summon via the Summon effect component. Fires immediately during the Action.
- **Rationale**: Summoner builds need a trigger for summon-related synergies (e.g., "Pack Tactics: OnSummon → buff all existing summons", "Overcharge: OnSummon → summon gains bonus stats").
- **Implications**: Content gains summoner-synergy Perks. AI should factor OnSummon triggers into summon Action evaluation.

### Trigger Timing Classification (#143)

- **Decision**: Triggers are classified into three timing categories: immediate (fire during Action resolution), revive (fire during Step 0), and deferred (fire in Step 5). This classification determines when effects from triggers can interact with other combat events.
- **Rationale**: Different trigger types need different timing to produce correct behavior. Kill triggers must be deferred (to avoid false kills from healed targets). Revive triggers must be immediate (to work within effect resolution). All others are immediate (standard batch-then-triggers model).
- **Implications**: Implementation needs three trigger dispatch paths. Content authors should understand timing when designing trigger-dependent Perks.
