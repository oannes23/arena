# {{DOMAIN_NAME}} — Domain Specification

**Status**: 🔴 Not started
**Last interrogated**: —
**Last verified**: —
**Depends on**: {{DEPENDENCIES or "None (primitive)"}}
**Depended on by**: {{DEPENDENTS or "TBD"}}

---

## Overview

{{Brief description of what this domain covers and why it matters.}}

---

## Core Concepts

### {{Concept 1}}

{{Explain the concept.}}

### {{Concept 2}}

{{Explain the concept.}}

---

## Decisions

<!-- Decisions are added during interrogation. Format: -->

### {{Decision Area}}

- **Decision**: {{What was decided}}
- **Rationale**: {{Why this choice was made}}
- **Implications**: {{What this affects downstream}}
- **Alternatives considered**: {{Optional — what else was evaluated}}

---

## Open Questions

<!-- Questions that need resolution before this spec is complete -->

1. {{Question 1}}
2. {{Question 2}}

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [other-spec.md](other-spec.md) | {{How this spec affects it}} |

---

## Converting to a Directory

When a single-file spec grows beyond ~500 lines of content, convert it to a directory:

1. Create `spec/domains/<name>/` directory
2. Create `index.md` with metadata, overview, and Table of Contents
3. Split content into topical files (named by concept, not numbered)
4. Always extract `decisions.md` and `open-questions.md` as separate files
5. Update all cross-references in other specs (`<name>.md` → `<name>/index.md`)
6. For deep anchor links, route to the specific sub-file (`<name>/sub-file.md#anchor`)
7. Delete the original single file
8. Update MASTER.md domain table link

---

_Last updated: 2026-02-10_
