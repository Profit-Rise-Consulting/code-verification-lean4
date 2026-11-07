# Research (Phase 1: gathering only)

Raw findings for the hypothesis: Lean 4 cannot verify protocols, and so cannot check implementations of them.
No analysis here. Each file holds sourced facts, quotes, URLs and open questions. Analysis is Phase 2.

| File | Topic |
|---|---|
| 01-lean4-protocol-modeling.md | Existing Lean 4 protocol models: state machines, transition systems, TLA-style and process-calculus encodings, temporal logic and liveness in Lean |
| 02-lean4-crypto-and-network-verification.md | Lean 4 in crypto and network verification: AWS/Cedar, SampleCore, Verified-zkEVM, Lean-based TLS/QUIC/Noise work, cryptographic game-based proofs |
| 03-lean4-code-extraction-and-fidelity.md | Linking Lean to real code: Aeneas (Rust), hax, LNSym, CompCert-style routes, Lean FFI, `lake`, Lean-to-C, verifying C/Rust/Go/Python/TypeScript |
| 04-other-tools-protocol-verification.md | Comparators: Tamarin, ProVerif, TLA+, Ivy, Coq/Rocq (Fiat-Crypto, VST), Isabelle, F*/Project Everest, Dafny, Verus, IronFleet/Verdi |
| 05-implementation-conformance-methods.md | Model-based testing, refinement, fuzzing from models, state-machine learning, runtime verification, differential testing of implementations vs specs |
| 06-lean4-language-limits-and-automation.md | Lean 4 language/tooling limits: concurrency, I/O, effects, probabilistic reasoning, SMT/automation (bv_decide, Duper, LeanSAT), Mathlib coverage, performance, the Lean 4 reference on IO and unsafe |
| 07-protocol-spec-formats-and-corpora.md | Protocol spec sources (RFCs, ABNF, ASN.1, P4, session types) and candidate benchmark protocols and their existing formal models |
| 08-academic-literature-and-community.md | Papers, theses, talks, Zulip threads, GitHub repos and groups on Lean 4 plus protocols, including claims that it can or cannot be done |

Conventions: every claim carries a URL and access date; mark each as primary or secondary source; list supporting and contradicting evidence side by side without judging; end each file with Open Questions.
