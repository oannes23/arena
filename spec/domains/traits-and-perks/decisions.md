# Decisions

### Three-Category Trait System

- **Decision**: Traits are divided into Core (identity), Role (capability), and Bond (affiliation) categories with separate slot pools.
- **Rationale**: Creates character identity along three axes. Prevents "all combat" builds by requiring investment across identity, capability, and affiliation.
- **Implications**: Character generation must produce Traits from all three pools. Each category needs enough content for meaningful choice at every star level.

### Perk as Multi-Component Package

- **Decision**: A single Perk can contain any mix of Actions, Stat Adjustments, and Triggers rather than being one component type. No hard system limit on component count — content guidelines suggest 1–3 total components per Perk.
- **Rationale**: Enables rich, thematic abilities (Flame Aura has an active component, a passive bonus, AND a reactive trigger). Reduces Perk count while increasing Perk depth. Soft guidelines keep content manageable without constraining design space.
- **Implications**: Perk balance must consider the combined value of all components. UI must display multi-component Perks clearly.

### Trait Level Amplifies All Child Perks

- **Decision**: Leveling a Trait (1–5★) amplifies ALL Perks within that Trait via a per-level multiplier (slightly accelerating: ×1.0 / ×1.2 / ×1.4 / ×1.7 / ×2.0). Leveling a specific Perk (1–5★) amplifies only that Perk using the same curve. Both stack multiplicatively — max total at 5★/5★ = 4.0× base values.
- **Rationale**: Creates a "wide vs. deep" investment choice — level the Trait for broad improvement, or level specific Perks for focused power. Accelerating curve rewards full commitment. Shared curve keeps the system simple.
- **Implications**: Balance must account for double-stacking (high Trait level × high Perk level). Exact multiplier values are tuning — deferred to combat spec.

### Fixed Escalating XP Schedule

- **Decision**: Both Traits and Perks use the same XP cost schedule: 0→1★ = 100, 1→2★ = 300, 2→3★ = 600, 3→4★ = 1000, 4→5★ = 1500 XP. Total to max (0→5★) = 3500 XP.
- **Rationale**: Escalating costs create natural diminishing returns at higher levels. Shared schedule between Traits and Perks keeps the system simple. Combined with trainer service fees (gold), creates dual-currency pressure.
- **Implications**: Economy spec must define trainer service fee scaling. A character needs 3500 XP to max one Trait + 3500 per Perk — full investment in one 5-Perk Trait costs ~21,000 XP.

### Cross-Category Perk Sharing — No Stack

- **Decision**: Identical Perks can appear in different Traits across categories. When a character has the same Perk from two sources, it does not stack — the second source provides redundant access only. The higher parent Trait's amplification multiplier applies.
- **Rationale**: Redundant access is insurance: if you respec away one Trait, you keep the Perk from the other. No stacking prevents degenerate double-dipping builds. Using the higher amplification is simple and always benefits the player.
- **Implications**: Players may intentionally overlap Perks across Traits for safety, but there is no power incentive to do so.

### Starter Perk Auto-Grant

- **Decision**: Every Trait has exactly one designated Starter Perk. This Perk is auto-granted (free, no XP cost) when the Trait is acquired.
- **Rationale**: Ensures every Trait is immediately useful upon purchase — no Trait is a blank slate requiring additional XP investment before it does anything. The Starter Perk is the Trait's "signature move."
- **Implications**: Trait design must always include a compelling Starter Perk. The Starter Perk defines the Trait's first impression and basic capability.

### Expensive Respec

- **Decision**: Removing Traits is an expensive vendor service from a Group. All XP invested in the removed Trait and its Perks is lost. Gold cost to the Group — formula deferred to [economy](../economy.md) spec.
- **Rationale**: Encourages roster turnover — it's often better to hire a new character than endlessly respec an existing one. Creates attachment to characters and investment in hiring decisions.
- **Implications**: Players need enough income to hire replacements. The meta shifts toward roster breadth over single-character perfection.

### Open Extensible Resource Pool System

- **Decision**: Resource pools are emergent from Perk content, not a Trait-level property. Five default resource types ship (Mana, Faith, Spirit, Focus, Stamina) with attribute-derived pool formulas. Content authors can define additional resource types with custom formulas and auto-tags. A resource pool activates when a character first owns a Perk that references it; the attribute formula provides the base, Perk Stat Adjustments add bonus capacity.
- **Rationale**: Emergent ownership is simpler than explicit Trait-to-family assignments and naturally handles multi-family Traits. Open extensibility means new resource types don't require system changes. Attribute-derived base provides meaningful variation between characters; Perk bonuses reward investment.
- **Implications**: Combat spec must implement resource pool formulas, activation, and scaling. Content authoring tool must support defining new resource types. No hard cap on how many resource pools a character can have — bounded by Perk content. AI must handle multi-resource Action costs.
- **Alternatives considered**: Fixed 5 families with Trait-level assignment (rejected — too rigid, prevented multi-family Traits), per-Perk-only pools with no attribute base (rejected — removed character stat relevance from pools).

### Species as Core Traits

- **Decision**: Species/ancestry are defined entirely via Core Traits with anatomy modification support. Multiple ancestry Traits are stackable. Conflict resolution uses per-slot maximum. Default humanoid anatomy for characters with no ancestry Traits.
- **Rationale**: Maximally extensible — new species are just new Core Traits. Supports hybrid ancestries naturally. No hardcoded species list means content can grow without schema changes.
- **Implications**: Core Trait definitions must specify any anatomical slot modifications. Equipment system must handle variable anatomy.

### Tag-Driven Synergies

- **Decision**: All cross-Trait synergies are emergent via shared tags. No explicit set bonuses between specific Traits. Tags live on Perks/Actions; Traits derive tags from the union of their Perks. Resource-referencing Perks auto-gain the corresponding resource tag (e.g., Mana → Arcane).
- **Rationale**: Tag-driven synergies are open-ended and scale with content — new Traits automatically interact with existing ones through shared tags. Explicit set bonuses would require manual curation and create a combinatorial explosion as content grows. Auto-tagging from resources ensures consistent categorization without manual effort.
- **Implications**: Tag vocabulary must be well-curated to enable meaningful but not overpowered synergies. Equipment and consumable specs share the same tag vocabulary.

### Perk Level Cap by Trait Level

- **Decision**: A Perk's level cannot exceed its parent Trait's level. Exception: Perks acquired via Perk Discovery override this cap at acquisition, but further manual leveling remains capped.
- **Rationale**: Creates a natural investment order — Trait first (broad power), then Perks (specific power). Prevents characters from having max-level Perks in a low-level Trait. Discovery exception rewards the rarity of the event at the moment of discovery, while maintaining the normal progression path afterward.
- **Implications**: UI must communicate the cap clearly. Leveling a Trait unlocks higher Perk level ceilings for all its Perks, including previously-discovered Perks.

### Acquisition-Only Requirement Gates

- **Decision**: All requirements (Trait, Perk, prerequisite) are checked only at acquisition. Once owned, items are never deactivated by subsequent stat changes or prerequisite loss.
- **Rationale**: Prevents frustrating "stat juggling" where equipping/unequipping items could cascade-deactivate build components. Simple mental model: once you have it, you keep it.
- **Implications**: Players can safely respec a Trait without worrying about deactivating Perks in other Traits that had cross-Trait prerequisites from it.

### Cross-Trait Interaction — Tags Only

- **Decision**: No Perk can name or reference another specific Trait or Perk. All cross-Trait interaction is via shared tags. The system is fully decoupled.
- **Rationale**: Tag-only interaction is maximally extensible — new Traits automatically interact with existing ones through shared tags without requiring manual cross-references. Prevents combinatorial explosion of explicit Trait-to-Trait interactions.
- **Implications**: Content authors must design Perks using tag references, not Trait/Perk names. Tag vocabulary curation becomes a key design task.
- **Alternatives considered**: Explicit cross-Trait synergies (rejected — scales poorly), hybrid tag + named references (rejected — complexity for little gain).

### Trainer Availability by Group

- **Decision**: Training is Group-specific — each Group's trainers teach only Traits/Perks aligned with their theme. Generalist Groups (combat academies, gladiatorial trainers) teach diverse low-level basic Traits.
- **Rationale**: Makes Group membership and Bond Traits more meaningful — players need connections to the right Groups to access desired training. Generalist Groups ensure baseline accessibility.
- **Implications**: Groups spec must define trainer menus per Group. Economy spec must handle pricing differences between specialist and generalist trainers.

### Perk Discovery via Combat — Per-Trait End-of-Combat Roll

- **Decision**: Single end-of-combat discovery check per qualifying Trait (Traits that had at least one Action used or Trigger fire — "active Traits only"). One roll per Trait using: `(Base Rate + Luck Bonus + Perk Bonuses) × Trait Level Multiplier` (×1.0–×2.0). Acquired at minimum star level, free of XP cost. Discovery roll result is rarity-weighted (lower-minimum-star Perks more likely). Discovered Perks override the Perk level cap at acquisition only; further leveling remains capped. Full tree = silently skipped.
- **Rationale**: Per-Trait end-of-combat rolls are simpler, fairer (no bias toward high-Action-count builds), and create a cleaner handoff to the [post-combat](../post-combat.md) phase. The "active Traits only" rule replaces the old "passive Stat Adjustments don't trigger" rule with cleaner semantics. Rewards Trait investment and active combat participation.
- **Implications**: Base rate is higher than the old per-use 0.1% (tuning value) to produce similar per-fight discovery rates. Post-combat spec owns resolution mechanics (accept/reject). Combat Scoreboard determines which Traits were "active."
- **Alternatives considered**: Per-Action/Trigger-use during combat (original model — rejected for complexity and Action-count bias), per-character single roll (rejected — would not reward diverse Trait investment).

### No Star Gate for Trait Acquisition

- **Decision**: A character's Star Rating does not restrict which Traits they can acquire. A 1★ character can acquire a 5★-minimum Trait if they have an empty slot, meet the Trait's requirements, and can afford the XP. Star Rating determines slot count only.
- **Rationale**: The XP cost is already a natural gate — a min-5★ Trait costs 3500 XP cumulative, which is prohibitively expensive for a low-star character early on. Adding a star gate would be redundant and reduce build flexibility for players who want to invest heavily in a single Trait on a low-star character.
- **Implications**: Economy is the primary gate for high-star Traits on low-star characters. The cumulative XP cost (100/300/600/1000/1500) naturally prevents abuse.

### Simultaneous Trigger Resolution

- **Decision**: When multiple Triggers fire on the same combat event, all qualifying Triggers fire simultaneously. Effects are collected and applied together with no ordering dependency.
- **Rationale**: Simplest model. Eliminates order-dependent interactions, which would be confusing for players and difficult to balance. No incentive to acquire Perks in a specific order.
- **Implications**: Combat spec implements collected-and-applied-together resolution. No Trigger can depend on the result of another Trigger firing on the same event.

### Multi-Resource Action Costs

- **Decision**: A single Action can cost resources from multiple pools (e.g., 30 Mana + 20 Stamina for a hybrid "Spellblade Strike"). Any combination of resource types is allowed.
- **Rationale**: Enables hybrid-fantasy abilities and creates interesting resource management tension. Characters who invest in multiple resource families can access powerful cross-discipline Actions.
- **Implications**: Combat AI must evaluate multi-resource costs when choosing Actions. Content authors must balance multi-resource Actions carefully — they should be powerful enough to justify the broader resource investment.

### One Bond Trait per Group

- **Decision**: A character can hold only one Bond Trait per Group. The Bond Trait levels (1–5★) to represent deepening affiliation. No separate "Apprentice" vs. "Master" Bond Traits for the same Group.
- **Rationale**: Simplifies the Bond–Group relationship to a clean 1:1 mapping per character. Bond Trait leveling already provides progression depth through hybrid scaling (smooth benefits + discrete tier unlocks). Multiple Bond Traits per Group would dilute the leveling system.
- **Implications**: Groups spec: each Group defines exactly one Bond Trait. Bond Trait level is the single axis of Group relationship depth.

### Empty Category Respec

- **Decision**: Characters can respec down to zero Traits in any category. A character with no Role Traits, for example, is valid — they simply lack Role-granted capabilities.
- **Rationale**: Maximum flexibility. The guaranteed-first-roll rule at generation ensures characters start with at least one Trait per category, but post-generation respec should not be artificially constrained. An empty category is a meaningful (if unusual) player choice.
- **Implications**: UI should clearly communicate when a category is empty. Characters with empty Role slots may have limited combat utility but are not system-invalid.

### Tag-Based Action Targeting

- **Decision**: Actions use a tag-based targeting system. Each Action specifies target tags (e.g., `[Enemy, Single]`, `[Ally, AoE-Zone]`, `[Self]`). Combat spec defines how tags resolve to valid targets.
- **Rationale**: Extensible — new targeting modes are new tags, no schema changes. Tags are already the universal interaction vocabulary in the system. Consistent with the tag-driven design philosophy.
- **Implications**: Combat spec must define canonical target tag vocabulary and resolution rules. AI must parse target tags when evaluating valid Actions.

### Effect Component List Model

- **Decision**: Actions and Triggers express their outputs as an ordered list of generic effect components, each with a type tag and key-value parameters. Core component types (illustrative): Damage, Heal, ApplyStatus, RemoveStatus, Move, Summon, Shield. Full catalog owned by combat spec.
- **Rationale**: Extensible — new effect types don't require schema changes, just new type tags. Structured enough for validation and AI reasoning, flexible enough for content authors.
- **Implications**: Data model needs a component-list container for Actions and Triggers. Combat spec defines the canonical effect type catalog. Content authoring tools must validate component parameters.

### Type-Ordered Effect Resolution Within Actions

- **Decision**: Effect components within a single Action or Trigger resolve in type order: status/buff/debuff effects first, damage/healing effects second, movement effects last. Components within the same type category resolve simultaneously. This is defined authoritatively by the [combat](../combat/index.md) spec.
- **Rationale**: Enables powerful "setup-and-exploit" Actions (e.g., apply armor shred then deal damage against reduced Soak in the same Action). Creates richer design space for content authors while maintaining deterministic resolution.
- **Implications**: An Action CAN have an ApplyStatus that reduces Soak followed by a Damage component that benefits from the reduction. Content authors can design debuff+damage combo Actions. Multi-Trigger resolution remains simultaneous (no ordering between different Triggers firing on the same event).
- **Alternatives considered**: Fully simultaneous resolution (original model — rejected in favor of type-ordered for richer design space), fully sequential per-component ordering (rejected — too complex, ordering-dependent).

### Shared Effect Components Across Actions and Triggers

- **Decision**: Triggers use the exact same effect component list as Actions (Damage, Heal, ApplyStatus, Move, Summon, Shield, etc.). The only difference is activation: Actions are chosen by player/AI, Triggers fire automatically.
- **Rationale**: Unified effect model simplifies the system — one set of effect types, one scaling model, one resolution model. Content authors learn one system for both Actions and Triggers.
- **Implications**: Trigger balance must account for automatic activation — a Trigger that Summons on every hit would be more impactful than an Action that does the same. Content guidelines should address this.

### Flat + Tag-Scoped Stat Adjustments

- **Decision**: Stat Adjustments use two subtypes: flat bonuses (direct numeric bonus to any stat — Attributes, derived stats, resource pools, crit chance, etc.) and tag-scoped bonuses (conditional bonus when matching tag is present — any numeric stat, any tag scope).
- **Rationale**: Flat bonuses cover simple passive power. Tag-scoped bonuses create build depth by rewarding tag concentration without requiring explicit cross-Trait references. Covers both simple and conditional modifiers in a unified system.
- **Implications**: Data model needs two Stat Adjustment representations: `{stat_key, value}` for flat, `{tag, stat_key, value}` for tag-scoped. Tag-scoped bonuses interact with Tag System — same tags drive synergies and conditional modifiers.

### Bond Trait Hybrid Perk Pattern

- **Decision**: Bond Trait Perks grant both out-of-combat utility (discounts, service access, crafting bonuses) and combat capabilities (themed Actions, Stat Adjustments, Triggers).
- **Rationale**: Deep Group affiliation naturally grants combat knowledge and power alongside practical benefits. Keeps Bond investment meaningful for combat-focused players — Bonds aren't pure utility tax.
- **Implications**: Bond Perks create another source of combat power, giving Groups direct combat relevance beyond training access. Content balance must ensure Bond combat Perks complement (not replace) Role Trait combat power.

### Perk Discovery Modifiers: Additive Base × Trait Level

- **Decision**: Multiple sources modify the base ~0.1% Perk Discovery chance. Additive flat bonuses (Luck Attribute, specific Perks) combine into a base rate, which is then multiplied by the parent Trait's level using the standard amplification curve. Formula: `(Base Rate + Luck Bonus + Perk Bonuses) × Trait Level Multiplier`.
- **Rationale**: Multiple modifier sources reward diverse investment. Additive base model is transparent. Trait Level as a multiplier creates meaningful incentive to level Traits and uses existing mechanics. Perk-granted bonuses scale with level (consistent with all-numeric-outputs-scale rule), creating compounding optimization for discovery builds.
- **Implications**: Tuning must keep total discovery rate low enough that trainers remain primary acquisition. Luck gains another meaningful use. Trait level now has a secondary benefit beyond Perk amplification. At 5★ Trait (×2.0), effective discovery rate doubles. Exact bonus values are tuning — deferred to balance.

### Tag-Scoped Bonuses: Both Flat and Percentage, Additive Stacking

- **Decision**: Tag-scoped Stat Adjustment bonuses can be flat (+5 Soak vs [Poison]) OR percentage (+10% [Fire] damage). When multiple percentage bonuses apply to the same stat and tag, they stack additively. Application order: flat bonuses first, then combined percentage as a single multiplier. Level scaling multipliers apply before Stat Adjustments.
- **Rationale**: Allowing both flat and percentage tag-scoped bonuses maximizes design space — flat bonuses are better for small conditional buffs, percentages for scaling conditional multipliers. Additive percentage stacking is transparent and prevents exponential blowup from stacking many small percentage sources.
- **Implications**: Combat resolution must apply tag-scoped bonuses in the correct order (flat → percentage). Content authors should be aware that percentage bonuses become more powerful on high-base-damage Actions. Balance must monitor percentage accumulation across many Perks.

### Pre-Multiplier Bonus Pool Capacity

- **Decision**: Bonus pool capacity from Stat Adjustments (e.g., "+50 Mana pool") adds to the weighted attribute blend before the per-stat scaling multiplier is applied. The scaling multiplier amplifies both the attribute-derived base and the bonus capacity together.
- **Rationale**: Pre-multiplier placement makes bonus pool capacity a powerful and valuable investment — it scales with the same multiplier as the base pool. This rewards deep Perk investment in pool capacity and creates meaningful differentiation between characters who invest in pool bonuses and those who don't.
- **Implications**: Content authors must account for the scaling multiplier when designing bonus pool capacity values — a "+50" bonus becomes much larger after the multiplier (e.g., ×10 → +500 effective). Balance must ensure bonus capacity values are tuned for pre-multiplier placement. Combat spec's per-stat scaling multipliers directly affect the power of bonus pool capacity.

### Trait Level Multiplier on Perk Discovery

- **Decision**: Replace the "Bond Level Bonus" in the Perk Discovery formula with a universal Trait Level Multiplier. The parent Trait's level multiplies the discovery chance using the standard amplification curve (×1.0/×1.2/×1.4/×1.7/×2.0). Applies to all Trait categories (Core, Role, Bond). Perk-granted discovery bonuses are numeric outputs and thus also scale with level.
- **Rationale**: Simpler and more universal than a Bond-to-Trait association lookup. Eliminates the ambiguity of which Group is "thematically associated" with Core/Role Traits. Uses existing mechanics (the standard amplification curve) rather than introducing a new concept. Creates a natural incentive to level Traits beyond just Perk amplification.
- **Implications**: The Bond Level Bonus is removed from the formula — Bond Traits benefit from this equally via their own Trait level, not through a special association mechanism. Discovery chance at 5★ Trait is 2× the base rate, creating meaningfully higher discovery for deeply invested Traits. Perk-granted discovery bonuses compound with Trait level, creating interesting build optimization for discovery-focused characters.
- **Alternatives considered**: Bond Level Bonus with per-Trait Group association metadata (rejected — added data complexity, ambiguous for many Core/Role Traits), trainer-derived association (rejected — many-to-many relationship made it unclear which Bond level to use).

### Per-Combat Cooldown Reset

- **Decision**: All Action cooldowns reset fully between fights. Every combat encounter starts with a clean slate — no cooldown state persists across combats, even within multi-round tournament events.
- **Rationale**: Simplest model. Eliminates cross-combat state tracking. Ensures every fight is self-contained and fair — no player is disadvantaged by having used powerful abilities in a previous round. Consistent with resource pools starting at full capacity at combat start.
- **Implications**: Tournament balance relies on per-fight resource management, not cross-fight attrition. Powerful long-cooldown Actions are usable every fight. If cross-fight attrition is desired for tournaments, it should be expressed through other mechanics (injuries, Stamina, etc.) rather than cooldown carry-over.

### Automatic Redundant Access for Shared Perks

- **Decision**: When a character acquires a new Trait that contains a Perk they already own from another Trait, redundant access is granted automatically with no separate purchase. The Perk is the same entity regardless of which Trait tree it appears in.
- **Rationale**: A Perk is a single identity — "Perk X from Trait A" and "Perk X from Trait B" are the same Perk. Requiring separate purchase would create confusing UX (paying XP for something you already have) and undermine the "redundant access as insurance" design. Auto-access is the simplest model consistent with the no-stacking rule.
- **Implications**: When displaying Trait trees, Perks already owned from other sources should be visually marked as "already owned." Perk Discovery rolls in a new Trait should skip already-owned shared Perks (they're not "unowned"). Respeccing one source Trait leaves the Perk intact via the other source — no re-purchase needed.

### Acquisition-Only Equipment Requirements

- **Decision**: Equipment that required a specific Trait for equipping stays equipped when that Trait is respecced away. Equipment requirements are checked at equip time only, consistent with the acquisition-only gate philosophy.
- **Rationale**: Prevents frustrating cascade effects where respeccing a Trait causes gear to unequip, which could further reduce stats. Consistent with the universal "once you have it, you keep it" model.
- **Implications**: Equipment spec must implement equip-time-only requirement checks. Players can exploit this by equipping gear before respeccing — this is intended behavior (a form of planning reward).

### All Numeric Outputs Scale With Level

- **Decision**: Every numeric value in an effect component that represents an output (damage, healing, status stacks, shield amount, movement range) scales with the Perk/Trait level multiplier. Resource costs, cooldowns, and requirement thresholds stay flat at all levels. This is a universal rule with no per-component exceptions.
- **Rationale**: Simple universal rule that's easy to understand and balance. Scaling outputs + flat costs means higher levels are always worth it — more power for the same resource investment. No per-component opt-in keeps content authoring simple.
- **Implications**: Content authors design base values knowing they'll be multiplied up to 4.0× at max level (5★ Trait × 5★ Perk). Combat balance must account for this range.

### Discovery Rate: No Cap, Small Trees Sufficient

- **Decision**: No hard cap on effective Perk Discovery rate per Trait. At high investment (5★ Trait + Luck + discovery-boosting Perks), per-Trait rates can be significant. The natural limit is the small tree size (2-4 discoverable Perks per Trait). Discovery-optimized builds filling trees faster is the intended reward for that investment.
- **Rationale**: The tree size is the natural cap. Once all Perks are owned, rolls are skipped. A character investing heavily in discovery is choosing to use Perk slots and Trait levels for discovery bonuses rather than direct combat power — that's a legitimate build choice with meaningful opportunity cost.
- **Implications**: Trainer economy remains viable because discovery requires deep investment to reach high rates, and each Trait tree has very few undiscovered slots. Monitoring recommended: if discovery rates make trainers irrelevant, tuning the base rate is the lever.

### Post-Combat Discovery Resolution with Rejection

- **Decision**: Discovery rolls happen at end-of-combat (per qualifying Trait) and resolution occurs in the [post-combat](../post-combat.md) Perk Discovery phase (alongside injury checks, NPC recruitment, loot). The player can accept or reject each discovered Perk. Rejected Perks remain in the discovery pool for future rolls.
- **Rationale**: Post-combat resolution avoids interrupting combat flow. Accept/reject adds player agency, especially important now that Perks can have tradeoffs (negative components). Rejection keeping the Perk in the pool prevents players from "clearing out" undesirable Perks — they must buy them from trainers or accept the recurring discovery risk.
- **Implications**: Post-combat spec owns the discovery resolution UI and flow. Post-combat UI must present each discovered Perk with full details for informed accept/reject decisions.

### Tradeoff Perks via Existing Components

- **Decision**: Perks can include built-in tradeoffs using negative Stat Adjustment values and harmful Triggers. No new "Drawback" component type is needed. Content authors design tradeoff Perks using the existing Stat Adjustment (negative values) and Trigger (harmful effects) systems.
- **Rationale**: The component system already supports negative values — adding a separate Drawback type would be redundant complexity. Keeping tradeoffs within the existing framework means they benefit from all existing mechanics (level scaling, tag interaction, simultaneous resolution). Content guidelines can recommend tradeoff patterns without requiring system changes.
- **Implications**: Content authors can create Perks like "Berserker's Rage" (+30% melee damage, -20% defense) or Triggers like "Blood Frenzy" (gain damage on kill, lose Health each turn). Since all numeric outputs scale with level, both the positive and negative components of tradeoff Perks scale together — a 5★ tradeoff Perk has a bigger bonus AND a bigger penalty.

### Multi-Tag Additive Stacking

- **Decision**: When a multi-tagged Action (e.g., [Fire, Arcane]) is affected by tag-scoped bonuses from different tags, all percentage bonuses stack additively into a single total. `+10% [Fire] damage` + `+15% [Arcane] damage` = `+25% total damage` on that Action.
- **Rationale**: Additive cross-tag stacking is the simplest model and consistent with same-tag additive stacking. It rewards diverse tag investment without creating multiplicative explosions. Easy for players to reason about: add up all matching percentages.
- **Implications**: Multi-tag Actions benefit more from broad tag investment — a character with bonuses across multiple tags gets more value from Actions that match many tags. Content balance should be aware that Actions with many tags accumulate more bonuses. This is intentional — multi-tag Actions represent hybrid capabilities that reward broad builds.

### Equal Category Combat Power

- **Decision**: No Trait category "owns" combat power. Core, Role, and Bond Traits can all provide combat-relevant Actions, Stat Adjustments, and Triggers. A character built primarily around Core combat passives + Bond combat abilities (with no Role Traits) is a valid and potentially strong build.
- **Rationale**: Maximum build flexibility. Restricting combat power to Role Traits would make Core and Bond categories feel like taxes. Each category provides combat power through its own lens: Core = innate/biological advantages, Role = trained/specialized capabilities, Bond = group-taught thematic abilities.
- **Implications**: Build diversity comes from content variety and economic constraints, not from category restrictions. Combat-ai spec must handle characters with widely varying Action sources and compositions. Content design should ensure each category has a distinct flavor of combat contribution even though none is restricted.

### No Action Count Limit

- **Decision**: No mechanical limit on how many Actions a character can have available in combat. A fully built 5★ character with many Traits and Perks could have 20-40+ Actions. The combat AI evaluates all available Actions; situational filtering (targeting constraints, resource costs, cooldowns, tag-scoped bonuses) naturally narrows the effective choice set.
- **Rationale**: Limiting Actions would require an "equipped Actions" management layer that adds complexity without clear benefit. The AI auto-battles, so the player doesn't need to manually navigate large Action lists. Having many options is the reward for deep build investment.
- **Implications**: Combat-ai spec must handle large Action pools efficiently without performance issues or poor decision-making. The AI's ability to evaluate and prioritize among many Actions is a critical design challenge. UI for reviewing a character's Action list must handle large counts gracefully.

### Intentional Power Gap Across Star Ratings

- **Decision**: The power gap between low-star and high-star characters is large and intentional. A 5★ character has 5× the Trait slots of a 1★ character and potentially 4.0× multipliers on each. However, Trait/Perk star levels are NOT capped by character star level — a 1★ character can have 5★ Traits with 5★ Perks. The gap is breadth (slot count), not depth (individual Trait/Perk power).
- **Rationale**: Large power gaps create aspiration and reward long-term investment. Star-gated tournaments (1★-only, 3★-only, open) prevent unfair mismatches. A well-built 2★ beating a poorly-built 4★ is a design goal — build quality should matter. Roguelike turnover means reaching true max on a 5★ character is a rare achievement, not the norm.
- **Implications**: Tournament spec must support star-gated entry. Meta-balance spec (see below) provides automatic underdog bonuses for systematically disadvantaged builds. The combination of star gates + underdog bonuses + build quality creates a competitive landscape where investment matters but isn't insurmountable.

### Build Diversity Philosophy

- **Decision**: Build diversity is sustained by three complementary forces: economic scarcity (can't afford everything), content variety (design space too large to "solve"), and roster pressure (need different characters for different situations). Additionally, an automatic **underdog balancing** system tracks win/loss ratios per Trait and applies bonuses to underperforming Traits, preventing any single dominant strategy from persisting.
- **Rationale**: No single mechanism is sufficient for diversity — economic scarcity alone leads to "save up for the best" behavior, content variety alone can still have a solved meta, roster pressure alone doesn't address 1v1 balance. The combination, plus automatic meta-balance, creates a self-correcting ecosystem.
- **Implications**: **New spec needed**: a meta-balance spec to define the automatic underdog balancing system (win/loss tracking scope, bonus calculation, update frequency, visibility to players). Economy spec must create genuine scarcity. Content must be broad enough that no single optimal path exists. Roster-management spec must ensure multi-character investment is necessary.

### Prerequisite Graph: No Cycles Only

- **Decision**: The only hard constraint on Perk prerequisite graphs is no circular dependencies (A requires B, B requires A). No maximum depth or branching limit. Cross-Trait prerequisites remain allowed (rare). Content validation tools catch cycles at authoring time.
- **Rationale**: Minimal constraints maximize content design flexibility. Deep prerequisite chains create meaningful progression within a Trait. Cross-Trait prerequisites (rare) enable "prestige" Perks that require broad investment. Cycles are mathematically impossible to satisfy and must be prevented.
- **Implications**: Content authoring tools must include cycle detection for prerequisite graphs. No runtime enforcement needed — validation is purely at content creation time.
