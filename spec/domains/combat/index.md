# Combat — Domain Specification

**Status**: 🟢 Complete (updated 2026-02-22: rounds 26-31 — 23 additional decisions: implementation readiness pass — hit bonus primary-only, deferred OnKill queue, Step 5 kill triggers, Effects Phase ordering, default Action Speeds, Revive Step 0, detection per-turn, trigger vocabulary expansion)
**Last interrogated**: 2026-02-22
**Last verified**: —
**Depends on**: [characters](../characters.md), [traits-and-perks](../traits-and-perks/index.md)
**Depended on by**: [combat-ai](../combat-ai.md), [post-combat](../post-combat.md), [tournaments](../tournaments.md)

---

## Overview

Combat is the core gameplay loop — automated tactical fights between teams of characters on zone-based maps. Players configure builds and AI before combat; combat executes without player input. This domain covers the spatial system, initiative, action economy, combat resolution, damage types, status effects, stealth, targeting, resources, shields, attrition ramp, win conditions, and the combat scoreboard. Post-combat resolution (injuries, Perk discovery, recruitment, loot) is in [post-combat](../post-combat.md). Combat AI configuration is in [combat-ai](../combat-ai.md).

---

## Table of Contents

- [Foundation](foundation.md) — Core Concepts: Numeric System, Battle Format, Spatial System, Movement, Tick Resolution Order, Effects Phase Internal Ordering, Dying Blows, Initiative System, Action Economy, Action Speed
- [Resolution](resolution.md) — Combat Resolution Pipeline, Attack & Damage Formulas, Formula Representation, DoT Damage Rules, Passive Detection Timing, Summon Mechanics, Stun Mechanics, Friendly Fire Rules, Block Mechanics, Shield Mechanics, Effect Resolution Order, Critical Hit System, Damage Types, Status Effect System
- [Systems](systems.md) — Stealth & Detection, Targeting, AoE Resolution, Per-Damage-Type Soak, Dual Defense, Combat Event Stream, Resources, Judgment, Combat Context Flags, Resource Pool Scaling, Default Actions, Fallen State and Revival, Attrition Ramp, Fight Phase Signal, Post-Combat Handoff, Combat Scoreboard, Zone Effects, Morale
- [Vocabularies](vocabularies.md) — Canonical Vocabularies: Target Tags, Trigger Event Types, Trigger Timing Rules, Deferred Kill Trigger Model, Effect Component Types, Equipment Combat Mechanics Cross-Reference
- [Decisions](decisions.md) — All design decisions (143 across 31 rounds)
- [Open Questions](open-questions.md) — Remaining tuning values, deferred systems, and implications for other specs
