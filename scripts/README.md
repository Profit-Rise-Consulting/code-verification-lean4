# scripts/

All scripts are stdlib Python 3 or bash, idempotent, and safe to run locally.

| Script | Responsibility | Requirement |
|---|---|---|
| `test` | Full local gate, identical to CI: check-docs, build-catalogue, check-research | all |
| `check-docs` | Validates the PRD/TRD document spine and traceability matrix | PRD sections 8-9 |
| `check-research` | Validates the research corpus and handoff package (sub-claims, depth headers, excerpt quarantine, dossier sections, index JSON, catalogue freshness, conflicts, council kit, handoff, review, relative links) | FR-001 to FR-008, NFR-001, NFR-003 |
| `build-catalogue` | Regenerates `research/references/README.md` from `research/references/_index/*.json` | FR-004 |

No Lean toolchain is installed by the repo yet. The implementation stack is decided after the Phase 2 council (see [ADR-0002](../docs/adr/0002-research-first-scope.md)).
