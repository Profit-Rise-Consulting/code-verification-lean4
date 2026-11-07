# AGENTS.md

Research programme: can Lean 4 verify protocols and check their implementations in any codebase? The owner's hypothesis is that it cannot. Start every session by reading [handoff.md](handoff.md).

## Commands

```bash
scripts/test             # full gate, identical to CI (check-docs, build-catalogue, check-research)
scripts/build-catalogue  # regenerate research/references/README.md after editing any _index/*.json
scripts/check-research   # corpus and handoff validator; --output json for counts
```

There is no package manager or Lean toolchain in this repo yet ([ADR-0002](docs/adr/0002-research-first-scope.md)).

## Repository map

| Task | Location |
|---|---|
| Where to start, current phase, next steps | `handoff.md` |
| Phase 1 inventory, gaps, Phase 2 brief | `phase1-research.md` |
| Owner's requests, verbatim | `docs/handoff/original-requests.md` |
| Decisions | `docs/handoff/decision-log.md`, `docs/adr/` |
| Evidence rules and sub-claim IDs | `research/CONVENTIONS.md` |
| Topic evidence | `research/01-08` |
| Reference dossiers and catalogue | `research/references/` |
| Conflicts | `research/CONFLICTS.md` |
| Council design | `docs/phase2/council-kit.md` |
| Agent prompts used so far | `research/prompts/` |
| Requirements and traceability | `PRD.md`, `TRD.md` §4 |

## Rules

IMPORTANT: evidence must be [FULL] (source read in full) or [CODE] (repo inspected at a recorded SHA). Search excerpts are leads only and go under "Unverified leads".

- Do not state a verdict on the hypothesis outside a Phase 2 council report.
- Preserve conflicts. Add them to `research/CONFLICTS.md`; never resolve them silently.
- Do not install Lean toolchains or build Mathlib in the sandbox (disk limit). Use shallow clones and delete them after use.
- Dossier and index IDs are never renumbered.
- No remote repository settings changes without explicit owner approval.

## Pull requests

Conventional Commits, scoped with the requirement ID where one applies (`docs(FR-003): add REF-L04 Veil`). `scripts/test` must exit 0. See `CONTRIBUTING.md`.
