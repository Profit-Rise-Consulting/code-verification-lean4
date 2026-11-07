# 08 — Academic Literature and Community Evidence (Phase 1b, full-read upgrade)

**COUNTS: [FULL] = @@FULL@@ entries; [CODE] = @@CODE@@ entries; Unverified leads = @@LEADS@@ (excluded from evidence).**

Scope: Lean 4 and protocols, distributed systems, security and implementation verification. Papers, preprints, talks, blog posts, Zulip, GitHub repos, Lean FRO announcements. Neutral evidence record for the Phase 2 debate; no verdict is given anywhere in this file.

- Access date for everything below: 2026-11-07 (Saturday). Hypothesis (owner wording): "LEAN4 cannot verify protocols AND therefore unable to check code implementation of these protocols."
- Sub-claim tags follow `research/CONVENTIONS.md`: H1 expressiveness, H2 provability at scale, H3 implementation link, H4 generality ("any codebase"), H5 inference ("therefore"), D definitions of "verify"/"check".
- [P] = primary source, [S] = secondary. [FULL] = full text of the source was retrieved and read, [CODE] = repository cloned (`git clone --depth 1`, static inspection only, no builds).
- Read-scope note for [FULL] papers: the full PDF/page text was retrieved and read through targeted extraction (abstract, introduction, scope/trust/limitations sections, evaluation tables, conclusion and keyword passes), not line-by-line. Sections not read are named in each entry where it matters.
- Changes from the Phase 1 version of this file: all excerpt-only entries were either re-read in full or moved to "Unverified leads". Three Phase 1 statements were corrected on re-reading (see "Conflicts" C-10, C-11, C-12).

---

## 0. How this file counts repository metrics

Method (CONVENTIONS rule 3): `git clone --depth 1` into /tmp; GitHub REST API not used. Commit SHA and last commit date are those of the cloned HEAD on 2026-11-07. Stars come from the repository HTML page (`stargazerCount` field). Counts of `sorry`, `admit`, `axiom` (lines beginning with `axiom`), `native_decide`, `implemented_by`, `@[extern`, `opaque`, `partial def` are naive text matches over tracked `*.lean` files (lines starting `--` excluded). They include occurrences inside docstrings, strings and tooling that generates `sorry` text, so they are upper bounds on real uses. Where I classified hits by hand this is stated. LOC is raw line count of `*.lean`.

---

## 1. [FULL] Papers and preprints

### E01 — Veil: A Framework for Automated and Interactive Verification of Transition Systems
- Pîrlea, Gladshtein, Kinsbruner, Zhao, Sergey. CAV 2025, LNCS 15933, pp. 26–41. Springer: https://link.springer.com/chapter/10.1007/978-3-031-98682-6_2 ; PDF read: https://ilyasergey.net/assets/pdf/papers/veil-cav25.pdf [P] [FULL]. Tags: H1, H2, D.
- Substance:
  - Scope. "Veil… a framework for… concurrent and distributed algorithms" embedded in Lean 4, with a DSL "inspired by Alloy, Ivy, and mypyvy"; SMT (cvc5 default, 5 s per query) for decidable first-order fragments, Lean interactive proofs otherwise. Main soundness theorem "states the equivalence of the two transition" semantics (the VC generator is proved sound in Lean).
  - Evaluation: 16 case studies (9 from IvyBench with manually added invariants, 2 from Padon et al. on Paxos, 5 from other sources); total 1,704 non-empty non-comment Lean lines (average ~100 per file), 85 actions, 185 invariants. "Veil successfully verified all benchmarks… while Ivy has failed to verify two"; "Veil verifies all but 2 benchmarks in under 15 seconds (87.5%)". Rabia needed a 120 s timeout.
  - Case-study wording: the running example is "an implementation of the Suzuki-Kasami protocol… in Veil" (Fig. 1: "An implementation of the Suzuki-Kasami locking protocol in Veil"), i.e. a transition-system model written in the Veil DSL.
  - Veil vs Isabelle: "Compared with the combination Ivy + Isabelle/HOL, our approach allows the abstract model, the concrete model, and their correspondence to be verified in the same framework, without relying on a trusted manual assumption translation" (for the SCP case study, both models are Veil models).
  - Trust: SMT solvers "do not have to be trusted: Veil supports proof reconstruction (at a 3-5x performance" cost); `set_option veil.smt.reconstructProofs true` "removes SMT solvers from the TCB".
- Does not cover (from the text read): executing or checking an external implementation; liveness (the paper's properties are safety/inductive invariants).

### E02 — Lessons from Building an Auto-Active Verifier in Lean (Veil)
- Pîrlea, Gladshtein, Zhao, Sergey. Dafny Workshop 2026 (POPL 2026 co-located). https://verse-lab.org/papers/veil-dafny26.pdf [P] [FULL]. Tags: H2, D.
- Substance: "For the past 18 months, we have been continuously developing Veil." Stated limitation: "Veil is more than 10x slower than Ivy, the most closely related tool… verification of small specifications taking on the order of a few seconds, and our largest case study taking 160 seconds, compared to Ivy's 50 seconds." On trust: other verifiers' "trusted computing bases (TCBs) are large, encompassing, at the very least, the language semantics axiomatisation, the VC generator, and the SMT solver(s)"; Veil's "mechanised meta-theory serves as the actual implementation of our" VC generator. Proof reconstruction costs "3-5x".

### E03 — Foundational Multi-Modal Program Verifiers (Loom)
- Gladshtein, Pîrlea, Zhao, Kurin, Sergey. POPL 2026, Proc. ACM Program. Lang. 10(POPL). PDF read: https://verse-lab.org/papers/loom-popl26.pdf [P] [FULL]. Tags: H1, H2, H3(partial), D.
- Substance:
  - Loom generates "foundational verifiers for executable effectful programs, shallowly embedded into Lean" via "monad transformer algebras"; Veil 2.0 and Velvet are the two clients ("more than 25 case studies").
  - Veil 2.0 benchmark: "17 specifications of 15 different distributed protocols (two protocols are verified both in a decidable fragment, and separately, in general first-order logic)". Workflow: specify, "SMT-based symbolic bounded model checking (BMC) to ensure the specification is not vacuous", then "iteratively discover an inductive invariant".
  - Non-determinism "to abstract away implementation details" is a stated design choice for Veil specs.
  - A randomised simulation framework for Veil protocols "as a monadic program" was verified in Loom, "proving that it produces only traces that are admitted by the specification". Simulation of NOPaxos found an action whose `require` was always false; fixed and the corrected specification's safety "certified".
  - Future work named: separation logic on top of the state monad transformer; invariant inference; proof repair.
- Does not cover: connecting Veil/Loom specs to external implementation code (no such section in the text read).

### E04 — Velvet: A Foundational Multi-modal Verifier for Imperative Programs in Lean
- Gladshtein, Kurin, Feng, Kafle, Pîrlea, Zhao, Sergey. CAV 2026, LNCS 16683, pp. 228–242 (Open Access, CC BY). https://link.springer.com/chapter/10.1007/978-3-032-32526-6_11 ; PDF read from https://link.springer.com/content/pdf/10.1007/978-3-032-32526-6_11.pdf [P] [FULL]. Tags: H1, H2, H4. Distinguished Paper Award per [NUS news](https://www.comp.nus.edu.sg/bytes/associate-professor-ilya-sergey-and-team-win-distinguished-paper-award-at-cav-2026/) [S, not re-read in full].
- Substance: "Dafny-style verifier for imperative programs embedded in the Lean proof assistant… mutable state, loops, and non-determinism". Evaluation: 27 medium-complexity case studies from the VERINA benchmark; 21 discharged automatically, 6 needed manual proof effort (all but one via `grind`; `isPrime` needed cvc5). Three further case studies: first-fit memory allocator with a free list, prime-number check, Riemann-sum approximation.
- Stated limits: "it does not currently support global heap state or OOP-style classes; programs that essentially rely on shared mutable references" are outside scope. "Currently, the results of SMT solvers are trusted" (reconstruction possible via Lean-SMT). Verified programs are Velvet-language programs, not external code. Mentions Veil for distributed protocols as a separate tool.

### E05 — LeanDY: Type-Based and Trace-Based Symbolic Protocol Verification in Lean
- Jeanteur, Veronese, Solitro, Maffei (TU Wien / Fondazione Bruno Kessler). arXiv:2607.03406v2, 7 Jul 2026 ("technical report"; the Phase 1 claim "submitted to PLAS 2026" was not found in the text read). https://arxiv.org/abs/2607.03406 [P] [FULL]. Tags: H1, H2, H3(no), D.
- Substance:
  - Library "for the Lean proof assistant, building on and extending the design of DY*", supporting "secrecy and authentication for stateful protocols, as well as recursive conditional secrecy for protocols using XOR".
  - Case study: "SegWit-style blockchain primitives" and "payment channels… verifying punishment mechanisms and properties that depend on chain liveness".
  - Wording on "implementation": an "embedded domain-specific language (eDSL) for specifying protocol implementations" and "well-typed protocol implementations respect some confidentiality, integrity, or authentication invariants" — implementations are the library's own eDSL programs; the paper does not extract or link to external code.
  - Central lemma: `transition_valid` proves trace validity "once and for all"; `Traceful := ReaderT ChoiceFn (StateT (T α η) Option)`, with `ChoiceFn` universally quantified so proofs hold for adversarial out-of-order or malformed messages.
  - The paper states the type-based approach's limitation: "proofs cannot be fully automated and require manual intervention".
- Code cross-check: see C04. Repo contains liveness theorems `punish.sound` and `punish.eventually_onchain` ("under continuous mining and miner liveness").

### E06 — The End of Consensus Research? (AI-greement; ConsensusVerifier)
- Burgelin, Guerraoui, Hong, Hu, Kermarrec, Pires, Veski, de Vos (EPFL). arXiv:2610.30114v1 [cs.DC], 21 Oct 2026. https://arxiv.org/abs/2610.30114 [P] [FULL]. Tags: H2, H3, D.
- Substance:
  - "ConsensusVerifier… comprises approximately 135K lines of Lean code, covering shared definitions and formal specifications and proofs for Multi-Paxos, FastPaxos, SwiftPaxos, EPaxos*, Bodega and A-greement. Of these, approximately 98K lines support the final model and proofs of A-greement." Implementation framework ConsensusArena is Go: "approximately 6,800 lines of shared Go runtime code, 19,100 lines across the baseline implementations, and 8,300 lines for A-greement". Whole discovery run about 24 hours, 778.45 M tokens, est. $198.69.
  - Safety only: "Our verification currently targets safety only; we discuss the extension to liveness in § 10."
  - Verified property is linearizability of every execution of the Lean transition-system model: "we prove its linearizability when the early-accept optimization is disabled; when it is enabled, we exhibit a counterexample to linearizability" (Bodega). "four previously unreported safety vulnerabilities" (two in Bodega).
  - Verbatim on the model/implementation boundary: "Although researchers must still manually check that the specification faithfully captures the intended protocol and its assumptions…" and "Lean checks proofs against the encoded specification; researchers must check that this specification faithfully captures the protocol and its implementation. … formalizing a protocol from pseudocode presents substantially different challenges from formalizing it from implementation code… Which approach produces more faithful specifications with less human intervention remains an open question."
- Does not cover: proofs about the Go code; liveness.

### E07 — Scaling Verification of Cryptographic Software with Aeneas, Rust, and Lean
- Ho, Fournet, Protzenko, Naehrig, Clune, Longa, Boisseau, Leal Sánchez, Fromherz, Delignat-Lavaud (Microsoft, Google, CMU, Inria, ENS Paris-Saclay). "Draft Technical Report, September 2026", arXiv:2609.15648v1 [cs.CR], 14 Sep 2026. https://arxiv.org/abs/2609.15648 [P] [FULL]. Tags: H3, H4, D. (Phase 1 recorded Protzenko as Microsoft; this draft lists Google.)
- Substance:
  - Abstract: "Our 237 KLOC Lean development establishes safety, panic-freedom, and functional correctness of 16.7 KLOC of Rust code supporting post-quantum cipher suites for x86-64 and ARM." Effort: "at the time of writing, it takes about a week for an agent and a few days of human review to formalize, implement, and verify an additional algorithm."
  - Method: Charon extracts rustc MIR to LLBC; Aeneas translates to a Lean model; specs are written "independently of the Rust implementation and its correctness proofs" and aligned line-by-line to NIST/IETF standards.
  - Trust assumptions (verbatim): "we trust the Rust toolchain, notably its compiler and its platform-specific back-ends… we verify only functional correctness, not leakage resistance. We trust that Charon correctly extracts MIR into LLBC and that Aeneas correctly translates LLBC into Lean, the soundness of the Lean kernel… and the soundness of the compiled native_decide and bv_decide tactics. Finally, we trust the soundness of the Aeneas axiomatization for some functions of the Rust standard library, and in particular the axiomatization of intrinsics". The paper says the remaining review questions are "is it the right formal specification, is it the right set of properties for this code, and what trust assumptions do they rely on".
  - Aeneas scope: "Aeneas has limited support for unsafe Rust and does not support raw pointers, interior mutability, or concurrency; functions using unsupported features must be modelled by hand."
  - Bugs found: a correctness bug in SHA-3 `extract` (dormant for one-shot calls, reachable via multi-call SHAKE XOFs used by ML-KEM/ML-DSA sampling; the SymCrypt team found it independently by differential testing), an NTT arithmetic bug in a runtime assertion (also found by external audit), a constant-time comparison bug in a Fujisaki-Okamoto routine, and erroneous comments.
  - Side channels: "constant-time style" code but "verify only functional correctness".
- Does not cover: protocol-level (key exchange, handshake) properties; networked or concurrent code.

### E08 — Aeneas: Rust Verification by Functional Translation
- Ho, Protzenko. ICFP 2022 (PACMPL). https://ar5iv.labs.arxiv.org/html/2206.07185 (text read from arXiv PDF 2206.07185) [P] [FULL]. Tags: H3, H4.
- Substance: functional translation for "a large class of Rust programs, as long as they do not rely on interior mutability or unsafe code". Original backend was F\*: "We currently extract to F\*, and have a Coq backend in the works, along with plans for an HOL4 and a Lean backend." Charon "9.5 kLoC"; Aeneas "13.5kLoC… The implementation is, naturally, trusted." "We do not tackle unsafe blocks"; "Our next unit of work is a proof of soundness of our semantics". Case study: a resizing hash table. Positions Aeneas as complementary to RustBelt for unsafe code.

### E09 — A Rust-to-Lean Verification Pipeline with AI Provers: An Experience Report
- Klaus, Conejero, Tolmach (Runtime Verification). arXiv:2605.30106v2 [cs.LO], 2 Jul 2026. https://arxiv.org/abs/2605.30106 [P] [FULL]. Tags: H3, H4.
- Substance: Pipeline uses Charon+Aeneas or Hax to Lean 4, ArkLib/CompPoly specs, AI provers Aristotle (Harmonic) and Aleph (Logical Intelligence). Applied to Plonky3 (FRI folding, Mersenne31 and KoalaBear arithmetic, Horner evaluation) and RISC Zero (Merkle inclusion). Reports "Lean 4 toolchain drift across tools and specific Aeneas/Hax extraction limits"; "Generic functions with trait bounds (e.g., F: Field + TwoAdicField) cannot be extracted"; workaround "Extract a model" when production code uses patterns Aeneas cannot handle. States: "The Lean 4 backend [of hax] is currently under active development, while the F\* backend is more mature", and "the verification work reported here stays within the fragment of pure, bounded-loop functions where Hax's translation is reliable", citing "The Verification Facade: Structural Gaps in Cryspen's Hax Pipeline" (Symbolic Software, 2026) for semantic gaps between hax extractions and Rust semantics.

### E10 — Cedar: A New Language for Expressive, Fast, Safe, and Analyzable Authorization (Extended Version)
- Cutler et al. PACMPL, DOI 10.1145/3649835 (the extended-version PDF 2403.04651 was read; the venue label OOPSLA was not visible in the text read, so it is not asserted). https://dl.acm.org/doi/full/10.1145/3649835 [P] [FULL, sections 1–4 and implementation passages]. Tags: H3, D.
- Substance: Properties proved in Lean: forbid trumps permit, default deny, explicit allow, sound slicing, validation soundness. Implementation link, verbatim: "The Lean model is executable by compilation to native code, so we use extensive differential random testing to confirm that our Rust code and Lean model agree… This testing regime has been fruitful, uncovering nearly two dozen bugs since the project's inception, and forcing us (via CI) to keep the Lean spec and proofs up to date." Proof effort exposed bugs: "Carrying out these proofs, and in particular the proof of validation soundness, revealed several bugs." Symbolic compiler: "proofs of soundness and completeness using the Lean encoder are underway, as of this writing" (as of the paper).
- Does not claim: a proof that the Rust code refines the Lean model. Cedar is an authorization language, not a networked protocol.

### E11 — Verified Foundations for Differential Privacy (SampCert)
- de Medeiros, Naveed, Lepoint, Kahsai, Ravitch, Zetzsche, Joshi, Tassarotti, Albarghouthi, Tristan (NYU, Amazon). arXiv:2412.01671v3 (29 Apr 2025). Venue PLDI 2025 / PACMPL per a search-result record of the title; the PDF text I read shows the arXiv header only, so the venue is [S]-level. https://arxiv.org/abs/2412.01671 [P] [FULL, sections on architecture, extraction, deployment]. Tags: H3, D.
- Substance: "SampCert is written in Lean with over 12,000 lines of proof". Extraction to executable code: "compile SLang programs to conventional languages like C++ and Python… the Lean foreign function interface (FFI), … adding only 57 lines of C++ to our trusted computing base. The second involves a translation from SLang to Dafny and then from Dafny to Python, which we used in order to deploy our verified sampling" (AWS). "it is paramount that external definitions be kept as minimal as possible." Floating point is avoided on purpose.
- Notes: the guarantee flows from Lean spec to code extracted by a small trusted compiler path (the implementation is generated from the verified Lean program, not an existing codebase checked after the fact).

### E12 — Nobody Will Read the Lean: A Case for High-Level Cryptographic Languages
- Nadim Kobeissi (Symbolic Software). IACR ePrint 2026/2441, received 2026-10-10, approved 2026-10-14 (ePrint is screened for scope, not peer-reviewed). https://eprint.iacr.org/2026/2441 [P] [FULL for the abstract record only: the PDF is blocked to fetchers (robots.txt), so the body is NOT read; see Unverified leads]. Tags: D, H2.
- Substance (abstract, verbatim): "What Lean can't check is whether the theorem says what you think it says. … A Lean library's security theorem for Schnorr signatures passed the checker, yet as written it held for every adversary and so promised nothing." "on a one-page Verifpal model, two of five verdicts would flip under other standard definitions, with neither answer wrong." The page refers to Lean but not specifically Lean 4.

---

## 2. [FULL] Web pages, posts, announcements, Zulip

### E13 — Zulip: "Veil framework discussion thread" (stream general), 14 Apr 2025
- https://leanprover-community.github.io/archive/stream/113488-general/topic/Veil.20framework.20discussion.20thread.html [P] [FULL, whole thread page]. Tags: D, H2.
- Substance (Pîrlea): "When proof reconstruction is off, the SMT solver's output is simply trusted rather than checked. This is similar to existing non-foundational verifiers. We keep it off by default since it's quite slow (>5x slowdown) and currently a bit brittle". Without reconstruction the generated theorem is "a `theorem` whose body is `by sorry`" (Srinivas: "a lean project using Veil won't be sorry free"; `lake build` "would not distinguish Veil-generated sorries from forgotten, manually inserted ones").

### E14 — Postmortem for Kernel Soundness Bug #14576 (de Moura, 2026-08-01)
- https://leodemoura.github.io/blog/2026-8-1-postmortem-for-kernel-soundness-bug-14576/ [P] [FULL]. Tags: D (trust), H4.
- Substance: root cause was phantom parameters of a nested inductive escaping type checking in kernel elimination, letting "an ill-typed argument… make the kernel accept a proof of `False`". "The bug is only reachable through metaprogramming, by sending the inductive declaration to the kernel directly." Reported 2026-07-28 (Kiran Gopinathan), "We pushed a fix one hour after the report (#14577)." Author: "This is an implementation bug, not a hole in Lean's meta-theory." "The elaborator is untrusted by design."

### E15 — Postmortem for the Kernel Soundness Bug Hunt (de Moura, 2026-08-24)
- https://leodemoura.github.io/blog/2026-8-24-postmortem-for-the-kernel-soundness-bug-hunt/ [P] [FULL]. Tags: D (trust).
- Substance: Daniel Selsam (OpenAI) used internal models 2026-07-30 to 2026-08-20. Kernel PRs listed as soundness fixes: #14613, #14616, #14806, #14807/#14843. Two runtime exploits: reference-count overflow (#14838) and Linux binaries built with GMP 6.1.2, which "contained a bug that could be exploited to construct a proof of `False`" (#14833). The page says the bogus proof for #14807/#14843 "was accepted by `nanoda`", while `lean4lean` was not affected and `ind-models` rejected it. (Phase 1 summarised this as "5 kernel + 2 runtime bugs"; the page does not state a single total.)

### E16 — Signal Shot: One Giant Lean for Protocol Security (BAIF, 2026-04-20)
- https://www.beneficialaifoundation.org/blog/signal-shot [P] [FULL]. Tags: H3, D.
- Substance: "We're verifying the Signal protocol and its Rust implementation using the Lean Proof Assistant, in a public moonshot to show that formal verification of major applications is doable today." Stated plan: "first, we write down a mathematical model of the protocol and prove its security properties. Then, we take Signal's existing Rust implementation and translate it to Lean using the Aeneas tool. Finally, we verify that the translated code actually satisfies the assumptions of our proven model — bridging the gap from theory to real-world code." Stack stages: cryptographic infrastructure, PQXDH, Double Ratchet, SPQR/Triple Ratchet, functional correctness, anonymous credentials, session management, protocol composition. The page presents a plan; no completion is claimed. Tooling named: VeriLib, Verso-Blueprint.

### E17 — Signal Shot: The Platform Is Ready (de Moura, 2026-04-20)
- https://leodemoura.github.io/blog/2026-4-20-signal-shot-the-platform-is-ready/ [P] [FULL]. Tags: H3, H4, D.
- Substance: "Signal Shot is a joint effort of Signal (Rolfe Schmidt), the Beneficial AI Foundation (Max Tegmark), and the Lean FRO." "LTE proved mathematics scales in Lean. Signal Shot is the same question for deployed software." "`SymM` is our new monadic framework for software verification." "Aeneas, the tool Signal Shot will lean on for `libsignal` verification conditions, is being ported onto it now." Stated gaps: "Aeneas needs help scaling to `libsignal`. The verification pipeline around it needs benchmarks, regression tests, and glue. The cryptographic primitives need you. The protocol specs need you. We need your eyes on the definitions before anyone proves anything." "Not long ago this was impossible."

### E18 — Verifying Linux's Rust Code: From Binder To Lean 4 (Runtime Verification, 2026-08-13)
- https://runtimeverification.com/blog/verifying-linux-rust-code-from-binder-to-lean4 [P] [FULL]. Tags: H3, H4.
- Substance: Property proved: "For all byte inputs from userspace, the Binder deserializer returns cleanly: a valid object or EINVAL, never a panic. Machine-checked end-to-end through Rust, Charon, Aeneas, and Lean 4, sorry-free, modulo one opaque axiom about size_of." Tool limits: Charon extracted the unsafe/union code "with zero changes"; "Aeneas does not support union types, and the deserializer is built on exactly that… no Lean output at all for the parsing core" (upstream issue aeneas#1199). Workaround: re-model the union as `[u8; 40]` with accessor functions, "Compile-time assertions pin every accessor offset and every size to the real `#[repr(C)]` layout"; unsafe-block invariants become explicit Lean propositions. "The re-model is the interim path while union support lands."

### E19 — A case study with Aeneas and jxl-rs (Protzenko, 2026-05-05)
- https://jonathan.protzenko.fr/2026/05/05/jxl-rs.html [P] [FULL]. Tags: H3, D.
- Substance: Proves absence of panics (specs of the form `f ⦃ r => True ⦄`), not functional correctness: "we are only doing partial verification". "Aeneas does not have much support for unsafe code; but unchecked accesses such as the one above are modeled like regular accesses". Missing third-party functions are emitted "as axioms in a Lean (file `FunsExternal_template.lean`), but no body is provided". "the `read` function does not actually preserve the invariant: it only does so as long as we have not reached the end of the input stream."

### E20 — SymCrypt README-VERIFIEDCRYPTO.md (branch feature/verifiedcrypto)
- https://raw.githubusercontent.com/microsoft/SymCrypt/feature/verifiedcrypto/README-VERIFIEDCRYPTO.md [P] [FULL]. Tags: H3, D. Branch last commit c2e575a, 2026-07-12 (partial clone, tree not inspected); default-branch HEAD 286762b, 2026-08-05; stars 898.
- Substance: "Formal theorems apply only in a model of the Rust code and the system that uses it." Trust list: Rust toolchain and back-ends for x86_64 and AArch64; "the Charon/Aeneas toolchain correctly extracts code from Rust to Lean, and the soundness of its axiomatization of the Rust standard library"; "We verify safety, panic-freedom, and functional correctness but not leakage resistance." Table: ML-KEM Rust 2,125 cloc, spec 308, properties and proofs 37,989. A third-party document (github.com/mjbommar/axeyum, `docs/prover-track/research/04-software-ir-verification.md`, seen in a search result only, not fetched) gives totals ~5,529 Rust LOC and ~58,377 proof LOC (~10.6:1); treat as an unverified lead.

### E21 — Lean FRO Year 4 roadmap, Part 1 (Sep 2026–Feb 2027)
- https://lean-lang.org/fro/roadmap/y4-1/ [P] [FULL]. Tags: H3, H4, D.
- Substance: "On the software verification side, we will complete the new do notation, including first-class support for intrinsic verification, and deliver improvements to the verification-condition generator… We will also continue improving the performance of `SymM`, our new tactic framework, which is used to implement the verification-condition generator, Velvet, and, in the future, Aeneas." Trust: new `lake check`, `lake check --paranoid`, comparator in the distribution. The page does not mention Std.Http, Async, network libraries or protocols.

### E22 — Amazon is investing in the Lean Focused Research Organization (Amazon Science, 2026-07-26)
- https://www.amazon.science/news/amazon-is-investing-in-the-lean-focused-research-organization [P] [FULL]. Tags: H2, H3.
- Substance: "Amazon uses Lean-based verification across multiple products including Bedrock AgentCore for agentic safety, SampCert for differential-privacy guarantees, and AWS Neuron for AI chip compilation, with scientists using LLM-Lean combinations to prove complex distributed protocol correctness." "One scientist recently used an LLM with Lean to prove the correctness of Amazon Aurora's segment repair protocol, our most durability-critical distributed protocol, in a fraction of the time it would have taken manually." No paper, theorem or scope is given for the Aurora claim; the page does not mention Cedar.
