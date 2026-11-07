# Architecture Principles

| # | Principle | Rationale | Implication |
|---|---|---|---|
| P1 | Evidence before opinion | The owner requires a debate grounded in actual information | Full-read rule, [ADR-0003](../adr/0003-full-read-evidence-standard.md) |
| P2 | Decompose compound claims | "cannot verify AND therefore cannot check" has separable parts | Sub-claims H1-H5 and D in research/CONVENTIONS.md |
| P3 | Preserve disagreement | Conflicts are debate material | research/CONFLICTS.md; never resolve conflicts silently |
| P4 | State the trusted base | Verification claims are meaningless without their assumptions | Every dossier has a TCB section with sorry/axiom counts |
| P5 | Independent review | Same-family review shares blind spots | [ADR-0004](../adr/0004-council-model-diversity.md) |
| P6 | Machine-check the process | Handoffs rot unless gated | scripts/check-research in CI |
