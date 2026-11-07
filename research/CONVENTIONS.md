# Research Conventions (Phase 1 and Phase 1b)

Every research agent must follow this file. Phase 1 exists to equip the Phase 2 model-council debate. The debate can only be as strong as the evidence, so these rules are strict.

## The hypothesis and its sub-claims
Hypothesis (verbatim, owner-supplied): "LEAN4 cannot verify protocols AND therefore unable to check code implementation of these protocols."

Phase 2 will debate it as these sub-claims. Tag evidence with the sub-claim IDs it bears on.

| ID | Sub-claim |
|---|---|
| H1 | Expressiveness: Lean 4 cannot express protocol specifications (state machines, message passing, concurrency, nondeterminism, adversary models, time, probability). |
| H2 | Provability: Lean 4 cannot, in practice, prove protocol properties (safety invariants, liveness, refinement, symbolic or computational security) at realistic scale. |
| H3 | Implementation link: Lean 4 cannot connect a protocol specification to implementation code in a mainstream language with a meaningful, stated guarantee. |
| H4 | Generality: no Lean 4 route works for "any codebase", whatever its language, runtime and build. |
| H5 | Inference: H1/H2 being true would imply H3 (the "therefore"). Evidence that implementations can be checked against a Lean spec without protocol-level proofs bears on H5. |
| D  | Definitions: what "verify" and "check" mean (proof, model checking, testing, refinement, conformance). Record how each source uses these words. |

## Evidence rules
1. No excerpt-only evidence. A search-result snippet may be used only to find a source. A claim counts as evidence only if you fetched and read the full primary page, paper, or source code it comes from. Move anything you could not read in full to a final "Unverified leads" section, which Phase 2 will ignore.
2. Prefer substance to description: actual theorem statements (verbatim Lean code with file path, commit and line), definitions, numbers (lines of code, proof size, effort, bugs found, performance), and verbatim author quotes about scope and limitations.
3. For any Lean repository you cite as a reference: `git clone --depth 1` into /tmp. Record the commit SHA, `lean-toolchain`, last commit date and licence. Also record counts of `sorry`, `admit`, `axiom`, `native_decide`, `implemented_by`, `extern`, `opaque` and `partial`, plus Lean LOC. Do not use the GitHub REST API for this; it is rate-limited.
4. Each source gets: ID, title, URL, access date (2026-11-07), [P] primary or [S] secondary, read depth [FULL] or [CODE] (source inspected), and a 1-3 sentence factual summary.
5. Record conflicts between sources explicitly in a "Conflicts" section, citing both sides, without resolving them.
6. Be neutral. Do not state a verdict on the hypothesis. You may describe what a source claims, what it actually demonstrates and what it explicitly does not cover. That factual scope review is required.
7. Flag AI-generated or unreviewed repositories as such and state how you determined it (README statements, commit patterns).
8. Only edit the files assigned to you. Do not run git commit, push or any GitHub write operation.
