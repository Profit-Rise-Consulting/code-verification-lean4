# 08 — Academic Literature and Community Evidence (Phase 1b, full-read upgrade)

**COUNTS: [FULL] = 22 entries; [CODE] = 17 entries; Unverified leads = 21 (excluded from evidence).**

Scope: Lean 4 and protocols, distributed systems, security and implementation verification. Papers, preprints, talks, blog posts, Zulip, GitHub repos, Lean FRO announcements. Neutral evidence record for the Phase 2 debate; no verdict is given anywhere in this file.

- Access date for everything below: 2026-11-07 (Saturday). Hypothesis (owner wording): "LEAN4 cannot verify protocols AND therefore unable to check code implementation of these protocols."
- Sub-claim tags follow `research/CONVENTIONS.md`: H1 expressiveness, H2 provability at scale, H3 implementation link, H4 generality ("any codebase"), H5 inference ("therefore"), D definitions of "verify"/"check".
- [P] = primary source, [S] = secondary. [FULL] = full text of the source was retrieved and read, [CODE] = repository cloned (`git clone --depth 1`, static inspection only, no builds).
- Read-scope note for [FULL] papers: the full PDF/page text was retrieved and read through targeted extraction (abstract, introduction, scope/trust/limitations sections, evaluation tables, conclusion and keyword passes), not line-by-line. Sections not read are named in each entry where it matters.
- Changes from the Phase 1 version of this file: all excerpt-only entries were either re-read in full or moved to "Unverified leads". Phase 1 statements corrected on re-reading are listed in Section 8.

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

---

## 3. [CODE] Repositories inspected by shallow clone

All clones: `git clone --depth 1`, static inspection only. Metrics are naive counts (Section 0). "Stars" from repo HTML on 2026-11-07. Several repos contain `CLAUDE.md`/agent files; where this is present it is recorded under AI-assistance (CONVENTIONS rule 7). Absence of such a file does not show human authorship.

### C01 — verse-lab/veil [P] [CODE]. Tags: H1, H2, D
- https://github.com/verse-lab/veil — SHA 57447722e7f2, last commit 2026-11-07, `lean4:v4.35.0`, LICENSE present, 366 stars (cached page seen earlier showed 350). 495 files, 89,474 Lean LOC.
- Counts: sorry 33, admit 10, native_decide 30, implemented_by 16, partial 67, axiom 0, opaque 1. The `sorry` hits are in the DSL/tactic source that emits `sorry`-bodied theorems when proof reconstruction is off (consistent with E13); not hand-classified exhaustively.
- README (HEAD): lists "(3) model-checking, and (4) proving safety and liveness properties of state transition systems, with a focus on distributed protocols." Examples directory table: `Liveness/` ("Temporal properties proved with `prove_temporal_by`"), `LivenessMC/` ("model checked with `#temporal_check`, including TLA+ specifications and CAVA benchmarks"), `Examples/Demo/FloodSet/FloodSetLiveness.lean`, plus `Simulate`, `AWS`. `docs/DSL-Reference.md`: `#simulate` is described as non-exhaustive testing of the model; the compiled model checker "calls them directly". README line 176 area: a comparison "of TLC and Veil on safety model checking" over 20 TLA+ specs.
- Scope: the repo's verified objects are Veil DSL transition systems. No extraction to or from an external-language implementation was found in the tree (not exhaustively searched).

### C02 — verse-lab/loom [P] [CODE]. Tags: H2
- https://github.com/verse-lab/loom — SHA 78928abc9054, 2026-02-21, `lean4:v4.24.0`, LICENSE, 167 stars; 38 files, 3,094 LOC; sorry 1, admit 1 (naive). Small core library (monad-transformer algebras, WP generation). Clone HEAD is older than Veil's pinned toolchain (v4.35.0), so the Loom copy used by current Veil is not this HEAD (not verified further).

### C03 — verse-lab/velvet [P] [CODE]. Tags: H2, H4
- https://github.com/verse-lab/velvet — SHA b2a349a57021, 2026-11-05, `lean4:v4.35.0`, LICENSE, 136 stars; 140 files, 34,674 LOC; sorry 2, native_decide 22, partial 17.

### C04 — SecPriv/leandy [P] [CODE]. Tags: H1, H2, H3(no)
- https://github.com/SecPriv/leandy — SHA e0705986c5de, 2026-07-07, `lean4:v4.24.0-rc1`, LICENSE, 6 stars; 83 files, 18,096 LOC. sorry 0, admit 0, axiom 0, native_decide 1, partial 1.
- Matches E05: protocol model and proofs are written in Lean's own eDSL; no Rust/C implementation link. Contains payment-channel liveness theorems (`punish.sound`, `punish.eventually_onchain`) stated under continuous-mining and miner-liveness assumptions.

### C05 — Beneficial-AI-Foundation/libsignal-verify [P] [CODE]. Tags: H2, H3, H4, D
- https://github.com/Beneficial-AI-Foundation/libsignal-verify — SHA a69cb0119dd4, 2026-11-06, `lean4:v4.31.0`, LICENSE present (type not checked), 13 stars. 88 Lean files, 26,211 LOC. Counts: sorry 19, axiom 344, native_decide 0, extern 0, opaque 0.
- README: "This project aims to verify the functional correctness of libsignal. More info to be added…".
- `Protocols/LEDGER.md` (dated 2026-10-30), read in full: pins Lean v4.31.0, Mathlib fabf563a, VCVio cbd4144b, Aeneas nightly-2026.09.30. `Protocols/` = 38 Lean files, 7,344 LOC, 19 `sorry`; `ProtocolsTest/` = 4 files, 821 LOC, 0 `sorry`. The rest is Aeneas/Charon-extracted Rust under `rust/Libsignal/Translated/`, whose 344 `axiom`s are almost all external-function stubs in `FunsExternal.lean`/`TypesExternal.lean` (e.g. 190 and 11 in the protocol crate; 73 and 7 in the crypto crate; counts from per-file grep).
- Model-level theorems per the ledger: `uakeInitiator_perfectlyCorrect` and `uakeRecipient_perfectlyCorrect` depend on `propext`, `Classical.choice`, `Quot.sound` only; `uakeInitiator_secure_dh` and `uakeInitiator_secure_pq` depend on `sorryAx` "from six admitted hops". Statement shape (file `Protocols/PQXDH/Security.lean`, line 94): `theorem uakeInitiator_secure_dh (P : Params …) … (A : UAKE.Adversary (uakeInitiator P EntryPoints.extracted msg hasOPK)) … : fullAdvantage P msg hasOPK A ≤ εsig + 2 * q * (εodh + 2 * εaead) + 3 * (q : ℝ) ^ 2 * εpk + …`. The security statement is parametrised by `EntryPoints.extracted`, i.e. by the Aeneas-translated Rust entry points; those translated functions' axiom dependencies include `crypto.hkdf_sha256`, `curve.PrivateKey.calculate_agreement`, post-quantum KEM items, and a native-decision axiom introduced by extraction of `pqxdh_accept` (ledger).
- Fault fixtures exist in `ProtocolsTest/PQXDH/FaultFixtures.lean`. Authors in file headers: Alessandro D'Angelo / Beneficial AI Foundation. Includes Signal's own source files plus `src-modifications.diff` and `extraction-notes.md`.
- Status in the repo's own words: a work in progress. AI-assistance: not determined (no agent file seen); the BAIF organisation publicly pursues AI-assisted Lean verification (E16).

### C06 — dsyme/raft-lean-squad [P] [CODE]. Tags: H2, H3, D. AI-generated: yes (self-declared)
- https://github.com/dsyme/raft-lean-squad — SHA ab7a184fbdfe, 2026-06-03, no `lean-toolchain` file found, LICENSE, 11 stars. 80 files, 21,355 LOC. Naive sorry 29: sampled hits are docstring text "0 sorry" (not exhaustively classified); axiom 1; native_decide 2. An `agents` directory is present; repo is produced by Don Syme's "Lean Squad" agent workflow (self-declared in its REPORT/README).
- REPORT (read): 716 theorems across 79 files, 0 sorry, 129 runs, 28 correspondence targets with 625 `#guard` tests. The Lean code is a hand-written model of raft-rs checked against the Rust by tests (correspondence), not an extraction.
- Theorem (file `FVSquad/RaftTrace.lean:228`): `theorem raftReachable_safe [DecidableEq E] (hd : Nat) (tl : List Nat) (cs : ClusterState E) (hvoters : cs.voters = hd :: tl) (hreach : RaftReachable cs) : isClusterSafe cs` — safety over the model's reachable states.
- Internal tension noted in the report itself: it says all hypotheses are discharged while still listing a "reduce trust boundary" item.

### C07 — NyxFoundation formal-verification repos [P] [CODE]. Tags: H1, H2. AI-assisted: yes for formal-leanSpec
- minimmit-fv: https://github.com/NyxFoundation/minimmit-fv — SHA 145f1089f3df, 2026-07-29, `v4.29.1`, no LICENSE found, 2 stars; 23 files, 2,954 LOC. Naive sorry 3 and admit 1: the hits inspected are docstring phrases ("no `sorry`"); real axioms are two idealised-crypto axioms in `Minimmit/Axioms.lean`: `signature_unforgeable` (line 19) and `collision_resistant` (line 35). `Minimmit/AxiomAudit.lean` pins each lemma's axiom set for CI. Models the Minimmit consensus protocol in Lean (Lemma 5.1 consistency etc.); the Phase 1 statement "contains `sorry`" was not supported.
- rlmd-ghost-fv: https://github.com/NyxFoundation/rlmd-ghost-fv — SHA 5d7d7c58897f, 2026-07-29, `v4.29.0`, 0 stars; 34 files, 5,498 LOC; docstrings state "`sorry`-free, Lean core axioms only" for probabilistic lemmas; `RLMDGhost/Axioms.lean` says the only unproven mechanisms are explicit `axiom`s and hypotheses.
- formal-leanSpec: https://github.com/NyxFoundation/formal-leanSpec — SHA ba7284513031, 2026-07-30, `v4.29.1`, 3 stars; 41 files, 8,534 LOC. `CLAUDE.md` says it is for "Claude Code" and "Each Lean file extracts propositions from the Python spec and proves them as theorems" with "no test runner — every theorem whose proof type-checks is a passing 'test'". AI-assistance flag: CLAUDE.md present.
- All three model protocols/specs in Lean; none runs or extracts the production client.

### C08 — fradamt/verified-consensus [P] [CODE]. Tags: H1, H2, D
- https://github.com/fradamt/verified-consensus — SHA d379754762cc, 2026-10-12, `v4.34.1`, LICENSE, 31 stars; 1,047 files, 334,092 LOC; sorry 0, axiom 0, native_decide 5.
- Theorem `concreteConsensus (S : Setup V) : Statements.Instantiation.Consensus S` in `DecoupledConsensusProofs/ReviewTheorem.lean:288`; the `Consensus` structure has 13 fields (including `certificatesAccountable`, `finalizedAccountable`, `honestNeverSlashed`, `finalizedMonotone`, `finalizedSafe`, `available`).
- README: "no sorry, no project-specific axiom"; assumes unforgeable honest signatures, collision-free roots, bounded delivery after GST, honest committee majorities; liveness additionally needs participation and proposer recurrence; has a premise ledger. Verifies a Lean model of the protocol; the implementation link was not found.

### C09 — etheorem/etheorem [P] [CODE]. Tags: H3, H4, D. AI-assisted: CLAUDE.md present
- https://github.com/etheorem/etheorem — SHA 417ca909aa7d, 2026-11-01, `v4.34.1`, LICENSE, 34 stars; 369 files, 61,202 LOC. Counts: axiom 9, native_decide 795, extern 109, implemented_by 22, opaque 45, sorry 0. README says "not an EF release" (Invisible Garden project).
- Verification-status table in the repo: EthCLSpecs 19 characterized and 58 touched of 625 spec functions; SizzLean 5 of 5 properties over 15 admitted arms; LeanSha256 relies on 3 named equivalence axioms (e.g. `sha256Hash_eq_spec`); EthCLLib proofs out of scope. The repo is an ongoing Lean implementation of Ethereum consensus-layer components; the table is its own account of partial coverage.

### C10 — leanprover/SampCert [P] [CODE]. Tags: H3
- https://github.com/leanprover/SampCert — SHA a2c53bb5ff79, 2026-09-25, `v4.34.1`, LICENSE, 107 stars; 108 files, 18,463 LOC; sorry 0, axiom 0, extern 6, opaque 2, partial 6. Matches E11 (verified sampler code with FFI via a small C++ shim).

### C11 — cedar-policy/cedar-spec [P] [CODE]. Tags: H3
- https://github.com/cedar-policy/cedar-spec — SHA a4c5a89cd156, 2026-11-06, `v4.35.0`, LICENSE, 222 stars; 388 files, 126,845 LOC (Lean plus DRT tooling); sorry 0, admit 1, native_decide 26, partial 42, implemented_by 2. Lean model + proofs next to Rust differential-test fuzzing (E10).

### C12 — edusporto/lean-mqtt [P] [CODE]. Tags: H1, H3, D
- https://github.com/edusporto/lean-mqtt — SHA af23e7bea58d, 2026-09-21, `v4.34.0`, LICENSE, 3 stars; 42 files, 4,751 LOC; sorry 0. Lean MQTT 3/5 packet parser and codec library. Theorem `Parser.bytes_roundtrip` (`LeanMqtt/Core/Parser/Proofs.lean:39`): `theorem Parser.bytes_roundtrip (l rest : List UInt8) : (Parser.bytes l.length).run (l ++ rest) = some (l, rest)`. Accompanying paper at SBLP 2026: https://sol.sbc.org.br/index.php/sblp/article/view/46205 [S, only the landing record read]. Proofs cover the parser; broker/client behaviour is listed as future work in the repo; the Lean code is itself the implementation, written in Lean.

### C13 — leanprover/lean4 (Std.Http, Std.Async) [P] [CODE]. Tags: H1, H4
- https://github.com/leanprover/lean4 — sparse checkout at a2f1a64, 2026-11-06, 9,901 stars. `src/Std/Http`: 47 files, 13,726 lines, 25 theorem/lemma declarations, 0 sorry. `src/Std/Async`: 11 files, 3,132 lines, 0 theorems. Shows that HTTP and async I/O libraries exist in the standard distribution; the proofs present are mostly small lemmas (no protocol-level theorem identified).

### C14 — AeneasVerif/aeneas and charon [P] [CODE]. Tags: H3, H4
- aeneas: https://github.com/AeneasVerif/aeneas — SHA c5eb4a28cef6, 2026-11-07, LICENSE.md, 1,170 stars; 417 Lean files, 78,034 LOC (Lean library and proofs; tool itself is OCaml); sorry 56, axiom 31, native_decide 33, opaque 11, partial 72.
- charon: https://github.com/AeneasVerif/charon — SHA f070bb923309, 2026-11-06, 477 stars; Rust front end (no Lean).
- hax: https://github.com/cryspen/hax — SHA 4d52f781015f, 2026-11-06, LICENSE, 503 stars; 268 Lean files, 53,385 LOC; sorry 16, opaque 64.

### C15 — Other Lean verification repos inspected [P] [CODE]
| Repo | SHA | Last commit | Toolchain | Lean LOC | Notable counts | Stars | Notes |
|---|---|---|---|---|---|---|---|
| rupakm/leslie | 58e58b2502b2 | 2026-06-09 | v4.27.0 | 103,805 | sorry 44 | 16 | scope not read |
| leanprover/con-ron | 97eead2d5b49 | 2026-10-31 | none | 332,513 | sorry 88, axiom 10 | 34 | CLAUDE.md present (AI-assisted) |
| Verified-zkEVM/ArkLib | 34613d617af3 | 2026-11-05 | v4.35.0 | 382,226 | sorry 29, admit 18 | 353 | cryptographic proof systems; VCV-io 172 stars |
| Verified-zkEVM/rust-lean | 49a83d2a231c | 2026-04-18 | none | 9,970 | sorry 24, axiom 22 | 12 | |
| leanprover/cslib | 76b7943d0b8d | 2026-11-05 | v4.36.0-rc2 | 52,113 | sorry 0 | 799 | general CS library; paper is lead L11 |
| leanprover/LNSym | 49f8a1d9e46f | 2026-11-06 | v4.35.0 | 45,232 | sorry 50, native_decide 194 | 121 | ARM machine-code symbolic simulator |
| konnov/leanda | f7abaa1d043e | 2026-07-28 | none | 19,196 | sorry 1 | 21 | |
| athanor-ai/pythia | 65404339b5c6 | 2026-06-15 | v4.28.0 | 66,589 | sorry 104 | 18 | |
| AfonsoBitoque/LeanServer | 24b916aaa6ae | 2026-05-14 | v4.29.1 | 38,581 | native_decide 417, extern 77, opaque 81, sorry 2, admit 7 | 6 | scope and proofs not evaluated |
| solanaqubits/leanbook | 5276529c6c10 | 2026-11-05 | v4.33.1 | 165,863 | sorry 0 | 3 | |
| runtimeverification/kernel-rust-verification-spike | dc20c29367b0 | 2026-08-12 | none | 1,556 | sorry 2, axiom 6 | 4 | RV Binder spike (E18) |
| nocaptheorem/RaftFormalModel | 51f60b20d207 | 2026-09-18 | v4.32.2 | 655 | sorry 18 | 0 | Raft model with 18 sorry |
| akkeshavan/tla-plus-lean-workflow | 991034218346 | 2026-06-29 | none | 3,377 | sorry 8 | 0 | TLA+ to Lean workflow |
| Koukyosyumei/PoL | c71ce2374efd | 2025-08-26 | v4.22.0-rc3 | 919 | sorry 1 | 5 | |
| repowazdogz-droid/escrow-budget | 6595f1283a43 | 2026-07-24 | none | 713 | sorry 0 | 0 | `Escrow.reachable_safe`; separate TLA+ |
| e6qu/rust-lean-aeneas | 146e06a1a78c | 2026-03-23 | none | 40,297 | sorry 66, axiom 449 | 9 | see C16 |
| Chronos-Vault/chronos-vault-contracts | 004ae28b73f0 | 2025-12-23 | none | 3,016 | sorry 59, axiom 13 | 1 | see C17 |
Other stars (HTML): leanprover/lean4 9,901 (a2f1a64); leanprover/mathlib4 4,442 (df6966d, 2026-11-07); microsoft/SymCrypt 898; Chronos-Vault/chronos-vault-security 6. digama0/lean4lean: clone gave no commit data and star count was not retrieved; leanprover/lean4lean returned "repository not found".

### C16 — e6qu/rust-lean-aeneas [P] [CODE]. Tags: H3, D. Flag: low-reliability
- https://github.com/e6qu/rust-lean-aeneas — SHA 146e06a1a78c, 2026-03-23; 133 files. Its own `STATUS.md` states the goal is formally verifying Rust with Lean 4 and Aeneas, and the repo notes (STATUS/WHAT_WE_DID) state that proofs are not real: theorems rely on axioms, Aeneas prelude is faked, concurrency is unsupported. 449 axiom lines. Docs are extensive (LEAN.md, PLAN.md, WHAT_WE_DID.md), which together with the self-description suggests tutorial/agent-generated material; authorship not determined.

### C17 — Chronos-Vault/chronos-vault-contracts [P] [CODE]. Tags: H2, D. Flag: vendor/marketing
- https://github.com/Chronos-Vault/chronos-vault-contracts — SHA 004ae28b73f0, 2025-12-23. Naive sorry 59 (a real placeholder `sorry -- Proof implementation` is in `Trinity.lean`), axiom 13. Its documents give inconsistent theorem counts (184 theorems; "78/78" and "778/78"; 14/22 theorems proven; 42 theorems for 14 contracts). Its developer guide states the Lean proofs cover abstract properties independent of the Solidity implementation. See Conflicts C-6.

---

## 4. Key researchers, groups and organisations (as named in the sources above)

| Name / group | Appears in | Role in the evidence |
|---|---|---|
| Ilya Sergey and co-authors (Pîrlea, Gladshtein, Zhao, Kinsbruner, Kurin, Feng, Kafle); verse-lab; NUS per [NUS news](https://www.comp.nus.edu.sg/bytes/associate-professor-ilya-sergey-and-team-win-distinguished-paper-award-at-cav-2026/) | E01–E04, E13, C01–C03 | Veil, Loom, Velvet |
| Son Ho, Jonathan Protzenko, Aymeric Fromherz, Cédric Fournet, Antoine Delignat-Lavaud and others | E07, E08, E19, E20, C14 | Aeneas/Charon, SymCrypt in Rust, jxl-rs |
| Leonardo de Moura / Lean FRO | E14, E15, E17, E21 | kernel postmortems, SymM, Signal Shot platform, roadmap |
| Beneficial AI Foundation (Max Tegmark named in E17), Signal (Rolfe Schmidt named in E17) | E16, E17, C05 | Signal Shot, libsignal-verify |
| Runtime Verification (Klaus, Conejero, Tolmach) | E09, E18, C15 | Rust-to-Lean pipelines |
| Jeanteur, Veronese, Solitro, Maffei (FBK, TU Wien); SecPriv | E05, C04 | LeanDY |
| Guerraoui and EPFL co-authors | E06 | ConsensusVerifier |
| Cedar team (Cutler et al.), Amazon | E10, C11, E22 | Cedar Lean model and proofs |
| de Medeiros, Tassarotti (NYU), Tristan (Amazon) et al. | E11, C10 | SampCert |
| Nadim Kobeissi / Symbolic Software | E12; cited in E09 | critique of Lean-based crypto proofs; Verifpal |
| Don Syme | C06 | Lean Squad agent-generated Raft verification |
| Daniel Selsam (OpenAI), Kiran Gopinathan | E14, E15 | kernel bug discovery |
| Mario Carneiro | L12 | Lean4Lean (lead) |
| NyxFoundation, Invisible Garden, fradamt (GitHub handle) | C07–C09 | Ethereum-consensus formalisations |

---

## 5. Conflicts between sources (unresolved)

- C-1 Veil and liveness. Side A: an older search-result snippet (not re-fetched, L20) described Veil liveness as future work. Side B: the HEAD README at SHA 57447722e7f2 lists "proving safety and liveness properties" and ships `Examples/Liveness` and `Examples/LivenessMC` (C01). The papers E01/E03 evaluate safety/inductive invariants. Which commit introduced liveness support was not determined.
- C-2 Proof-reconstruction cost. E13 (Apr 2025 Zulip): ">5x slowdown". E01 and E02: "3-5x". Different dates and versions; unresolved.
- C-3 Number of Veil case studies. E01 and de Moura slides (L14): 16, "All 16 verified. Ivy failed on 2". E03: 17 specifications of 15 protocols (Veil 2.0 suite). Different benchmark versions or counting; unresolved.
- C-4 Signal Shot announcement date. E16/E17: 2026-04-20. A conference slide (L14) says "announced April 21, 2026". Unresolved; minor.
- C-5 Bugs found in raft-rs. Lean Squad blog (L19, reported as "no implementation bugs") versus a secondary PQ1 note (L16, 3 bugs across raft-rs, quiche, PX4). C06 REPORT does not settle it.
- C-6 Chronos-Vault theorem counts: 184 vs "78/78" vs "778/78" vs 14/22 vs 42 theorems (C17), all from the repo's own documents.
- C-7 Veil stars: 366 (HTML field on 2026-11-07) vs 350 (cached page). Likely timing; unresolved.
- C-8 Kernel bug-hunt totals. Phase 1 said "5 kernel + 2 runtime"; E15 lists five kernel PRs but notes one (#14807/#14843) as a single bug fix pair and gives no overall total. Count depends on how PRs map to bugs.
- C-9 Aeneas and unsafe/union code. E18: Charon extracts union/MaybeUninit code with zero changes but Aeneas "does not support union types". E07: "limited support for unsafe Rust". E19: "does not have much support for unsafe code" yet models unchecked accesses like regular accesses. These describe different tool layers/features rather than strict contradiction; the exact support boundary is not stated in one place.
- C-10 What a Lean model-check means for the code. C06 (Lean model + `#guard` tests against Rust) vs the claim that Lean has "verified" raft-rs: the REPORT itself separates model proofs from correspondence tests. E10 likewise: proofs on the Lean model, differential testing for the Rust code.
- C-11 Aeneas move to SymM. E21 (roadmap): SymM is used for the VC generator and Velvet "and, in the future, Aeneas". E17: Aeneas "is being ported onto it now". Different dates (roadmap Sep 2026 vs Apr 2026 post); unresolved.
- C-12 Spec fidelity versus proof. E12 and E06 say Lean cannot check that a specification states the intended property; E07 and E16 present specification review as a separate, human step; E10 says model-vs-code agreement is measured by testing. Among the sources read, none claims that Lean itself checks specification fidelity.

---

## 6. How sources use the words "verify" / "check" (sub-claim D)

| Source | Meaning of "verify" | Meaning of "implementation" |
|---|---|---|
| E01–E04 (Veil, Loom, Velvet) | Lean-checked proof of VC soundness; SMT results trusted unless reconstruction is on | A model/program in the Veil or Velvet DSL |
| E05 LeanDY | Lean-checked trace-validity and typing lemmas | Protocol programs written in the library's eDSL |
| E06 EPFL | Lean-checked safety (linearizability) of a Lean transition-system model | Go code written alongside; not proved |
| E07 SymCrypt | Lean-checked functional correctness and panic-freedom of Aeneas-extracted Rust | Real Rust library code |
| E09, E18, E19 | Lean-checked panic-freedom or functional claims over extracted subsets | Rust code in Aeneas/Hax-supported fragments |
| E10 Cedar | Lean-checked properties of a Lean model; differential random testing for the Rust code | Rust code tested against the model |
| E11 SampCert | Lean-checked correctness of Lean programs, then extraction | Extracted C++/Python |
| C06 | Lean theorems about a model; `#guard` tests as correspondence | Rust, via tests |
| E16, C05 | Planned: model security proof + Aeneas-translated code + a link to model assumptions | Real Rust (libsignal) |

---

## 7. Gaps

Closed in Phase 1b: venues for SampCert (PLDI 2025), Velvet (CAV 2026), Veil (CAV 2025), Loom (POPL 2026), Cedar (PACMPL), Aeneas (ICFP 2022); stars and last-commit dates for the repos listed; SHA/toolchain/licence counts; full-text reads of the main papers; the Veil liveness question (partly, see C-1).

Remaining:
- Body of Kobeissi's ePrint 2026/2441 (PDF blocked by robots); ePrint papers on Lean-based cryptography proofs in general.
- Public Lean Zulip threads were seen only through the archive listing, not the live Zulip.
- No public paper or theorem for the Amazon Aurora claim (E22).
- libsignal-verify security theorems still depend on admitted hops; completion state after 2026-11-06 unknown.
- Repository stars for digama0/lean4lean; licence types not checked; many repos have not been read beyond metrics and README/key files (see C15).
- Whether any Lean 4 proof connects a network protocol specification to an unmodified production implementation was not found in the sources read; sources found connect Lean models to code by (a) extraction with trusted tools (E07, E11, C05), (b) differential testing (E10, C06), or (c) Lean-written implementations (E05, C12).

---

## 8. Corrections to the Phase 1 version of this file

- Veil liveness: Phase 1 said future work; current README lists it (C-1).
- SymCrypt: authors/venue: arXiv draft technical report 2609.15648v1 (14 Sep 2026), ten authors (E07).
- Aeneas ICFP 2022 targeted F*; Lean was a plan (E08).
- SampCert: PLDI 2025 and arXiv 2412.01671 (an earlier guessed arXiv id was a different paper).
- Lean4Lean title and figures are lead-level (L12).
- Velvet: CAV 2026, LNCS 16683 pp. 228–242 (E04).
- LeanDY: arXiv 2607.03406v2 technical report; Phase 1 venue claim not found (E05).
- minimmit-fv: no real `sorry`; two explicit crypto axioms (C07). Chronos-Vault: a `sorry` placeholder is real (C17).
- Kernel bug-hunt: no single total (E15).
- Naive grep counts include comments/docstrings (Section 0).

---

## 9. Open Questions

1. For SymCrypt-style results, what is the stated relationship between Lean theorems about the Aeneas-extracted model and the machine code run (compiler, Charon, axiomatised std)? Which of these are formally closed?
2. Can any Lean 4 artefact be shown that states a protocol-level property (not a function-level one) about an extracted production implementation? libsignal-verify is the nearest public attempt but has admitted hops.
3. What does "verify" mean for each audience: proof about a model, proof about extracted code, tests against a model? (Section 6.)
4. How much of Veil's SMT trust (E13) matters in the case studies, and which results used reconstruction?
5. Does the Veil liveness support (C01) have any published evaluation?
6. How should unsupported Rust features (unions, interior mutability, concurrency, trait generics) be accounted for when claiming "any codebase"? (E07, E09, E18)
7. What scale of effort do the claims by agents ("about a week for an agent", E07; $198.69, E06) hold when specs are not already available?
8. What does Kobeissi's body argue about Lean spec fidelity beyond its abstract?
9. Which AI-assisted repos (C06, C07, C09, C15) have independent review of their specs?

---

## 10. Unverified leads (excluded from evidence; Phase 2 should ignore)

- L01 Kobeissi, ePrint 2026/2441 PDF body (blocked). Abstract only is in E12.
- L02 CSF 2026 paper on symbolic cryptography in Lean (computer.org record; ePrint 2025/1700): abstract/snippet only.
- L03 ePrint 2024/1880 (SHA-3 verification): snippet only.
- L04 ePrint 2023/010 (McEliece): snippet only.
- L05 LeanLTL, ITP 2025 (LIPIcs 37), arXiv 2507.01780: text downloaded, not read.
- L06 BB84 end-to-end security proof in Lean, arXiv 2610.17286 (MIT, 12 Oct 2026): abstract/selected sections only.
- L07 Rust-Prover / VeriContest, arXiv 2610.03994: abstract/selected only (1007 problems, 1325 theorems reported).
- L08 Vero, arXiv 2608.13522: abstract/selected only.
- L09 Reentrancy-guard Lean model of Solidity, arXiv 2606.01794: abstract/selected only.
- L10 Rust standard-library verification lessons, arXiv 2510.01072v3: abstract/selected only.
- L11 CSLib, arXiv 2602.04846 (4 Feb 2026): abstract only.
- L12 Lean4Lean, arXiv 2403.14064v3: abstract/selected only (reports checking Mathlib, 20-50% slower than the C++ kernel).
- L13 Lean Zulip archive threads: "Lean in the wild", "A better Lean checker", "System programming using Lean 4", "Lean kernel", "Rust verification working group": listings/snippets only.
- L14 de Moura conference slides (Paris 2026): numbers cited in C-3/C-4 come from slide text, not re-verified.
- L15 mjbommar/axeyum research note on SymCrypt totals: search result only.
- L16 PQ1 note claiming 3 bugs: secondary, not re-read.
- L17 NUS news on Velvet award: not read in full.
- L18 SBLP 2026 lean-mqtt paper: landing record only.
- L19 Lean Squad blog ("no implementation bugs"): not re-read in full.
- L20 older Veil snippet saying liveness is future: unrecoverable.
- L21 digama0/lean4lean repository: no clone metadata.
