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
| H5 | Inference (the "therefore"), debated as two separate implications (review F-009). Tag evidence H5a or H5b where the distinction matters; a plain H5 tag means both. |
| H5a | H1 → H3: if an adequate protocol specification cannot be expressed in Lean, no Lean-based implementation check exists. A method that presupposes a Lean spec cannot by itself rebut this antecedent. |
| H5b | H2 → H3: if protocol properties cannot be proved in Lean, implementations still cannot be checked. Evidence of checking code against a Lean spec without protocol-level proofs (differential testing, trace validation) bears on this. |
| D  | Definitions: what "verify" and "check" mean (proof, model checking, testing, refinement, conformance). Record how each source uses these words. |

## Scope axes (review F-008)
Every ruling and every evidence summary that generalises should state: quantifier (some / typical / every), protocol class, property class, semantics (model, source, binary), implementation layer, assumptions, and resource envelope (human effort, compute, maintenance). A single existential example does not settle a universal claim, and a universal "any codebase" failure does not settle bounded practical use.

## Evidence depth levels (review F-002)
| Tag | Meaning | Admissible? |
|---|---|---|
| [FULL] | Whole source retrieved and read in full | yes |
| [SECT] | Whole source retrieved; named sections read; the claim is quoted verbatim from a read section | yes, for the quoted claim only |
| [CODE] | Repository inspected at a recorded commit SHA | yes, for what the inspected files show |
| [PART], [EXT], ABSTRACT, snippet, extractor | Anything else | no; goes under Unverified leads |

## Result evidence levels (review F-012)
Attach one to each principal result: `author-reported`, `static-inspected` (source read, not built), `rebuilt` (independently built and rechecked, for example `#print axioms`), `deployed` (externally used or replicated). Static counts of sorry or axiom are lexical measurements, not proof-closure audits.

## AI-provenance categories (review F-006)
Use exactly one: `agent-config-present` (AGENTS.md, CLAUDE.md, .claude and similar exist; says nothing about who wrote the proofs), `commit-trailers` (co-author trailers on N of M commits), `author-reported` (the authors state AI assistance), `independently-established`. Never use the first category to discount a mathematical result.

## Evidence rules
1. No excerpt-only evidence. A search-result snippet may be used only to find a source. A claim counts as evidence only at depth [FULL], [SECT] or [CODE] (see above). Move anything you could not read in full to a final "Unverified leads" section, which Phase 2 will ignore.
2. Prefer substance to description: actual theorem statements (verbatim Lean code with file path, commit and line), definitions, numbers (lines of code, proof size, effort, bugs found, performance), and verbatim author quotes about scope and limitations.
3. For any Lean repository you cite as a reference: `git clone --depth 1` into /tmp. Record the commit SHA, `lean-toolchain`, last commit date and licence. Also record counts of `sorry`, `admit`, `axiom`, `native_decide`, `implemented_by`, `extern`, `opaque` and `partial`, plus Lean LOC. Do not use the GitHub REST API for this; it is rate-limited.
4. Each source gets: ID, title, URL, access date (2026-11-07), [P] primary or [S] secondary, read depth [FULL], [SECT] or [CODE], and a 1-3 sentence factual summary.
5. Record conflicts between sources explicitly in a "Conflicts" section, citing both sides, without resolving them.
6. Be neutral. Do not state a verdict on the hypothesis. You may describe what a source claims, what it actually demonstrates and what it explicitly does not cover. That factual scope review is required.
7. Record AI provenance using exactly one of the AI-provenance categories above, with the basis.
8. Only edit the files assigned to you. Do not run git commit, push or any GitHub write operation.
