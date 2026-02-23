# Equipment — Domain Specification

**Status**: 🟢 Complete
**Last interrogated**: 2026-02-21
**Last verified**: —
**Depends on**: [characters](../characters.md), [traits-and-perks](../traits-and-perks/index.md), [combat](../combat/index.md)
**Depended on by**: [economy](../economy.md), [consumables](../consumables.md), [groups](../groups.md)

---

## Overview

Equipment is the gear system — weapons, focuses, armor, accessories, and shields that characters equip to enhance combat capabilities. Each character has a set of **Anatomical Slots** determined by ancestry (default humanoid: 11 slots). Hand slots hold two parallel implement categories: **Weapons** (physical combat implements) and **Focuses** (magical combat implements). Equipment provides implement-type-defined combat formulas, quality-scaled base stats, Diablo-style affixes with tiered pools, and optional granted Actions. Equipment degrades through usage-proportional wear, creating a permanent economic sink with tiered repair options.

---

## Table of Contents

- [Slots & Implements](slots-and-implements.md) — Anatomical slots, hand slots, wielding modes, off-hand bonus attack, implement resolution, default attack adaptation, unarmed/unfocused defaults
- [Types & Tags](types-and-tags.md) — Weapon types (12), focus types (7), structural tags, implement tags, armor tags
- [Armor](armor.md) — Per-slot armor, armor categories, multi-slot items, speed penalties, Soak families, mixing rules, shields, accessories
- [Quality & Affixes](quality-and-affixes.md) — Quality system, durability, degradation, repair, forging, enchanting, affix system, unique items
- [Inventory & Loot](inventory-and-loot.md) — Household stash, death handling, tournament lock, disposal, loot tables, enemy drops
- [Decisions](decisions.md) — Complete decisions summary table (103 decisions across 31 rounds)
- [Open Questions](open-questions.md) — Remaining tuning values, deferred systems, implications for other specs

---

_Last updated: 2026-02-21 — 31 interrogation rounds, 103 decisions. Rounds 1-15: core equipment system. Rounds 16-23: MAJOR RESTRUCTURE — parallel weapon/focus categories. Round 24: FOCUS TYPE REDESIGN — 7 types. Round 25: IMPLEMENT TAGS — Defend, Deflect, Deadly, Multihit. Rounds 26-29: Finesse/Deadly split, Sunder tag, Greataxe/Crossbow/Hand Crossbow/Sling, implement tags finalized, Channeling removed. Round 30: Multihit mechanics (1–5 triangular), focus power hierarchy, ranged formulas locked, Equipment Slot limit removed, full spec restructure. Round 31: CLEANUP — Deadly/Sunder per-implement scope, tag-derived Action Speed, enchanting source-themed, no Versatile focuses, accessory degradation tracks ticks, DW penalties (−15%/−35%), armor Speed penalties (Med −2/Hvy −5), degradation ~2/fight, percentage-based range/Multihit penalties (~20%), armor attribute scaling locked (3 attributes each). Status → 🟢._
