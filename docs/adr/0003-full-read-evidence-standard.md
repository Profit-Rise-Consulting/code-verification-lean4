---
status: accepted
date: 2026-11-07
decision-makers: Profit Rise Consulting (owner)
decision-scope: research
decision-type: process
---

# 0003. Full-read evidence standard

## Context and Problem Statement
The first Phase 1 pass relied heavily on search-engine excerpts; one topic file had 58 excerpt-only entries. The owner ruled: "EXCERPTS are not useful - the research should have actual information for Phase2 to be successful."

## Considered Options
1. Allow excerpts, labelled as such.
2. Allow only fully read sources and inspected code. Quarantine everything else.

## Decision Outcome
Chosen: option 2, as codified in research/CONVENTIONS.md. Evidence must be [FULL] (the primary page, paper or document read in full) or [CODE] (source inspected at a recorded commit). Excerpt-only material lives under "Unverified leads (excluded from evidence)". `scripts/check-research` fails CI if excerpt markers appear outside that section (NFR-001).

### Consequences
- Good: council debaters can quote theorem statements, numbers and author scope statements directly.
- Bad: coverage shrinks where sources are paywalled or unreachable. Those gaps are reported, not hidden.
