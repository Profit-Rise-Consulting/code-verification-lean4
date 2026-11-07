# Decision Log

Chronological, append-only. Architecturally significant items also have an ADR.

| # | Date | Decision | Reason | Decided by | Record |
|---|---|---|---|---|---|
| 1 | 2026-11-07 | Scaffold with bughunt8/skills start-github-repo, `internal-tool` profile, Apache-2.0, public | R1 named the skill. internal-tool was the closest profile with no Cloudflare or UI. Apache-2.0 for its patent grant. | agent, owner-approved by R1 | first commit |
| 2 | 2026-11-07 | Repository is `Profit-Rise-Consulting/code-verification-lean4` | R2 | owner | R2 |
| 3 | 2026-11-07 | Phase 1 split into 8 topic areas, one agent each | R1 "research many various aspects ... Spawn multiple agents" | agent | research/prompts/phase1-original.md |
| 4 | 2026-11-07 | Copyright holder is Profit Rise Consulting | R3 item 1 | owner | NOTICE, README |
| 5 | 2026-11-07 | Excerpt-only evidence is rejected; full-read standard adopted | R3 item 3 | owner | [ADR-0003](../adr/0003-full-read-evidence-standard.md) |
| 6 | 2026-11-07 | Add Phase 1b: upgrade 8 topic files and write 29 reference dossiers (20 Lean incl. REF-L21 CCF added on discovery, 9 comparators; REF-L09 unassigned) | R3 "references ... ALL READY for Phase2" | agent | research/prompts/phase1b-*.md |
| 7 | 2026-11-07 | Remove the Node/Cloudflare toolchain; CI runs research and docs gates only | The scaffold CI failed (no package.json). The implementation stack is undecided until after the council. | agent | [ADR-0002](../adr/0002-research-first-scope.md) |
| 8 | 2026-11-07 | No local Lean builds in the sandbox | Parallel toolchain installs filled the 20 GB disk. Builds are deferred to a CI reproducibility job. | agent | phase1-research.md, Known limits |
| 9 | 2026-11-07 | Skeptical review of Phase 1 by a different model family | Owner preference for independent adversarial review | agent | research/REVIEW-phase1.md |
| 10 | 2026-11-07 | Phase 2 council must mix model families and argue sub-claims H1-H5 and D separately | The hypothesis is compound ("AND therefore"), so the parts can have different truth values. | agent | [ADR-0004](../adr/0004-council-model-diversity.md) |
| 11 | 2026-11-07 | Accept all 13 independent-review findings; Phase 1c fixes | Review F-001..F-013 | agent | research/REVIEW-phase1.md Dispositions |
| 12 | 2026-11-07 | Add [SECT] depth level (claim-scoped section reads) | F-002: [FULL] had two meanings | agent | ADR-0003 Amendment 1 |
| 13 | 2026-11-07 | Split H5 into H5a (H1→H3) and H5b (H2→H3); add scope axes | F-008, F-009 | agent | research/CONVENTIONS.md |
| 14 | 2026-11-07 | Council uses six distinct model families; Claude excluded from voting seats | Reviewer found the family-assignment contradiction; Claude wrote the corpus | agent | ADR-0004, council-kit |
| 15 | 2026-11-07 | Add dossiers L21 CCF (discovered mid-run), L22 Perennial, L23 Telltale, C10 Aneris/Trillium, C11 compilation boundary | Discovery by d9; review F-010 | agent | research/references/ |
| 16 | 2026-11-07 | Track every incomplete task and real issue as a GitHub issue from docs/handoff/backlog.json | Owner request (handover) | owner | docs/handoff/backlog.md (#2-#19) |
| 17 | 2026-11-07 | Merge phase1b-wip to main once scripts/test passes and the reviewer's pass 2 is recorded | Owner request | owner | git history |
