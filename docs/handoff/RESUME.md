# Resume Point: Phase 1b (paused 2026-11-07, credits exhausted)

Branch: `phase1b-wip`. `main` still holds the original Phase 1 output.

## Done on this branch
- Copyright holder changed to Profit Rise Consulting (NOTICE, README).
- Node/Cloudflare scaffold removed. CI rewired to `scripts/check-docs`, `scripts/build-catalogue` and `scripts/check-research`. CodeQL set to actions and python. zizmor is clean offline.
- research/CONVENTIONS.md (sub-claims H1-H5 and D, full-read evidence rules), research/references/TEMPLATE.md and _index/SCHEMA.json.
- Agent prompts recorded in research/prompts/. Verbatim owner requests in docs/handoff/original-requests.md. Decision log. ADR-0002, ADR-0003 and ADR-0004.

## Not done (in order)
1. Phase 1b topic-file upgrades (r01-r08). None written yet. Re-send research/prompts/phase1b-upgrade.md to each agent.
2. 29 reference dossiers (d1-d9; list in research/prompts/phase1b-dossiers.md). None written yet. Constraint: no local Lean builds (disk limit); static inspection only, delete clones after use.
3. `scripts/build-catalogue` to generate research/references/README.md.
4. research/CONFLICTS.md (CF-NNN entries aggregated from all files).
5. Skeptical review by a different model family, written to research/REVIEW-phase1.md with a `## Dispositions` section.
6. docs/phase2/council-kit.md (headings are enforced by scripts/check-research FR-006).
7. Rewrite PRD.md, TRD.md, docs/architecture/*, AGENTS.md, CONTRIBUTING.md, the PR template, README, handoff.md and phase1-research.md for the research programme (FR-001 to FR-010, NFR-001 to NFR-003).
8. Run `scripts/test` until it exits 0, merge to main, and confirm CI is green. Then close or rebase Dependabot PR #1.
