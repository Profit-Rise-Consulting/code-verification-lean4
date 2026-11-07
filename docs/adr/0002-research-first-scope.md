---
status: accepted
date: 2026-11-07
decision-makers: Profit Rise Consulting (owner), coding agent
decision-scope: repository
decision-type: technology
---

# 0002. Research-first scope; no implementation toolchain until after Phase 2

## Context and Problem Statement
The start-github-repo `internal-tool` profile emits a Node/TypeScript toolchain. The project's first deliverable is evidence for a council debate on whether Lean 4 can verify protocols and their implementations. The implementation stack (Lean/lake, plus extractors such as Aeneas or hax, or none) depends on that outcome. The Node CI failed on every push because there is no `package.json`.

## Decision Drivers
- CI must be green and must test what this repo actually contains.
- Avoid premature commitment to a stack the council may reject.

## Considered Options
1. Keep the Node scaffold and add a stub package.json.
2. Add a Lean `lake` project now.
3. Remove the Node toolchain and gate CI on the research and docs validators until Phase 3.

## Decision Outcome
Chosen: option 3. CI runs `scripts/check-docs`, `scripts/build-catalogue` (freshness) and `scripts/check-research`. CodeQL scans `actions` and `python`. A Lean toolchain and a reproducibility job that builds the reference repos are added in Phase 3 via a new ADR.

### Consequences
- Good: CI is honest and green, and every Phase 1 requirement is machine-checked.
- Bad: the reference repos are not yet built anywhere, so their "it compiles" claims are unverified (tracked in phase1-research.md Known limits).
