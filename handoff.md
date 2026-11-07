# Handoff

Current phase: **Phase 1 complete (Phase 1, 1b and 1c). Next: Phase 2, the model-council debate.**
Read this file top to bottom before doing anything. It is written so that an agent without the original conversation can execute Phase 2.

## Objective

Enable protocol implementation verification on any codebase using Lean 4. Before building anything, test the owner's hypothesis:

> "LEAN4 cannot verify protocols AND therefore unable to check code implementation of these protocols."

Phase 1's goal (owner, R3): enable a strong, successful model-council debate on that hypothesis, based on actual information rather than excerpts. Conflicts are kept as debate material, and existing protocol and algorithm verification work is analysed as references.

## Original requests

The verbatim owner instructions R1-R3 and the agent's interpretation notes are in [docs/handoff/original-requests.md](docs/handoff/original-requests.md). They outrank every paraphrase in this repo.

## Decision log

[docs/handoff/decision-log.md](docs/handoff/decision-log.md) is append-only. Architecturally significant decisions are ADRs:

- [ADR-0002](docs/adr/0002-research-first-scope.md): research-first scope; no Lean toolchain until Phase 3.
- [ADR-0003](docs/adr/0003-full-read-evidence-standard.md): full-read evidence standard, refined by review F-002 into the [FULL]/[SECT]/[CODE] levels in [research/CONVENTIONS.md](research/CONVENTIONS.md).
- [ADR-0004](docs/adr/0004-council-model-diversity.md): six model families in the council; Claude is kept out of voting seats because Claude models wrote the corpus.

## What exists

| Artefact | Path | Purpose |
|---|---|---|
| Rules | [research/CONVENTIONS.md](research/CONVENTIONS.md) | Sub-claims D, H1-H4, H5a, H5b; scope axes; evidence depth; result evidence levels; AI-provenance categories |
| Phase 1 brief | [phase1-research.md](phase1-research.md) | Inventory, what changed in 1b/1c, gaps, how to read the corpus |
| Topic evidence | [research/](research/README.md) 01-08 | Breadth evidence, each with a depth-count header, a Conflicts section and quarantined leads |
| Reference dossiers | research/references/REF-L*.md (Lean), REF-C*.md (comparators) | Depth review of prior work, 15-section template plus §3a predicate semantics |
| Reference catalogue | [research/references/README.md](research/references/README.md) | Generated: property matrix (proved/tested/assumed), implementation-link patterns, reusable patterns, TCB, sub-claim coverage |
| Case comparison | [research/CASE-COMPARISON.md](research/CASE-COMPARISON.md) | Effort, scale, trust and evidence level per principal result; unknowns marked |
| Conflict register | [research/CONFLICTS.md](research/CONFLICTS.md) | CF-NNN entries, both sides quoted, unresolved |
| Independent review | [research/REVIEW-phase1.md](research/REVIEW-phase1.md) | GPT-family skeptical audit: findings, spot-check ledger, coverage by sub-claim, dispositions |
| Council kit | [docs/phase2/council-kit.md](docs/phase2/council-kit.md) | Motion, definitions, roles, evidence rules, protocol, rubric, packets, output template |
| Agent prompts | [research/prompts/](research/prompts/phase1-original.md) | Every prompt used in Phases 1, 1b and 1c, for reproducibility |
| Requirements | [PRD.md](PRD.md), [TRD.md](TRD.md) | FR-001 to FR-010, NFR-001 to NFR-003, and the traceability matrix |
| Open work | [docs/handoff/backlog.md](docs/handoff/backlog.md) (from [backlog.json](docs/handoff/backlog.json)) | Every incomplete task and real issue, as GitHub issues #2-#28 |

Snapshot at handover (2026-11-07): 8 topic files; 33 reference dossiers (22 Lean, 11 comparator); 128 conflicts (CF-001 to CF-128); 73 case-comparison rows; independent review with 14 findings, all verified closed (pass 3). Live counts: `scripts/check-research --output json` and the header of research/references/README.md.

## Phase 2 execution steps

Execute in order. Each step names its output. Do not skip the checks.

0. **Check the backlog.** Read [docs/handoff/backlog.md](docs/handoff/backlog.md). Issues labelled `phase-2` are this phase's work. Issue #21 (thin H5a evidence) should be resolved or explicitly accepted before Round 1. Review dispositions are verified closed (#14, closed).
1. **Verify the starting state.** Run `scripts/test`. It must print `0 error(s)` for check-docs and check-research. If not, stop and fix it first; the corpus is not in its handed-off state.
2. **Read, in this order:** original-requests.md → CONVENTIONS.md → council-kit.md → REVIEW-phase1.md (especially "Coverage by sub-claim") → references/README.md → CONFLICTS.md → CASE-COMPARISON.md.
3. **Pick the roster.** Use six different model families per council-kit "Roles and model families". Record exact model IDs in `phase2/roster.md`. If fewer than six families are available, drop the red team first and record the deviation.
4. **Prepare packets.** Give every participant packets P0-P7 from the council kit, unfiltered. Run each advocate and judge in an isolated context, with no shared scratchpad.
5. **Round 0: definitions (D).** Collect definition proposals, have the judges adopt one set, and write `phase2/round0-definitions.md` with dissent.
6. **Rounds 1-3.** Run openings, rebuttals and the red team for D, H1, H2, H3, H4, H5a and H5b. Save each transcript to `phase2/round<N>-<subclaim>.md`. The clerk checks every citation against the evidence rules and strikes inadmissible ones, logging them in `phase2/ledger.md`.
7. **Round 4: judgement.** Each judge writes `phase2/judge-<family>.md` with a ruling, confidence, scope axes, rubric scores and reasons for each sub-claim. For H2, the judge also states the cost threshold.
8. **Round 5: compound motion.** Rule under the adopted definitions and at least one alternative set.
9. **Report.** The clerk writes `phase2-council-report.md` following the council-kit "Output template". Then add a Phase 2 gate for FR-009 to `scripts/check-research` (required report headings), update TRD §4 status, and append to the decision log.
10. **Hand off.** Rewrite this file for Phase 3. Add ADR-0005, which states the toolchain decision implied by the verdict, then open GitHub issues for every gap the report lists.

## Known limits

- **Nothing was built.** The sandbox disk (20 GB) filled during parallel Lean toolchain installs, so all repository evidence is static inspection at recorded commit SHAs. Sorry and axiom counts are lexical (review F-012). The planned remedy is a Phase 3 CI reproducibility job that builds selected reference repos and runs `#print axioms` on headline theorems.
- **[SECT] evidence** is admissible only for the quoted claim. Some long papers were read by section, and each entry names the sections read.
- **AI-generated references.** Many recent Lean protocol repos (2026) carry AI co-author trailers or author statements of AI assistance; see the `ai_provenance` field. Per CONVENTIONS, this is recorded and never used on its own to discount a result.
- **Not covered:** no formal models were read for Noise, Kerberos, OAuth/OIDC, Bluetooth, EMV or BGP (research/07). The live Lean Zulip was not searchable (research/08). Some papers were paywalled. Each file lists its own remaining gaps.
- **Sources move.** Several repos changed during the research window, for example CCF removing TLA+ on 2026-11-05. Rely on the recorded SHAs, not on HEAD.
- **Independent review.** All 14 findings are verified closed in pass 3 (research/REVIEW-phase1.md). Source conflicts the review surfaced remain in CONFLICTS.md by design.
- **H5a is thin.** No conflicts and little evidence are tagged H5a (H1 → H3). See issue #21.
- **Remote settings not applied** (each needs owner approval): branch ruleset, secret scanning with push protection, private vulnerability reporting, CodeQL default setup (issue #16). Dependabot PR #1 is open (issue #17).
- **All open items** are GitHub issues; see [docs/handoff/backlog.md](docs/handoff/backlog.md).
