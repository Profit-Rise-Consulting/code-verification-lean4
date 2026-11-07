# PRD: code-verification-lean4

<!-- id: PRD-code-verification-lean4 · status: active (Phase 1b) · owner: Profit Rise Consulting · target release: Phase 2 council-ready · last updated: 2026-11-07 -->
<!-- Rules: IDs are assigned once and NEVER renumbered. Deleted requirements are marked [withdrawn] in place. -->

## 1. Summary

The project's objective is to enable protocol implementation verification on any codebase using Lean 4. It starts by testing the owner's hypothesis: "LEAN4 cannot verify protocols AND therefore unable to check code implementation of these protocols." Phase 1 produces an evidence corpus that is read in full, conflict-preserving and reference-backed. Phase 2 uses it to run a multi-model council debate on the hypothesis. Phase 3 builds whatever the verdict supports. The verbatim owner requests are in [docs/handoff/original-requests.md](docs/handoff/original-requests.md).

## 2. Problem & Context

Protocols such as TLS, QUIC, MQTT and Raft are typically specified in prose, sometimes modelled in a dedicated tool (Tamarin, ProVerif, TLA+), and implemented separately in C, Rust, Go and other languages. The gap between model and code is where many real vulnerabilities live. Lean 4 is a dependently typed proof assistant and programming language with growing industrial use (AWS Cedar, Microsoft SymCrypt via Aeneas). It is unclear whether Lean 4 can model protocols, prove their properties and connect those proofs to arbitrary implementations. The hypothesis states it cannot. That claim is decided by evidence and debate, not by assumption.

## 3. Goals

- G1. Phase 1: an evidence corpus that lets a model council debate each sub-claim (H1-H5, D in [research/CONVENTIONS.md](research/CONVENTIONS.md)) using only fully read sources and inspected code.
- G2. Phase 1: reference dossiers for existing protocol and algorithm verification work, in Lean and comparator tools, analysed, reviewed and generalised.
- G3. Phase 2: a council verdict per sub-claim, with recorded dissent and confidence.
- G4. Phase 3: if the verdict allows, a pilot pipeline that verifies or checks one real protocol implementation against a Lean specification.

## 4. Non-Goals / Out of Scope

- A verdict on the hypothesis in Phase 1. That belongs to the Phase 2 council.
- Choosing an implementation toolchain before the council reports ([ADR-0002](docs/adr/0002-research-first-scope.md)).
- Building or re-proving third-party reference repos in the sandbox (disk-limited; deferred to a CI reproducibility job).
- Verifying cryptographic primitives from scratch. Prior work is referenced instead.

## 5. Users & Personas

| Persona | Context | Primary need |
|---|---|---|
| Council debater (LLM) | Phase 2, argues for or against a sub-claim | Quotable, ID-referenced evidence on both sides |
| Council judge (LLM or human) | Phase 2, scores arguments | Clear motion, definitions, rubric, evidence rules |
| Profit Rise Consulting (owner) | Decides whether to fund Phase 3 | A defensible, sourced verdict |
| Future implementer (weaker LLM or engineer) | Phase 3 | Step-by-step handoff and generalised patterns |

## 6. Success Metrics

| Metric | Baseline (Phase 1, first pass) | Target (Phase 1b) | How measured |
|---|---|---|---|
| Excerpt-only evidence entries outside quarantine | about 110 across 8 files | 0 | `scripts/check-research` NFR-001 |
| Reference dossiers with all 15 sections | 0 | at least 20 (29 planned) | `scripts/check-research` FR-003 |
| Recorded conflicts | 1 | at least 5 | research/CONFLICTS.md |
| Independent review findings with disposition | none | 100% | research/REVIEW-phase1.md |

## 7. User Stories

### US1 — Council can debate with evidence (Priority: P1)

**Why this priority:** It is the owner's stated Phase 1 goal.
**Independent test:** `scripts/check-research` exits 0, and a debater can cite a [FULL] or [CODE] source for each sub-claim, for and against.

**Acceptance scenarios**

- **Given** the council kit **When** a debater argues H3 **Then** evidence packets list dossier IDs and source IDs relevant to H3.
- **Given** two sources disagree **When** the debate opens **Then** the conflict appears in research/CONFLICTS.md citing both.

### US2 — Next agent can execute without the original conversation (Priority: P1)

**Why this priority:** The owner requires handoffs that a weaker LLM can execute.
**Independent test:** handoff.md has the required sections, and every relative link resolves (FR-007, NFR-003).

### Edge cases

- A source is paywalled or unreachable: it goes to Unverified leads, and the gap is reported.
- A repo is AI-generated or unreviewed: it is flagged per CONVENTIONS rule 7 and kept as evidence, with that caveat.

## 8. Functional Requirements

| ID | Requirement | Story | Priority |
|---|---|---|---|
| FR-001 | System MUST record the hypothesis verbatim and decompose it into sub-claims H1-H5 and D | US1 | P1 |
| FR-002 | System MUST give each topic file a FULL/CODE count header and quarantine excerpt-only material under Unverified leads | US1 | P1 |
| FR-003 | System MUST provide reference dossiers with all 15 template sections and a valid index JSON each | US1 | P1 |
| FR-004 | System MUST provide a generated reference catalogue and generalisation matrix that cannot drift from the dossiers | US1 | P1 |
| FR-005 | System MUST keep a Conflicts section per topic file and an aggregated conflict register | US1 | P1 |
| FR-006 | System MUST provide a Phase 2 council kit: motion, definitions, sub-claims, roles and model families, evidence rules, protocol, rubric, evidence packets, output template | US1 | P1 |
| FR-007 | System MUST provide a handoff with objective, verbatim original requests, decision log, inventory, step-by-step Phase 2 execution and known limits | US2 | P1 |
| FR-008 | System MUST record an independent skeptical review by a different model family, with a disposition for each finding | US1 | P1 |
| FR-009 | System MUST produce a Phase 2 council report with a verdict per sub-claim, confidence and dissent | US1 | P2 |
| FR-010 | System MUST, if the council allows, deliver a Phase 3 pilot verifying or checking one protocol implementation against a Lean spec | US1 | P3 |

## 9. Non-Functional Requirements

| ID | Characteristic | Requirement | Target |
|---|---|---|---|
| NFR-001 | Reliability | No evidence entry rests on a search excerpt alone | 0 excerpt markers outside quarantine |
| NFR-002 | Security | Public repo hygiene: no secrets, SHA-pinned actions, least-privilege workflows | validate_repo 0 errors; zizmor 0 findings |
| NFR-003 | Flexibility | Every relative Markdown link resolves | 0 broken links |

## 10. Assumptions, Constraints, Dependencies

- **Assumption:** the public sources and repos cited remain reachable. Commit SHAs are recorded for reproducibility.
- **Constraint:** the sandbox has a 20 GB disk, so no local Lean or Mathlib builds are possible. Agent runs are limited by account credits; Phase 1b was paused once.
- **Dependency:** the start-github-repo skill (bughunt8/skills), GitHub Actions, and model families available for the council.

## 11. UX & Design References

Not applicable: this project has no user interface. WIREFRAME.md and DESIGN.md are deliberately omitted (internal-tool profile).

## 12. Open Questions

- Whether "verify" in the hypothesis means machine-checked proof only, or also includes conformance checking. This is resolved by the council as question D, not here.

## 13. Risks — top three reasons this could fail

1. Evidence skews toward what is easy to find online. Mitigation: comparator dossiers and a skeptical review by a different model family.
2. AI-generated reference repos overstate what is proved. Mitigation: code inspection, with sorry/axiom counts in every dossier.
3. The council converges on one model's prior. Mitigation: model-family diversity rule ([ADR-0004](docs/adr/0004-council-model-diversity.md)).

## 14. Rollout & Release Plan

Phase 1 → Phase 1b (this revision) → Phase 2 council → Phase 3 pilot. Each phase ends with a handoff file and a green `scripts/test`. See [docs/architecture/roadmap.md](docs/architecture/roadmap.md).

## 15. Traceability

The authoritative requirement-to-test matrix lives in [TRD.md](TRD.md) section 4. This document owns
the IDs; the TRD owns the mapping.

## 16. Change Log

| Date | Change | Author |
|---|---|---|
| 2026-11-07 | Initial research-programme PRD (Phase 1b) | coding agent for Profit Rise Consulting |
