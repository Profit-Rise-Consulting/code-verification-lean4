# Research corpus (Phase 1 complete)

Evidence for the Phase 2 council debate on the hypothesis "LEAN4 cannot verify protocols AND therefore unable to check code implementation of these protocols". No file here states a verdict. Rules: [CONVENTIONS.md](CONVENTIONS.md).

## Topic files (breadth)
Each opens with depth counts ([FULL]/[SECT]/[CODE]) and ends with Conflicts and quarantined Unverified leads.

| File | Topic |
|---|---|
| [01-lean4-protocol-modeling.md](01-lean4-protocol-modeling.md) | Lean protocol models: state machines, transition systems, TLA-style and process-calculus encodings, temporal logic, liveness |
| [02-lean4-crypto-and-network-verification.md](02-lean4-crypto-and-network-verification.md) | Lean in crypto and network verification: AWS, Microsoft, zk, VCVio, TLS/MQTT/DNS/HTTP work, consensus |
| [03-lean4-code-extraction-and-fidelity.md](03-lean4-code-extraction-and-fidelity.md) | Linking Lean to real code: Aeneas, hax, LNSym, FFI, Std.Do/mvcgen, translators from C/Go/Python/TS/Solidity/Move, TCB |
| [04-other-tools-protocol-verification.md](04-other-tools-protocol-verification.md) | Comparator tools: Tamarin, ProVerif, TLA+, Ivy, Rocq, Isabelle, F*, Dafny, Verus, Kani, P |
| [05-implementation-conformance-methods.md](05-implementation-conformance-methods.md) | Model-based testing, trace validation, spec-guided fuzzing, automata learning, runtime verification, proof-assistant oracles |
| [06-lean4-language-limits-and-automation.md](06-lean4-language-limits-and-automation.md) | IO/effects/concurrency, trusted base, automation (bv_decide, grind, SMT, LLM provers), releases to v4.36.0-rc2 |
| [07-protocol-spec-formats-and-corpora.md](07-protocol-spec-formats-and-corpora.md) | Spec formats and a candidate benchmark corpus with models, implementations and CVEs |
| [08-academic-literature-and-community.md](08-academic-literature-and-community.md) | Annotated bibliography, researchers, repositories |

## Depth and cross-cutting views
| File | Purpose |
|---|---|
| [references/README.md](references/README.md) | Generated catalogue of reference dossiers (REF-L Lean, REF-C comparators): property matrix, link patterns, reusable patterns, TCB, sub-claim coverage |
| [CONFLICTS.md](CONFLICTS.md) | Aggregated conflict register (CF-NNN), both sides quoted, unresolved |
| [CASE-COMPARISON.md](CASE-COMPARISON.md) | Effort, scale, trust and evidence level per principal result; unknowns marked; no ranking |
| [REVIEW-phase1.md](REVIEW-phase1.md) | Independent GPT-family review: findings, spot-check ledger, coverage, dispositions, verification pass |
| [prompts/](prompts/phase1-original.md) | Every agent prompt used (Phases 1, 1b, 1c) |
