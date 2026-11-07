# Phase 1 Research: Brief for Phase 2

Status: Phase 1 complete (1 → 1b → 1c). No verdict on the hypothesis appears anywhere in the corpus. Verdicts belong to the Phase 2 council ([docs/phase2/council-kit.md](docs/phase2/council-kit.md)). Execution steps are in [handoff.md](handoff.md).

## How Phase 1 evolved

| Stage | What happened | Why |
|---|---|---|
| 1 | 8 topic agents gathered evidence into research/01-08 | R1: research many aspects with multiple agents; do not analyse yet |
| Review by owner | Owner rejected excerpt-based evidence and required prior work as analysed references (R3) | "EXCERPTS are not useful"; "THESE ARE REFERENCES ... ALL READY for Phase2" |
| 1b | All 8 topic files rewritten from full reads and code inspection. Reference dossiers written for Lean and comparator work. Catalogue generator, conflict register, council kit and CI gates added. | Make the corpus debate-ready |
| Independent review | GPT-family reviewer: 39 independent claim checks (32 confirmed, 1 wrong interpretation since repaired, 6 unverifiable). 13 findings (1 blocker, 10 major, 2 minor). | Owner prefers a cross-family adversarial review |
| 1c | Every finding accepted and actioned. Five more dossiers (CCF, Perennial, Telltale, Aneris/Trillium, compilation boundary), the conflict register, the case comparison and the issue backlog were added. Review pass 2 re-checked every disposition and added F-014 (Lean-SMT count discrepancy): 11 closed, 3 partial, 0 open, no blockers. | Close the review |

## How to read the corpus

1. **Rules first.** [research/CONVENTIONS.md](research/CONVENTIONS.md) defines the sub-claims (D, H1-H4, H5a, H5b) and the scope axes every ruling must state. It also sets the evidence depth levels: [FULL] whole read, [SECT] claim-scoped section read, [CODE] inspected at a SHA. Result evidence levels and AI-provenance categories are defined there too.
2. **Breadth.** research/01-08 ([index](research/README.md)). Each file opens with depth counts and ends with Conflicts and quarantined Unverified leads. Leads are inadmissible.
3. **Depth.** Reference dossiers in research/references/, listed in the [generated catalogue](research/references/README.md). Each dossier gives:
   - what the authors claim;
   - verbatim theorem statements with file:line;
   - §3a predicate semantics in plain language;
   - property status (proved / tested / assumed);
   - the exact link to implementation code;
   - TCB and soundness hygiene;
   - scale and effort;
   - author-stated limitations;
   - independent evidence;
   - generalisation into a reusable pattern;
   - facts mapped to sub-claims.
4. **Cross-cutting views.** [research/CONFLICTS.md](research/CONFLICTS.md) (disagreements, both sides quoted) and [research/CASE-COMPARISON.md](research/CASE-COMPARISON.md) (effort, scale and trust per result; unknowns marked, no ranking).
5. **Known weaknesses of the corpus itself.** [research/REVIEW-phase1.md](research/REVIEW-phase1.md), "Coverage by sub-claim". The reviewer notes a structural asymmetry: many positive examples are narrow existential results, while broad negative readings claim universal impossibility or impracticality. The council should expose this rather than count citations.

## Generalised implementation-link patterns found

The catalogue's "Implementation-link patterns" and "Reusable patterns" tables are generated from the dossiers. The mechanisms observed across Lean and comparator references are below. These are descriptions of prior work, not recommendations.

| Mechanism | What reaches code | Typical trust gap (from dossiers §5-§6) |
|---|---|---|
| Lean is the implementation | Lean code compiled and run (sometimes via FFI to C primitives) | Lean compiler and runtime, `@[extern]`/`implemented_by`, unverified exporters |
| Extraction from source language | Rust (Aeneas/Charon, hax) or Go (Goose) translated into Lean or Rocq; proofs over the translation | Translator soundness, supported language subset, pinned versions |
| Binary semantics | Machine code symbolically simulated against Lean functional specs | ISA model fidelity, transcribed code, partial specifications |
| Differential testing | An independent implementation tested against an executable Lean model | Test coverage; agreement is evidence, not proof |
| Trace validation / replay | Instrumented implementation traces replayed against a Lean or TLA+ model in CI | Instrumentation placement, dropped events, finite traces |
| Manual transcription | Hand-written Lean models of implementation functions, plus fixture tests | Transcription fidelity |
| Model only | Protocol or algorithm proofs with no code link | Entire model-to-code gap |

## Gaps that remain (consolidated)

- No local builds or `#print axioms` checks; all counts are lexical and static. Remedy: a Phase 3 CI reproducibility job.
- No Lean 4 mechanical refinement from a protocol model to mainstream-language code was found in the Lean topic files. The dossiers record the nearest cases (extraction-based functional proofs; trace replay). Phase 2 must argue what this absence means, per CONVENTIONS rule on absence of evidence.
- No semantics for Lean `IO`/`Task` in Std.Do (lexical search of Std/Do, Std/WP and Std/Tactic/Do; research/06).
- No formal models read for Noise, Kerberos, OAuth/OIDC, Bluetooth, EMV or BGP. No EMV implementation found (research/07).
- Not found: eBay trace validation, Quint/Jepsen, Tamarin-to-code beyond Arquint, Coq/QuickChick oracles, angr conformance (research/05).
- Live Lean Zulip not searchable (research/08). Some primary papers were paywalled. Each file lists its own residual gaps.
- Several 2026 references are fast-moving and AI-assisted. Rely on the recorded SHAs.

## What Phase 2 must produce

`phase2-council-report.md` per the council-kit output template: a ruling for each of D, H1-H4, H5a and H5b with scope axes, confidence and dissent; the compound-motion ruling under at least two definition sets; red-team findings; evidence gaps; and implications for Phase 3 (what may be built, at which assurance level, on which pilot protocol from research/07).
